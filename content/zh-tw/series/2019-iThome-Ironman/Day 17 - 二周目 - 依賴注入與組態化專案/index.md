+++
title = "Day 17 - 二周目 - 依賴注入與組態化專案"
date = "2018-10-17"
description = "依賴注入套件、組態化"
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

依賴注入套件、組態化

<!--more-->

# 回憶
昨天我們強化了後端專案結構，加入了services、daos的資料夾，這有助於切割商業邏輯，強化維護性。

觀察一下，我們將大部分的物件建立移到 `app.js` 中，方便控制物件相依

``` javascript
const echoDao = new EchoDao({mongoClient: client});
const mongoService = new MongoService({mongoClient: client, echoDao});
const {createRouter: createRootRouter} = require('./routes/index');
const indexRouter = createRootRouter({mongoService});
```

這延申出幾個問題：
1. 新的 service、dao 出現的時候會越來越長
1. 相依關係要自己串連，很不方便
1. 太多的相依可能會忘記
1. 不小心循環引用

# 目標

我們要使用依賴注入套件[awilix](https://github.com/jeffijoe/awilix)解決上面問題。

1. 使用 awilix
1. 組態化 `./configs/config.js`
1. 環境變數 `process.env`

過程請見 github commit log [ithelp-30dayfullstack-Day17](https://github.com/eugenechen0514/ithelp-30dayfullstack-Day17) 或 [codesandbox](https://codesandbox.io/s/github/eugenechen0514/ithelp-30dayfullstack-Day17)

# awilix 依賴注入

先問幾個問題：
* 為什麼要用 awilix？　－　自動處理相依關係，不用手動建立
* 不用會怎樣？　－　不會怎樣，就自己建立物件，相依關系自己處理，有更大的彈性，但也很麻煩
* 有什麼好處？　－　自動處理相依關係、避免循環引用、設定物件的生命期間(Lifetime)、物件建立的程式碼會集中起來

#  使用 awilix

大致上的流程是
1. 建立 container ：awilix container 裡面會放所有「物件建立方法」、物件
1. register/load module ： 在 container 中，註冊「物件建立方法」
    1. `asClass`, `asFunction`, `asValue`: 物件建立的方法，再用 `container.register()` 註冊在 container 中
        ``` javascript
        container.register({
          objName1: asClass(Class),
          objName2: asFunction(factoryFunction),
          objName3: asValue(value),
        });
        ```
    2. `container.loadModules(globPatterns, options)`：`loadModules()` 會掃描資料夾，套用我們設定的「物件建立方法」
1. `resolve(name)` : 取出物件，這時才開始建立相依的物件

所以任何地方只要有能存取 container，就可以透過 `resolve(name)` 取出物件。

## 用 awilix 建立物件關係
1. 先安裝 awilix
    ``` shell
    npm install awilix --save
    ```
1. 註冊部分「物件建立方法」：兩個物件名 mongoClient 和 indexRouter
    ``` javascript
    // app.js
    const { createContainer, asClass, asValue, asFunction, Lifetime } = require('awilix');
    
    // 建立 awilix container
    const container = createContainer();
    
    container.register({
      mongoClient: asValue(client, { lifetime: Lifetime.SINGLETON }), // 註冊為 mongoClient，且生命期為 SINGLETON (執行中只有一個物件)
      indexRouter: asFunction(createRootRouter, { lifetime: Lifetime.SINGLETON }), // 註冊為 indexRouter，利用工廠函數 createRootRouter 建立物件
    });
    ```
1. 掃描 services, daos 資料夾
    ``` javascript
    // app.js
    container.loadModules([
      'daos/*.js',
      'services/*.js',
    ], {
        formatName: 'camelCase',
        resolverOptions: {
          lifetime: Lifetime.SINGLETON,
          register: asClass
        }
      });
    ```
    掃描到的檔案，因為是 `module.exports =  clsss` 所以用 `asClass` 註冊。名稱命名規則為 **camelCase** ，生命期為 **SINGLETON**。例如：找到 `MongoService` 就好像用
    ``` javascript
    // app.js
    container.register({
      mongoService: asClass(MongoService, { lifetime: Lifetime.SINGLETON }),
    });
    ```
    註冊
1. 取出名為 **indexRouter** 的物件
    ``` javascript
    // app.js
    const indexRouter = container.resolve('indexRouter');
    ```
    **indexRouter** 有指定用 `createRootRouter(dependencies)` 建立 (`asFunction(createRootRouter)`)，它會拿 `const {mongoService} = dependencies`， **mongoService** 就會被 `asClass(MongoService)`建立，所以會一直建立相關的物件
    ```
    建立物件路徑:
    createRootRouter({mongoService}) -> mongoService -> mongoClient and echoDao
                                                                            |
                                                                            ∟ ->  mongoClient
    ```

awilix 幫助我們從
```
關注「建立物件」和「串接物件關係」
```
變成
```
關注「物件建立方法」
```
關連性透過命名串接，相依物件自動建立。

# 組態化專案

程式中的 MongoDB 連線有寫死(hard code) 常數值

``` javascript
//  daos/EchoDao.js
class EchoDao {
    ...略
    async insert(data) {
        const dbName = 'myproject';
        const db = this.mongoClient.db(dbName);
        ...略
    }
}
```
和
``` javascript
//  app.js
const MongoClient = require('mongodb').MongoClient;
const url = 'mongodb://localhost:27017';
const client = new MongoClient(url, { useNewUrlParser: true });
...略
```
接下來要將它們抽出到一個 `config.js` 檔案，方便未來修改

## 建立 `./configs/config.js`
提出連線的常數

``` javascript
// configs/config.js
module.exports = {
    mongodb: {
        url: 'mongodb://localhost:27017',
        dbName: 'myproject',
    }
}
```

## 為 `EchoDao` 加入 `config` 相依

1. `EchoDao` 加入 `config` 相依
    ``` javascript
    // daos/EchoDao.js
    class EchoDao {
        /**
         * 
         * @param {object} config
         * @param {MongoClient} mongoClient
         */
        constructor({ config, mongoClient }) {
            this.config = config;
            this.mongoClient = mongoClient;
        }
        ...略
    }
    ```
2. 加入名為 **config** 的物件建立設定
    ``` javascript
    // app.js
    const config = require('./configs/config');
    
    container.register({
      config: asValue(config, { lifetime: Lifetime.SINGLETON }),
      ...略
    });
    ```

## 為資料庫連線加入相依

為了套用控制反轉(Inversion of Control, IoC)，把連線過程放到 `createMongoClient()` 這工廠函數：
1. 建立 `createMongoClient()` 用來產生 MongoClient 物件
    ``` javascript
    // app.js
    /**
     * 
     * @param {object} config
     * @returns {MongoClient}
     */
    function createMongoClient({config}) {
      const url = config.mongodb.url;
      const client = new MongoClient(url, { useNewUrlParser: true });
    
      // 立即連線
      client.connect()
        .then((connectedClient) => {
          console.log('mongodb is connected');
        })
        .catch(error => {
          console.error(error);
        });
        return client;
    }
    ```
1. 改成用工廠函數 `createMongoClient()`
    ``` javascript
    // app.js
    container.register({
      ...略
      mongoClient: asFunction(createMongoClient, { lifetime: Lifetime.SINGLETON }), // 註冊為 mongoClient，且生命期為 SINGLETON (執行中只有一個物件)
      ...略
    });
    ```
1. 預先引起建立 mongoClient，為了可以 **儘早** 連線
    ``` javascript
    // app.js
    // 預先引起建立 mongoClient
    const mongoClient = container.resolve('mongoClient');
    
    const indexRouter = container.resolve('indexRouter');
    ```
    
這樣就大工告成了，剩下的像是：相依注入、建立物件…等會由awilix 幫我們完成

# 從執行外界接受參數

在 [Day 12 - 二周目 - 準備起程深入後端](https://ithelp.ithome.com.tw/articles/10201085) 提過 Node.js 執行 js 檔時是單一行程、單一執行緒。有 golbals 的變數叫 [`process`](https://nodejs.org/dist/latest-v10.x/docs/api/globals.html#globals_process)，它是指當前執行時的 process。 當我們執行 `node ./bin/www` 時 `process` 就會被建立，它是外界串連的橋樑。

## 行程執行環境(process.env)
我們常常希望 `node ./bin/www` 可以從外部設定參數，像是 **PORT** 環境變數，就是會拿來設定 Web Server 會開啟的 port。你可以直接打開檔案`./bin/www` 會看到 `process.env.PORT`，
``` javascript
/**
 * Get port from environment and store in Express.
 */

var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);
...略
```
這裡的 `process.env` 物件是 process 執行時的環境變數(environment)。

因此，下指令 `PORT=3001 node ./bin/www`，前面的 `PORT` 就會被設定在 `process.env.PORT` 中。若要更多環境變數就用「空白」區隔，例如 `MONGODB_URL=http://localhost:27017 PORT=3001 node ./bin/www`

### 為 `config.js` 加入環境變數
利用 `process.env` 就可以從外界修改環境變數，不用動程式
``` javascript
const config = {
    mongodb: {
        url: process.env.MONGODB_URL || 'mongodb://localhost:27017',
        dbName: process.env.MONGODB_DB_NAME || 'myproject',
    }
}
module.exports = config;
```
小技巧：`process.env.MONGODB_URL || 'mongodb://localhost:27017'` 可以在 `MONGODB_URL` 沒設定值時使用預設值  `'mongodb://localhost:27017'`

注意： `process.env.XXX` 是 `undefined` 或 `string`，所以萬一變數是其它的型態要進行轉型，如：
``` javascript
function __defaultFalse(bool) {
    if(bool === "true") {
        return true;
    }
    if(bool === "false") {
        return false;
    }
    return false;
}
const enable = __defaultFalse(process.env.ENABLE);
```

### 為 VSCode debug 執行加入環境變數
若你是透過 VSCode debug 模式執行 js，一樣可以加入環境變數

打開 `.vscode/launch.json`，加入 `env` 參數
``` json
{
  "configurations": [
          {
              "type": "node",
              "request": "launch",
              "name": "www",
              "program": "${workspaceFolder}/bin/www",
              "env": {
                  "PORT": "3001",
              }
          }
      ]
}
```

## 執行參數(process.argv)
[process.argv](https://nodejs.org/dist/latest-v10.x/docs/api/process.html#process_process_argv) 是另一個方法，它是執行參數的字串陣列

範例直接來自官網：

當執行
``` shell
node process-args.js one two=three four
```

`process.argv`陣列如下：

```
0: /usr/local/bin/node
1: /Users/mjr/work/node/process-args.js
2: one
3: two=three
4: four
```

若要讓程式可以輸入參數，我推薦使用 [argparse](https://www.npmjs.com/package/argparse)，可以做 cli (command-line interface) 做的比較完整。

# 總結
今天我們介紹如何用 awilix 做依賴注入，讓我們從 `關注「建立物件」和「串接物件關係」`轉為`關注「物件建立方法」`。另外，用 `process.env` 來組態化專案，藉此來從外界接收設定值。
