+++
title = "Day 20 - 二周目 - 軟體測試(Testing)：測試非同步函數"
date = "2018-10-20"
description = "Jest 寫單元測試/整合測試"
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

Jest 寫單元測試/整合測試

<!--more-->

# 回憶
昨天我們談了如何做驗証和掛入 middleware 來驗証 token。今天要著重測試這塊

想必大家一定常聽到 [自動軟體測試、TDD 與 BDD](https://medium.com/@yurenju/%E8%87%AA%E5%8B%95%E8%BB%9F%E9%AB%94%E6%B8%AC%E8%A9%A6-tdd-%E8%88%87-bdd-464519672ac5)這之類的名詞，這此觀念現在很流行，不過我們不是要講它們，有興趣的人可以看 [91 - 30天快速上手TDD](https://dotblogs.com.tw/hatelove/archive/2013/01/11/learning-tdd-in-30-days-catalog-and-reference.aspx)。

# 目標

今天的目標是拿我們昨天的程式來寫測試：

1. 使用 Jest 進行測試
1. 如何寫非同步程式的測試
1. 怎麼寫單元測試/整合測試

# 寫測試嗎？

## 為什麼要寫測試？

寫測試的好處很多，上網可以查到一堆，但我說說我目前有體會到的好處
1. 提升自信心：當你寫好一後函數，你是怎麼測試有沒有正常運作呢？當然是執行看看。測試就是在執行你的函數，你可以輸入一堆參數來執行你的函數，有被執行過的函數會比較放心。
2. 思考介面：
    * 對於還沒寫實作(production code)的函數前，你會考慮這函數的介面(簽章)以釐清目地。
    * 對於已寫實作的函數，引入測試時，你會重新省審視有沒有足夠的參數讓測試執行函數。
3. 停止無止盡的規畫：
    * 測試要求寫下預期的執行結果，所以你會針對目標寫出實作(production code)使測試通過，而不是陷在思考邏輯中。
    * 引起函數的執行可能要很長的前置作業，像是先執行函數A，再執行函數B後，最後才是你的函數。這路徑太長，所以你會調整介面(簽章)使得函數可以直接執行。
4. 保護程式碼：開發時是不是常出現「改東壞西」？已通過的測試會保護程式碼，每當你改完某程式，只要重新執行所有測試，就可以看以前通過的測試現在有沒有失敗，所以可以鎖定錯誤範圍、降低維謢成本。
5. 更完備程式碼(強健程式碼)：有了直接執行的手段，就可以輸入不同的參數跑看看你的程式會不會壞掉，所以可以加入更多確保運作的程式碼，像是空參數的檢查…等。

結論是：
```
寫測試是以目地為導向，讓測試隨時執行監控程式碼
```

## 不寫，會不會怎麼樣？我真的需要寫嗎？

其實我覺得也還好，就自己從重頭手動測試也可以。不過我為了前面提的好處，我會儘量寫，但速度和維謢性一直會互相拉扯。畢竟有些老闆要的是能跑的程式，不是能維謢的程式，但要讓他知道趕出來的程式通常有後續維謢的成本(像不容易加入新功能、程式碼太亂、命名過亂、程式架構混亂…之類的，常見的原因是「相依項太多」)。

當目標不明確時，我也不太會寫測試，老闆要什麼他自己都不知道了你會知道嗎？但是我在開發時會儘量抽工具和套件，或做重構的準備(ex:  寫註解、命名故意寫的又長又精確)，也會儘量把工具類(utilities)、套件庫(package/libraries)的測試寫好。

有時測試是不用寫或無法寫，像我之前文章的程式我也不會寫，因為是教學用。有時我要找什麼當範例我也不知道，能不能正常運作我也不知道，合不合適也不知道，都是要實作跑看看再改範例程式。

比起測試，我覺得重構、除錯技巧和使用工具輔助它們反而更常使用。

# Jest 測試框架
> 過程請見 github commit log [ithelp-30dayfullstack-Day20](https://github.com/eugenechen0514/ithelp-30dayfullstack-Day20)

[Jest](https://github.com/facebook/jest) 是 Facebook 底下的一個專案，提供：
1. 測試環境 API，如：`test(name, fn, timeout)`
2. Expect/Assertions API，如：`expect(actualData).beBe(expectData)`
3. 替身 Mock/Spy/Sub API，如：
    ``` javascript
    const mockFn = jest.fn();
    mockFn();
    expect(mockFn).toHaveBeenCalled(); // ture
    ```
1. 除了從套件執行 Jest, 還提供 Jest  CLI(command line intreface)，有 `jest` 指令 ，如：`jest path/to/my-test.js`

直接來個範例：

## 一個簡單的 Jest 執行

我們選擇從專案本地執行，不想每次都安裝的話，就用 `npm install jest -g` 裝在全域

1. 安裝 Jest
    ``` shell
    npm install jest --save-dev
    ```
    因為跟程式運作無關，jest 是屬於開發時才會運作，所以放 `devDependencies` 就可以了
2. 加入 npm 的腳本執行：加入名為 `test` 腳本執行 `jest`，因為是由 `npm run` 執行，所以會執行 `./node_modules/jest/bin/jest.js`，不是全域的 `jest`。
    ``` javascript
    // package.json
    {
      ...略
      "scripts": {
        ...略
        "test": "jest",
        "jest-version": "jest --version"
      },
      ...略
    }
    ```
    這裡我們只使用執行 `jest` 指令，它預設會從搜尋當前目錄下所所有的測試檔，檔案命名符合 [`**/__tests__/**/*.js?(x)`, `**/?(*.)+(spec|test).js?(x)`](https://jestjs.io/docs/en/configuration#testmatch-array-string) 比對的檔案進行測試。 Jest 有提供很多組態設定 [Configuring Jest](https://jestjs.io/docs/en/configuration)，一般會組態放在 `package.json` 中的 `jest` 屬性值中，或獨立出一個檔案執行時用 `jest --config` 引入。
    
    另外，`npm run jest-version` 可以看到現在 Jest 的執行版本。
3. 在專案根目錄，建一個 `__tests__` 資料夾，後加入一個檔案 `./__tests__/hello-jest.test.js`
    ``` javascript
    function sum(a, b) {
        return a + b;
    }
    
    test('adds 1 + 2 to equal 3', () => {
        expect(sum(1, 2)).toBe(3);
    });
    ```
    這是最簡單的測試。另外， Jest 也支援測試群組 `describe/afterAll/afterEach/beforeAll/beforeEach` 之類 [Globals API](https://jestjs.io/docs/en/api)。
4. 執行測試 `npm run test`
    ![Screen Shot 2018-10-20 at 2.06.05 PM.png](resources/2A09DD98AFB153316416EDEFA35A509D.png)
    會看到它找到一個測試(tests)

## 測試非同步函數： 測試 `VerfiyJWT()` 產生的 middleware
昨天我們用 `VerfiyJWT()` 產生的 middleware 來驗証 JWT 是否有效，我們也可以對 `VerfiyJWT` 寫測試。
開始前先開一個 `./middlewares/VerfiyJWT.test.js` 檔案，我們把測試往這放。

我們一步步看 `VerfiyJWT` 是什麼東西，請先不要看實作，實作會使你搞不清楚你的測試對象和目地

### 回傳值測試
首先 `VerfiyJWT` 他用來產生 middleware，所以他的簽章是： `VerfiyJWT() : function`，所以
```
VerifyJWT() 要回傳一個 function
```
這就是第一個測試
``` javascript
const VerifyJWT = require('../VerifyJWT');

describe('VerifyJWT', () => {
    test('VerifyJWT() 要回傳一個 function', () => {
        const middleware = VerifyJWT();
        const actual = typeof middleware;
        const except = 'object';
        expect(actual).toEqual(except);
    });
});
```

你可以刻意的把 `except` 改成值 `object`，就會測式不通過。

![Screen Shot 2018-10-20 at 2.37.21 PM.png](resources/1075EC0BAE735B9DAD927E71C4CC3BF6.png)

我們可以看出幾點：
1. 測試對象： `VerifyJWT`
2. 測試描述： `VerifyJWT()` 要回傳一個 function
3. 實際輸出： 放在 `actual`
4. 期望輸出： 放在 `except`

### 回傳 middleware 的測試
上面處理完回傳了，我們來看 middleware 預期的效果： `middleware 會從 cookie 拿 token，檢查 token 是否有效`。

所以
```
當 request 的 cookie token 值為有效，就會換下一個 regular middleware
```
就是一個測試項目：

``` javascript
    test('當 request 的 cookie token 值為有效，就會換下一個 regular middleware', (done) => {
        const validJWT = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImJpbGx5IiwiZW1haWwiOiJiaWxseUBnbWFpbC5jb20iLCJpYXQiOjE1NDAwMTg2ODQsImV4cCI6MTU0MTAxODY4NH0.BHWWQbxWeaeCRLgjmT9xoAD_LXTGkmX6zcTKoncv_Uk';
        const middleware = VerifyJWT();

        const res = {};
        _.set(res, 'cookie.token', validJWT); // 用 lodash 的　set() 設定值，相當於 const res = {cookie: {token: validJWT}}
        
        middleware(res, null, () => {
            done(); // 告訴 Jest 此測試正常結束
        });
    });
```
我們真的找一個有效的 token 放在 `req.cookie.token` 中，middleware 應該要叫起 `next()`。
這裡我們用到 `done` 這參數，這是 Jest 在非同步測時需要呼叫的，若執行 `done()` 就是跟 jest 說此測式正常結束了，若執行 `done(error)` 就是出問題了。假如你不叫 `done()` 或是 middleware 中 沒叫 `next()` 就會測試失敗。
![Screen Shot 2018-10-20 at 3.06.57 PM.png](resources/54868B6C34BE9796302E8B22BD6A9BA8.png)

> 除了這種 callback的版本，其它對於非同步的測試還有 Promise, async/await的版本[Testing Asynchronous Code](https://jestjs.io/docs/en/asynchronous)。

反之，
```
當 request 的 cookie token 值為無效，就會換下一個 error-handling middleware
```
就是另一個測試：
``` javascript
    test('當 request 的 cookie token 值為無效，就會換下一個 error-handling middleware', (done) => {
        const invalidJWT = 'invalidJWT.invalidJWT.invalidJWT';
        const middleware = VerifyJWT();

        const res = {};
        _.set(res, 'cookies.token', invalidJWT); // 用 lodash 的　set() 設定值，相當於 const res = {cookie: {token: validJWT}}
        
        middleware(res, null, (error) => {
            expect(error).toBeInstanceOf(Error); // error 非空(即 next(error)), 測試通過
            done(); // 告訴 Jest 此測試結束
        });
    });
```

###  `VerfiyJWT` 測試的小總結

我們寫了三個測試：
1. `VerifyJWT()` 要回傳一個 function
2. 當 request 的 cookie token 值為有效，就會換下一個 regular middleware
3. 當 request 的 cookie token 值為無效，就會換下一個 error-handling middleware

你可以任意的改動程式，看看各種錯誤訊息。

但是上面的測試有一些問題：
1. 有效的 JWT，會因為到期失效：因為我們是 hard code 寫的 token，失效後測試就會失敗
2. secret 變的話就測試也會失敗

這是因為 `VerifyJWT` 對 `jsonwebtoken` 套件和 `SECRET` 常數有相依關係，所導致的問題，所以我們稱 (我們忽略 `lodash` 套件，把它當原生的)
```
當 request 的 cookie token 值為無效，就會換下一個 error-handling middleware
當 request 的 cookie token 值為有效，就會換下一個 regular middleware
```
是 **整合測試(integration test)**。

然而 `VerifyJWT() 要回傳一個 function` 不相依其它人，所以它是 **單元測試(unit test)**

## 整合測試轉成單元測試

要把相依的問題解決，常用的方法就是 **依賴注入(DI)**。

我們先把上面三個分成兩個檔案：整合測試的群叫 `VerifyJWT.integration.test.js`，單元測試的群叫 `VerifyJWT.unit.test.js`。

### 處理 `SECRET` 相依：`SECRET` 改成外界送入
1. `options` 取出外界送入的 `secret` 屬性，預設值是原來的 `SECRET`
    ``` javascript
    const {tokenPath = 'cookies.token', secret = SECRET} = options; 
    ```
1. 送入 `verifyJWT()` 且修改它的簽章和使用 secret
    ``` javascript
    async function verifyJWT(jwt, {secret}) {
      if (!jwt) {
          return Promise.reject(new Error('No JWT'));
      }
      const decoded = jsonwebtoken.verify(jwt, secret);
      return decoded;
    }
    
    verifyJWT(jwt, {secret})
    ```
1. 再執行一次測試，還是通過，表示沒有程式壞掉
    ![Screen Shot 2018-10-20 at 4.08.30 PM.png](resources/D4C2771E5591E0087ABB8F16256EF90C.png)

### 處理 `jsonwebtoken` 相依：`jsonwebtoken.verify` 改成外界送入
1. 類似上面的操作，`options` 取出外界送入的 `verify` 屬性，預設值是原來的 `jsonwebtoken.verify`
    ``` javascript
    const {tokenPath = 'cookies.token', secret = SECRET, verify = jsonwebtoken.verify} = options; 
    ```
1. `verify` 送入 `verifyJWT()` 並使用
    ``` javascript
    async function verifyJWT(jwt, {secret, verify}) {
        if (!jwt) {
            return Promise.reject(new Error('No JWT'));
        }
        const decoded = verify(jwt, secret);
        return decoded;
    }
    ```
1. 再執行一次測試，還是通過，表示沒有程式壞掉

### 修改成單元測試

1. 複製二個整合測試
2. 把依賴整改成在測試中注入
    ``` javascript
      const jsonwebtoken = require('jsonwebtoken');
      const MY_SECRET = 'MY_SECRET'; 

      test('當 request 的 cookie token 值為有效，就會換下一個 regular middleware', (done) => {
            const validJWT = jsonwebtoken.sign({}, MY_SECRET); // 產生有效的 token
            const middleware = VerifyJWT({secret: MY_SECRET, verify: jsonwebtoken.verify}); // 依賴注入
    
            const res = {};
            _.set(res, 'cookies.token', validJWT);
            
            middleware(res, null, (error) => {
                done(error);
            });
        });
    
        test('當 request 的 cookie token 值為無效，就會換下一個 error-handling middleware', (done) => {
            const invalidJWT = 'invalidJWT.invalidJWT.invalidJWT';
            const middleware = VerifyJWT({secret: MY_SECRET, verify: jsonwebtoken.verify}); // 依賴注入
    
            const res = {};
            _.set(res, 'cookies.token', invalidJWT);
            
            middleware(res, null, (error) => {
                expect(error).toBeInstanceOf(Error);
                done();
            });
        });
    ```
1. 再執行一次測試，增加的兩個單元測試也會通過

### 轉成單元測試的小總結
我們成功透過依賴注入完成了 `VerifyJWT` 的單元測試，什麼時候執行都會有一樣的結果。同時，`VerifyJWT(options)` 也變的更有彈性，像是 `verify` 的實作再也不限定是 `jsonwebtoken.verify`，可以套用其它 JWT 的驗証套件。另外，別忘了之前的整合測試過了一段時間就會失敗。

我們自然想問一個問題：

### 我該寫整合測試還是單元測試？

這問題應該是因人而異，我列出一些我的看法

* 貼近終端目地：整合測試的測試目標比較能夠貼近終端使用者的目標，而單元測試比較碎片化。
* 程式完備性/健壯性：單元測試的程式一般比較小，所以比較能處理所有情況。
* 寫測試花費的時間：單元測試可能要處理更多的相依，所以可能比較花時間。
* 測試易碎：整合測試相對單元測試容易壞掉，因為他有較多的相依。
* 執行測試時間：單元測試比整合測試快。

在時間有限的情況下，我的原則是套件、工具類的程式最好寫測試。不然就是先寫整合測試，有時間才單元測試，或是針對重要、易錯的程式先寫單元測試。


# 總結
今天主要用 `VerifyJWT` 為例子介紹：
1. Jest 如何使用
1. 如何寫非同步的測試
1. 單元測試和整合測試的區別：看相依關係強度

# 參考資料
以下資料提供有興趣的人研究

* 按照功用分類：
    * 框架：[jest](https://jestjs.io/) 
    * Mock/Spy/Sub 替身類：[Sinon.js](https://sinonjs.org/)、[jest](https://jestjs.io/)
    * Assert/Except: ：[Sinon.js](https://sinonjs.org/)、[chai.js](https://www.chaijs.com/), [jest](https://jestjs.io/), [unexpected](http://unexpected.js.org/)
* 參考文章：
    * [Sinon.js 的 Spies, Stubs 和 Mocks 的最好實踐[譯文+筆記]](https://medium.com/@yujiechen0514/sinon-js-%E7%9A%84-spies-stubs-%E5%92%8C-mocks-%E7%9A%84%E6%9C%80%E5%A5%BD%E5%AF%A6%E8%B8%90-%E8%AD%AF%E6%96%87-%E7%AD%86%E8%A8%98-f799538ffb64)
    * [5 Questions Every Unit Test Must Answer](https://medium.com/javascript-scene/what-every-unit-test-needs-f6cd34d9836d)
    * [Rethinking Unit Test Assertions](https://medium.com/javascript-scene/rethinking-unit-test-assertions-55f59358253f)
    * [JavaScript Testing: Unit vs Functional vs Integration Tests](https://www.sitepoint.com/javascript-testing-unit-functional-integration/)
