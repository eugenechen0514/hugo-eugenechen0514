+++
title = "Day 15 - 二周目 - 用 Docker 玩轉 MongoDB"
date = "2018-10-15"
description = "今天要來準備存放資料的地方 MongoDB 和如何存取裡面的資料。"
featured = false
categories = [
]
tags = [
"2019 iT 邦幫忙鐵人賽",
"用js成為老闆心中的全端工程師"
]
images = [
]
series = [
"用js成為老闆心中的全端工程師 - 2019 iT邦幫忙鐵人賽"
]
+++

今天要來準備存放資料的地方 MongoDB 和如何存取裡面的資料。

<!--more-->

# 回憶
前二天我們花了很多時間在講 Prmoie 和 async/await，對於到處都是非同步的 Node.js 來說是很重要的，務必多多練習。

# 目標
今天要來準備存放資料的地方 MongoDB 和如何存取裡面的資料。
1. 安裝 docker
1. 安裝並執行 MongoDB
1. 使用 MongoDB driver for Node.js
2. 使用 mongoose

# Docker 架設 MongoDB
因為我希望二周目可以集中心力在講前後端，所以 Docker 的詳細使用會放到三周目，但我還是會保持最底限度的了解。

Docker 它可以安裝在不同的作業系統，它關鍵的技術是 **容器(container)**。當容器初次建立時會載入某個映象檔(image)，映象檔打包著相關程式碼、函式庫、環境配置檔，然後容器就會執行在一個沙箱的執行環境。這技術跟模擬整個作業系統的虛擬機器不同，容器是更小的執行環境。

