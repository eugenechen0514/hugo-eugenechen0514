+++
title = "一窺 Kafka 內部儲存資料"
date = "2021-08-22"
description = "從 Kafka 內部儲存資料來看 Kafka 的基本概念是很合適的，因為怎麼存就會了解它的架構和限制。"
featured = false
categories = [
  "隨手筆記"
]
tags = [
  "Kafka"
]
images = [
  "posts/notes/2021-08-22-一窺kafka內部儲存資料/images/cover.png"
]
+++


從 Kafka 內部儲存資料來看 Kafka 的基本概念是很合適的，因為怎麼存就會了解它的架構和限制。

<!--more-->

Kafka 本質上是 *Distributed, Replicated Messaging Queue* ，在微服務和分散式計算的經常會被提及。要最大化效能就要對資料的存放有些了解。

Kafka 的基本概念如下圖：

![IMAGE](images/cover.png)

下圖是概念和內部儲存資料的關係圖：

![IMAGE](images/look-deeper.jpg)

> 注意：上圖的 index/timeIndex 檔案只是示意圖，它們不是一個每筆 message 都有一筆 index 資料，見下面的實測。 

## 窺視一個 partition 的資料夾

```shell
$ ll /data/kafka/kafka-logs/test.eugene.test-7
```

```
total 8.0K
-rw-r--r-- 1 root root 10M Aug 22 18:31 00000000000000000000.index
-rw-r--r-- 1 root root  88 Aug 22 18:35 00000000000000000000.log
-rw-r--r-- 1 root root 10M Aug 22 18:31 00000000000000000000.timeindex
-rw-r--r-- 1 root root   8 Aug 22 18:31 leader-epoch-checkpoint
```

用工具 [DumpLogSegments](https://jaceklaskowski.gitbooks.io/apache-kafka/content/kafka-tools-DumpLogSegments.html) 可以一窺內容

### [OffsetIndex](https://jaceklaskowski.gitbooks.io/apache-kafka/content/kafka-log-OffsetIndex.html) - Index Of Offsets Of Log Segment
```shell
$ bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration --print-data-log --files /data/kafka/kafka-logs/test.eugene.test-7/00000000000000000000.index
```
```
Dumping /data/kafka/kafka-logs/test.eugene.test-7/00000000000000000000.index
offset: 0 position: 0
```

### [TimeIndex](https://jaceklaskowski.gitbooks.io/apache-kafka/content/kafka-log-TimeIndex.html) - Index Of Timestamp And Offsets Of Log Segment

```shell
$ bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration --print-data-log --files /data/kafka/kafka-logs/test.eugene.test-7/00000000000000000000.timeindex
```

```
Found timestamp mismatch in :/data/kafka/kafka-logs/test.eugene.test-7/00000000000000000000.timeindex
  Index timestamp: 0, log timestamp: 1629628512555
```

### [Log File](https://jaceklaskowski.gitbooks.io/apache-kafka/content/kafka-log-Log.html)

```shell
$ bin/kafka-run-class.sh kafka.tools.DumpLogSegments --deep-iteration --print-data-log --files /data/kafka/kafka-logs/test.eugene.test-7/00000000000000000000.log
```
```
Dumping /data/kafka/kafka-logs/test.eugene.test-7/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1629628512555 size: 88 magic: 2 compresscodec: NONE crc: 1254090055 isvalid: true
| offset: 0 CreateTime: 1629628512555 keysize: 0 valuesize: 18 sequence: -1 headerKeys: [] key: 12 payload: {
    "data": 12
}
baseOffset: 1 lastOffset: 1 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 88 CreateTime: 1629628976272 size: 90 magic: 2 compresscodec: NONE crc: 2535940961 isvalid: true
| offset: 1 CreateTime: 1629628976272 keysize: 2 valuesize: 18 sequence: -1 headerKeys: [] key: 13 payload: {
    "data": 13
}
```


## 相關連結
[A Practical Introduction to the Internals of Kafka Storage | Medium](https://medium.com/@durgaswaroop/a-practical-introduction-to-kafka-storage-internals-d5b544f6925f)
