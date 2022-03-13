+++
title = "Day 19 - 二周目 - 帳密認証與JWT (JSON Web Token)傳遞"
date = "2018-10-19"
description = "帳密認証與JWT、利用 middleware，認証 JWT middleware "
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

帳密認証與JWT、利用 middleware，認証 JWT middleware 

<!--more-->

# 回憶
昨天完整的介紹了 router 路由，它幫助我們輕易的組裝出 API。完成此目的三個核心概念是 **中間件(middleware)**、**路由(routing)**、**流(stream)**，其中 middleware 是更加的重要。

因為 middleware 可以做很多事，例如：
1. 任易串接在 router 中，想放哪都可以：因為 middleware 簽章都長的一樣，可以放在 `router.use(middleware` 、 `router.METHOD(path, middleware)`、 `app.use(middleware` 和 `app.METHOD(path, middleware)`中。
1. 共用處理流程：放在較上層的路由中可以使下層所有路由都作用到。像是 `app.use(express.json())`放在越前面，可以使後面串進的 router 都可以被 `express.json()` 這 middleware 作用(`req.body` 解析成 JSON object)。
1. middleware 可以獨立出套件：當 middleware 的相依越少，越不會有 [Side Effect](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%89%AF%E4%BD%9C%E7%94%A8)，就可以獨立出來做成套件供別人使用。
1. middleware 命名直覺增加可讀性：像`express.json()`就是跟 JSON 有關，越直覺的命名可以增加程式碼可讀性。下面雖然串更多 middleware，但意思很明顯
    ``` javascript
    router.post('/api/accounts', LogMiddleware, AuthMiddleware, AddAccountMiddleware, WrappedDataEndpointMiddleware)
    ```

# 目標

1. 基本帳密認証
2. 什麼是JWT？
3. 利用 middleware，認証 JWT middleware


# 帳密認証


## 基本的帳密認証(token 版)

我們採用的模式是，認証完成後回應 token 給 client，client 自己留著，每當client要對後端操作時會帶著 token 一併給後端，而不是用在後端保留登入資訊(session)。
這樣做有個好處是後端是無狀態的(stateless)，任何一個台可以識別 token 的後端都可以服務 client，以提高後端的可擴展性(scalability)。

![30天鐵人-Day19-auth.png](resources/62D256A4692DB8F5925C73AF7C6122C9.png)

基本的認証機制有幾個步驟(與上圖對應)
1. 使用者送出帳號、密碼
2. 後端到資料庫比對
3. 資料庫回傳用戶資料
4. 回傳 token

雖然看起來很單純但在這些環節中有些資安注意事項