![IMAGE](resources/0DACD1CCE1EB72B8DAAA71E45E6D0ED4.jpg =720x431)
上圖截取自 [Docker Reference Architecture: Designing Scalable, Portable Docker Container Networks](https://success.docker.com/article/networking)

因為容器是執行在 Docker Engine 上，且我們在三周目也會自己做映象檔並寫執行組態檔，所以我們要安裝 [Docer Desktop](https://www.docker.com/products/docker-desktop)，它的功能比較多。

## 用 docker 架設 mongodb
### docker 環境
1. 依照自己作業系統安裝
    ![Screen Shot 2018-10-14 at 8.31.34 PM.png](resources/A5C5F4F954277B7F0BDF61F4BB19350F.png =744x367)
1. 灌完後執行 docker，就會有個常駐的小圖標![Screen Shot 2018-10-14 at 8.33.44 PM.png](resources/AF9F155ACD00D483BA5D270B7D76CC9A.png =55x21)，之後我們就可以用 `docker` 指令了

### mongodb 容器執行
1. 複製 `hello-express`專案(或繼續使用)改名成 `hello-mongo`。(若沒有 `hello-express` 的人可以看 [Day 8 - 一周目- 開始玩轉後端(一)](https://ithelp.ithome.com.tw/articles/10200476))
1. 下載 mongodb 的映象檔 ([mongodb 官方印像檔](https://hub.docker.com/_/mongo/) 也有其它版本可以下載)
    ``` shell
    docker pull mongo:4.1
    ```
    ![Screen Shot 2018-10-14 at 9.30.20 PM.png](resources/352BC4CC7A664C4E49C6C4E7A91EDE59.png =760x275)
    > `docker images` 可以列出有下載的映象檔，看看有沒有下載成功

1. 在 `hello-mongo`根目錄建立一個 data 資料夾 `mkdir data`。用來放 mongodb 的資料，可以讓 mongo 容器刪除時可以留下資料，下次建立容器可以繼續使用
1. terminal 移到專安根目錄後，建立容器且執行 mongo 容器
    ``` shell
    docker run --name mongo4 -v $(pwd)/data:/data/db -d -p 27017:27017 --rm mongo:4.1
    ```
    > `docker ps` 確認容器有執行
    ![Screen Shot 2018-10-14 at 10.12.28 PM.png](resources/4CA0455B790BCCEE71D4C32F684240C2.png =988x44)
    
    > 我們用映象檔 `mongo:4.1` 建立了名為 `mongo4` 的容器，掛載 `hello-mongo` 根目錄下的 `data` 資料夾到容器內。容器在背景執行，且對外的 port 號是 27017。當容器停止後自行移除。

1. 確認 mongodb 資料庫有運行
    ``` shell
    docker exec mongo4 mongo --eval "print(version())"
    ```
    ![Screen Shot 2018-10-14 at 10.23.34 PM.png](resources/8DD4D1DB582E24824B844B9A3670598E.png =635x87)
1. 若需要停止容器 (關掉 mongodb 資料庫)，請輸入
    ``` shell
    docker stop mongo4
    ```
    就會停止容器並刪除

到目前為止，我們完成了用 docker 架設 mongodb 資料庫

> 若要部署 docker mongo，應該還要加上安全性的設定(像是帳密)，且用長駐的 docker container 執行(像是用 `docker-compose` 組態設定)

# 在 Node.js 中使用 MongoDB
架設完 mongodb 資料庫後，我們就可以用 client 端連入資料庫使用了。

mongodb官方提供很多程式語言的 driver (就是套件)連入資料庫，見[
MongoDB Drivers and ODM](https://docs.mongodb.com/ecosystem/drivers/#mongodb-odm-object-document-mapper)。我們的 Node.js 是 [MongoDB Node.JS Driver](https://mongodb.github.io/node-mongodb-native/?jmp=docs)。

## 用 `mongo` 指令連入資料庫
另外，我強力建議開發時，安裝 shell(cli) 版本的 client，可以方便我們直接下 `mongo` 指令連線後做練習和測試，但不同的作業系統有自己的安裝方法，請見 [
Install MongoDB](https://docs.mongodb.com/manual/installation/)。只是上面的安裝方法不僅安裝 shell 版還安裝伺服器版，也有只安裝 shell client 的方法，但請自己研究一下。

針對有潔癖的人、不想在本機端安裝的人，我提供一個方法使用 `mongo` 指令，就是：
```
進入容器內
```
因為 mongo 容器中除了資料庫的執行檔還有包含 shell client，所以進入容器就可以使用 `mongo` 指令，也不用擔心本機端安裝的 shell clinet 版本號不合資料庫版本號。

1. 輸入 `docker exec -it mongo4 bash`，就可以進入容器內
    ![Screen Shot 2018-10-14 at 10.57.36 PM.png](resources/58781C2E13DC70A6E0FAFFE8074812BE.png =460x36)
1. 就能用 `mongo` 指令，連入資料庫
    ![Screen Shot 2018-10-14 at 10.55.17 PM.png](resources/349D63CF3B4B04FB24699787A1561FB2.png =935x425)
1. 不用的時候直接關 terniaml　或 ctrl + c 和 `exit` 就可以離開容器
    ![Screen Shot 2018-10-14 at 11.01.11 PM.png](resources/29B627D1AF384074D3ADDE453BB3E241.png =937x483)

> `mongo` 指令直接下時是連到本機端 localhost 的 27017 port 的預設值

### Node.js Driver 連入

Node.js Driver 提供的非同步的函數，幾乎同時支援 callback 和 回傳 Promise，之後的例子我們儘量使用 Promise

1. 安裝 Node.js Driver 
    ``` shell
    npm install mongodb --save
    ```
1. 修改 `./router/index.js`，使用 `mongodb` 套件，加入一個 `GET /api/mongo` api 來看 mongodb 是否連線成功
    > 注意: 下面的程式，每次打一次 API 都會建立一條連線，很浪費資源，晚一點我們會修改下面的程式碼

    ``` javascript
    const MongoClient = require('mongodb').MongoClient;
    
    router.get('/api/mongo', function (req, res, next) {
      // mongodb 位置
      const url = 'mongodb://localhost:27017';
    
      // 資料庫名
      const dbName = 'myproject';
    
      // 連立一個 MongoClient
      const client = new MongoClient(url, {useNewUrlParser: true});
    
      // client 開始連線
      client.connect()
        .then(() => {
          res.json({
            isConnected: true,
          });
        })
        .catch(error => {
          console.error(error);
          res.json({
            isConnected: false,
          });
        });
    });
    ```
1. 用 Postman 打看看`GET /api/mongo`，若有連上 mongodb
    ```　json
    {
        "isConnected": true
    }
    ```
    若把 mongodb 關掉 (`docker stop mongo4`)，會得到
    ``` json
    {
        "isConnected": false
    }
    ```
#### 重用(reuse)連線
``` javascript
const client = new MongoClient(url);
client.connect();
```
會建立一條連線，一但接上，沒有手動 `client.close()` 或發生斷線，此連線不會釋放。因此，我們要留下已連線的 client，放在未來可以拿的到的地方

``` javascript
const MongoClient = require('mongodb').MongoClient;

// index.js 執行就建立連線
const url = 'mongodb://localhost:27017';
const dbName = 'myproject';
const client = new MongoClient(url, {useNewUrlParser: true});
client.connect()
  .then((connectedClient) => {
    console.log('mongodb is connected');
  })
  .catch(error => {
    console.error(error);
  });

router.get('/api/mongo', function (req, res, next) {
  res.json({
    isConnected: client.isConnected(),
  });
});
```

### Collection 的 CRUD (Create, Read, Update, Delete)
上一節我們談了建立連線，現在要來操作資料，先來了解 MongoDb 的名詞對比 MySQL

| type | data collection | element |
| ---- | --- | -- |
| MySQL(SQL)| table | record |
| MongoDb(NoSQL)| collection | document |

MongoDB 的 document 有以下特性：
* 物件資料：一個 collection 可放入很多 document，且 document的資料可以是 JSON 或 BSON (Binary JSON)。
* 物件格式：collection 不用預先定義 sechma，也就是 document 想塞什麼物件都可以。

#### 插入資料
修改 `POST /api/echo`，利用 **Immediately-Invoked Function Expressions (IIFE)** 立刻執行 `async function()`，寫入完成後才回應 request

``` javascript
router.post('/api/echo', function (req, res, next) {
  const body = req.body;

  // 寫入 MongoDb
  const worker = (async function (data) {
    const db = client.db(dbName);
    const collection = db.collection('echo');
    const result = await collection.insertOne(data);
    console.log(result);
    return result;
  })(body);

  // 回應
  worker.then(() => {
    res.json(body);
  })
    .catch(next); // 發生 error 的話，next() 交給之後的 middleware 處理，express 有預設的處理方法
});
```

#### 確認資料有寫入
1. shell 連入 mongodb
    ``` shell
    mongo 127.0.0.1:27017/myproject
    ```
1. 查看 `echo` 內的 document 
    ``` javascript
    db.echo.find()
    ```
    ![Screen Shot 2018-10-15 at 10.57.45 AM.png](resources/40459C6C80889ED4341F52A0CF042A4F.png =480x95)

### 怎麼查文件
可能的操作很多，自己查文件比較有效率，所以說說我怎看文件的，提供剛學習的人一些方向

一進入[MongoDB Node.JS Driver](https://mongodb.github.io/node-mongodb-native/?jmp=docs)，會看到 Reference 和 API
![Screen Shot 2018-10-15 at 11.04.19 AM.png](resources/DCCC4B74E643431DD39F89ED7B639CB1.png =1045x494)

1. Reference：你可能要學習使用，就選這個
    ![Screen Shot 2018-10-15 at 11.10.10 AM.png](resources/9B76BA79C040663B689954AF79179534.png =1055x453)
    * Quick Start/Getting started：大部分的文件都會有這個，給想要立刻使用、體驗，不需要太多的預備知識。沒概念的人或新手就選這個玩一玩。
    * Tutorials: 學習一些必要知識的教學
        ![Screen Shot 2018-10-15 at 11.12.54 AM.png](resources/E4631C7FDA077D896FC79827692FCE01.png =249x408)
1. API：你已有使用概念，但想要查函數的定義/簽章或看看還有提供什麼函數？
    ![Screen Shot 2018-10-15 at 11.14.54 AM.png](resources/48779F9FDAE96FDA6DC0A567EFC55488.png =736x690)
    
    還可以查函數名，輸入 `insert`，就可以查到 `Collection`類別的 `inserOne` 函數定義
    ![Screen Shot 2018-10-15 at 11.17.39 AM.png](resources/98FB1A5A478C17F435A2E7D9C25ADA37.png =1446x517)
    
    例如：我要插入資料時
    1. 怎麼拿到 collection？ `db.collection('echo')` 回傳 collection 物件，不是 promise
    1. 怎麼拿到 db：`client.db(dbName)`　回傳 db 物件，不是 promise
    1. 就可以插入資料
        ``` javascript
        const db = client.db(dbName);
        const collection = db.collection('echo');
        const result = await collection.insertOne(data);
        ```
    我寫文章的當下也是在查文件、看用法、看函數的簽章，不是憑空寫的

查文件的技巧是要練習的，也會越來越快。若沒概念就找 `Quick Start/Getting started/Tutorials`，或用 Google 找關鍵字看別人的用法，最後在學習查 API 文件或看官方教學文件。

> 另外，有個叫 [Dash](https://kapeli.com/dash)(for mac, iOS)的軟本可以查文件用(和程式碼自動輸入)，也是可以用用看。我自己是用不習慣，所以我還是喜歡去官網看文件。

接下來，我們最後介紹 object modeling 套件`mongoose` 做為結尾。

# 使用 object modeling 套件：`mongoose`
有沒有發現 MongoDB Node.JS Driver 的 github 網址是 [https://github.com/mongodb/node-mongodb-native](https://github.com/mongodb/node-mongodb-native)？ `node-mongodb-native` 是官方提供操作 mongodb 用的底層套件，它提供的函數很底層，像我們之前的例子：

``` javascript
const db = client.db(dbName);
const collection = db.collection('echo');
const result = await collection.insertOne(data);
```
要一個個串起來使用。

當後端肥大起來似乎沒這麼方便，為了提供更高層級的思維，就有人寫出 object modeling / ORM(Object-Relational Mapping) / ODM(Object Document Mapper) 這類的套件，甚至不同資料庫、不同的程式語言大多有。
```
為了把資料(document/record)當做是物件，當操作物件時，套件會自己使用底層套件與資料互動。
```
你可以想想以自己寫 SQL 的經驗，ORM套件使你從串SQL文字解脫，變成物件的操作。但也不是這麼美好，因為你又要多學一個套件，不同的 ORM 也可能有不同的用法。

要不要用見人見智(見：[Should I Or Should I Not Use ORM ?](https://medium.com/@mithunsasidharan/should-i-or-should-i-not-use-orm-4c3742a639ce))，我是覺得要用，寫起來比較快，但也要學會用底層套件，因為當你預到效能問題或ORM 套件有bug，才有解套的可能。我的學習策略是先用底層套件 `mongodb`，等用一陣子才引入 `mongoose`，你會了解用/不用 `mongoose` 後的酸甜苦辣。

## Quick start
這例子來自[mongoose](https://mongoosejs.com/) 官方首頁， 直接表明了核心想法
``` javascript
// 建立連線
const mongoose = require('mongoose');
const dbName = 'myproject';
mongoose.connect(`mongodb://localhost:27017/${dbName}`); // resolve with Connection value

// 定義 "Cat" 的模型(也可以說是Schema)，預設的 collection 的名稱會用復數、小寫，所以是 "cats"
const Cat = mongoose.model('Cat', { name: String });

// 建立物件
const kitty = new Cat({ name: 'Zildjian' });

// 存入資料庫
kitty.save().then(() => console.log('meow'));
```

有幾點要知道一下：
1. `mongoose` 有一個預設的 connection，`mongoose.connect()` reolve value 是 Connectino 物件。若需要建立更多的連線可以用 `mongoose.createConnection()`(見：[Multiple connections](https://mongoosejs.com/docs/connections.html#multiple_connections))
1. CRUD 可能有型如：`xxx()`，`xxxMany()`，`xxxOne()`
1. `find()` 是回傳的 `Query`，不是資料本身
1. Query 和 Aggregate　使用時可能會用鍊式語法，最後才用 `exec()` 實際送出查詢

# 總結
今天我們利用 Docker 架設 MongoDB，還用 Node.js Driver 連入後插入資料。最後介紹 object modeling 套件 mongoose，方便我們對 MongoDB 操作。