1. 使用者送出帳號、密碼：
    1. 儘量使用 https 連線，request message 會被加密傳送(包含網址)，攔截封包的人最多只能看到 hostname。 (其實，還有個人會知道，就是 Chrome，因為你是用它的瀏覽器)    
    1. 儘量不要明碼儲存密碼，可以經過「不可反解」加密後(ex: sha256)才存或傳送，因為 [Web Storage](https://www.w3schools.com/html/html5_webstorage.asp) / cookie，是可以透過 javscript 存取查看的
    1. 能不儲密碼就不要存，就算是加密密碼它也是密碼阿! 若要持續性登入，可以用存 token 代替
1. 後端到資料庫比對：拿到帳密，就去資料庫比對
    1. 依需求決定：後端與資料庫連線要不要加密連線或資料庫要不要設帳密
    2. 儘量不要明碼儲存密碼在資料庫中
1. 資料庫回傳用戶資料：
    1. 用戶資料要確保把敏感資料過濾，也可以在後端一收到用戶資料立刻過濾敏感資料，減少敏感資料外洩的可能性
1. 回傳 token
    1. token 可以存在 client cookie 中，並設定 `httpOnly`(Cookie只能被伺服端存取，client 無法用 javascript 讀取)、`secure`(只能透過https的方式傳輸)
    1. 若用上述存在 cookie 中，也不用刻意帶 token 到未來發出的 request 中(少寫一些程式)，瀏覽器會自動帶入

## Token 像什麼？

token 是經過認証單位認証後，所簽發(sign)的字串。 client 拿到 token 後當要求後端服務時一併送出，後端就可以依 token 識別身份給與服務。

### JWT (JSON Web Token) 是什麼？
我們要採用 [JWT (JSON Web Token)](https://jwt.io/) 做為 token 的資料格式。

它是由三個部分組成的

```
標頭(Header).內容(Payload).簽名(Signature)
```

1. 標頭(Header)：Base64編碼的字串。一般內含兩個屬性：token 類型、雜湊(hashing)函數的名字(ex: HMAC SHA256 or RSA)，如：
    ``` json
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```
    再透過 [Base64Url 編碼](http://www.utilities-online.info/base64/#.W8nIVBMzZQI)，一般在轉換前，會把不可見字元(ex: 空白, 換行)拿掉
    ```
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
    ```
    就是 Header。
2. 內容(Payload)：Base64編碼的字串。內含一堆 Claims，像是：(截錄自[JSON Web Token (JWT) 簡介](https://mozillazg.com/2015/06/hello-jwt.html))
    * `iss`: The issuer of the token，token 是給誰的
    * `sub`: The subject of the token，token 主題
    * `exp`: Expiration Time。 token 過期時間，Unix 時間戳記
    * `iat`: Issued At。 token 建立時間， Unix 時間戳記
    * `jti`: JWT ID。針對當前 token 的唯一標識
    
    上面只列出一些 JTW 定義的 claims，其它見 [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)。你也可以自己放任何的資料。來個 Payload 可能資訊
    ``` json
    {
      "sub": "1234567890",
      "name": "John Doe",
      "iat": 1516239022
    }
    ```
    再透過 [Base64Url 編碼](http://www.utilities-online.info/base64/#.W8nIVBMzZQI)，一般在轉換前，會把不可見字元(ex: 空白, 換行)拿掉
    ```
    eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
    ```
    就是 Payload。
    
    到目前為止的 Header 和 Payload 雖然是 Base64Url 編碼後，但它們 **可以解碼**，所以也算是明文資料，不應該放敏感資料。
1. 簽名(Signature)：拿 Header、Payload和一個密鑰(secret)當參數，經過**不可反解**的雜湊函數後得到。以 HMAC SHA256 來說
    ```
    HMACSHA256(
      base64UrlEncode(header) + "." +
      base64UrlEncode(payload),
      secret)
    ```
    這裡 header, payload是指未經過 Base64Url 編碼, secret 是 `your-256-bit-secret`，產生：
    ```
    SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    ```
最後的 JTW 就是
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
[JWT首頁](https://jwt.io/) 有 JTW 加解密互動網頁可以玩玩看。

要注意，這個 **secret 要好好保存在後端**，這是拿來判斷 JWT 是否有效的鑰匙。

### JWT 帶來什麼好處？

1. 自帶狀態：自帶JWT 中可以自帶狀態，不用把狀態存在後端，也不會增加後端負擔。適合運行在 **無狀態** 架構的後端。若是把 token 過期時間放在 payload 中，就可以為 token 加入效期特性。
2. 自驗性：只要有同樣的 secret ，就可以判斷 JWT 是否有效，只要簡單在做一次
    ```
    HMACSHA256(
      JWT.header + "." +
      JWT.payload,
      secret)
    ```
    看結果和 Signature 一不一樣，就可以判斷 JWT 是否有效。
3. 易變性／完整性(integrality)：跟 hashing 函數一樣，只要 Header 或 Payload 的內容改一點點，Signature 的結果就有很大的改變，所以保証資料的完整性。

最後要注意一件是：
```
JWT 解決的是簽証(sign)安全，不是傳輸全安全，要配合加密通道(ex: https)才能安全地傳遞 JWT。
```

### JWT 要自己寫嗎？
[JWT首頁](https://jwt.io/) 有不同程式語言的實作套件，直接用就可以了。
Node.js 用的是 [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)。

# 實作自己的認証
> 過程請見 github commit log [ithelp-30dayfullstack-Day19](https://github.com/eugenechen0514/ithelp-30dayfullstack-Day19)
    執行前需要先開 mongodb (`npm run startdb`)，不用時可以關 mongodb (`npm run stopdb`)

我們例用 JWT 來實作認証機制
1. `POST /api/auth/login` 這 api 是用來登入
2. `POST /api/echo` 這 api 需要有 token 才能運作，利用middleware 輕鬆掛入


## 實作 `POST /api/auth/login`

1. 加入 `./routers/AuthRouter.js` 專們處理受權相關
    ``` javascript
    const express = require('express');
    
    /**
     * 
     * @param {object} dependencies
     */
    function createRouter(dependencies) {
        // Get dependencies
        const { } = dependencies;
    
        // Create a router
        var router = express.Router();
    
        /* POST log */
        router.post('/login', function (req, res, next) {
            next(new Error('Not implement'));
        });
        return router;
    }
    
    module.exports = {
        createRouter
    };
    ```
1. 串入 root router
    串入root router (不懂的請見：[Day 18 - 二周目 - 剖析 express 路由(router) 三概念：中間件(middleware)、路由(routing)、流(stream)](https://ithelp.ithome.com.tw/articles/10202754))
    ``` javascript
    // routers/index.js
    router.use('/api/auth', authRouter);
    ```
    
    authRouter 物件設定 (不懂的請見：[Day 17 - 二周目 - 依賴注入與組態化專案](https://ithelp.ithome.com.tw/articles/10202513))
    ``` javascript
    // app.js
    const { createRouter: createAuthRouter } = require('./routes/AuthRouter');
    
    container.register({
      ...略
      authRouter: asFunction(createAuthRouter, { lifetime: Lifetime.SINGLETON }),
    });
    ```
    
    Postman 打看看
    ![Screen Shot 2018-10-19 at 9.23.01 PM.png](resources/2C88EE51C728E80DF1828481EB66678D.png)
1. 帳密比對實作
    我們假設 `verifyUser()` 會做資料庫查詢
    ``` javascript
    // routers/AuthRouter.js
    async function verifyUser(data) {
        const username = _.get(data, 'username');
        const password = _.get(data, 'password');
    
        if(username === 'billy' && password === '1234') { // pass
            return Promise.resolve({
                username,
                email: 'billy@gmail.com',
            });
        }
        return Promise.reject(new Error('Fail'));
    }
    ```
1. 套用帳密比對
    ``` javascript
    // routers/AuthRouter.js
    router.post('/login', function (req, res, next) {
        const data = req.body;
        verifyUser(data)
            .then(user => {
                res.json(user);
            })
            .catch(next);
    });
    ```
到目前我們做出簡單的帳密驗証。接下來，我們要為驗証成功的 client 回傳 JWT

### 回傳 JWT
我們約定把 JWT 儲存在client 的 cookie中

1. 安裝 [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)
    ``` shell
    npm install jsonwebtoken --save
    ```
1. 當帳密比對成功，就產生 JWT 並回傳
    ``` javascript
    // routers/AuthRouter.js
    const EXPIRES_IN = 10 * 1000; // 10 sec
    const SECRET = 'YOUR_JWT_SECRET';

    router.post('/login', function (req, res, next) {
            console.log(JSON.stringify(req.cookies));　// 印出 cookies
            const data = req.body;
            verifyUser(data)
                .then(user => {
                    const token = jwt.sign(user, SECRET, { expiresIn: EXPIRES_IN });
                    res.json({
                        token
                    });
                })
                .catch(next);
        });
    ```
    這裡用 `expiresIn` 選項可以方便地指定簽發的 JWT 多久到期，像我們設 10 秒。
1. 試打看看
    ![Screen Shot 2018-10-19 at 9.56.39 PM.png](resources/9782569A0A7150B227675D494DEBD681.png)
    我們就可以得到 JWT
1. 強化安全性
    加入這行，後端要求 client 設定 cookie
    ``` javascript
    res.cookie('token', token, { maxAge: EXPIRES_IN, httpOnly: true}); // 回應 client ，把 token 存在名為 token 的 cookie 並設定相關屬性
    ```

再試打一下，查看 client(Postman) 收到回應的 headers
![Screen Shot 2018-10-19 at 10.41.03 PM.png](resources/67EBA22B24F3AD1B269052064459317A.png)
`HttpOnly` 設定時，cookie `token` 不能用 javascript 取出。但你可以在 Postman 的 **Cookies**，可以查看所有 cookies，你會看到 `token` 被設定
![Screen Shot 2018-10-19 at 10.45.28 PM.png](resources/6E9F688FBB818E197957F81CF6D4A7AA.png)

另外，觀察到：
* 因為我們設定 cookie 10 秒到期(`EXPIRES_IN`)，所以時間到後再看一次就會消失。
* 之後的 request 會一直帶著 cookies 一併送給後端
  ![Screen Shot 2018-10-19 at 10.54.54 PM.png](resources/886A88136B411FD0398BE31DB9AE61F5.png)
  第一次打，沒有 cookies 所以印出 `{}`，第二次打，因為前一次登入成功並設定cookies，所以就會有 cookies。


## 為 `POST /api/echo` 加入 token 驗証
假設 client 會把 JWT 放在 名為 `token` 的 cookie 中，所以後端可以由
```
req.cookies.token
```
得到來自 client 的 token。因此，我們只要驗証此 token 就可以知道，request 是否有授權。

我們利用 middleware 來做 JWT 的驗証

1. 加入 `VerifyJWT`，當 `VerifyJWT()` 動態產生 middelware
    ``` javascript
    // middlewares/VerifyJWT.js
    const _ = require('lodash');
    const jsonwebtoken = require('jsonwebtoken');
    
    const SECRET = 'YOUR_JWT_SECRET'; // 要和簽發時一樣，所以可以放在 ./configs/config.js 中
    
    async function verifyJWT(jwt) {
        if (!jwt) {
            return Promise.reject(new Error('No JWT'));
        }
        const decoded = jsonwebtoken.verify(jwt, SECRET);
        return decoded;
    }
    
    
    module.exports = function (options = {}) {
        const {tokenPath = 'cookies.token'} = options; // tokenPath 是取出 token 的路徑
        return function (req, res, next) {
            const jwt = _.get(req, tokenPath);
            verifyJWT(jwt)
                .then(decoded => {
                    console.log(decoded);
                    next(); // next middleware
                })
                .catch(next);
        };
    }
    ```
    我們很刻意的利用閉包技巧，輸入 `tokenPath` 來動態產生 middelware。使用時，`VerifyJWT()` 才是 middelware 的簽章。
1. `POST /api/echo` 掛入 JWT 驗証
    ``` javascript
    router.post('/api/echo', VerifyJWT(), function (req, res, next) {
      ...略
    }
    ```

這樣就完成對`POST /api/echo` 掛入 JWT 驗証，要有 JWT 才能執行這支API。

# 總結

今天介紹基本認証的機制，還利用 JWT 來傳遞驗証結果。最後，利用 middleware 可以方便的掛入需要驗証的 APIs.

未來有機會在來談 [passport.js](http://www.passportjs.org/)、[OAuth](http://blog.kenyang.net/2012/09/15/google-oauth-20-google-api)、[Time-based One-Time Password(TOTP)二階段驗証](http://blog.brandonc.me/2012/01/totp-google-authenticator-two-factor.html)。
