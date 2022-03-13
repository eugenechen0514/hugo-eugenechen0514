+++
title = "Day 10- 一周目- 開始玩轉前端(一)"
date = "2018-10-10"
description = "介紹 React、用`create-react-app` 建立，我們第一個前端網頁"
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

介紹 React、用`create-react-app` 建立，我們第一個前端網頁

<!--more-->

# 回憶
昨天後端完成了二隻api`GET /api/asyHi` 和 `POST /api/echo`，以及用 POSTMAN 發出 request 到後端並得到回應。 同時，也提到可以用 Chrome devTools 來監控網路 request。

# 目標
1. 介紹 React
2. 用`create-react-app` 建立，我們第一個前端網頁


# React 是什麼？
React 是 Facebook 所開發的一個專案，現在能如此風行有很大的原因是開放社群，開發出以React 為基礎/相關的套件，然而之前有暴出 license 的問題，使人想要退出 React。2017/09/25 正式的改成 [MIT license](http://lucien.cc/20080117-the-mit-license-mit%E6%8E%88%E6%AC%8A%E6%A2%9D%E6%AC%BE%E4%B8%8D%E8%B2%A0%E8%B2%AC%E4%BB%BB%E4%B8%AD%E8%AD%AF%E7%89%88/)，license 的問題得到了解決(參考：[為什麼Facebook更改了React.js的授權？](https://medium.com/@lovenery/%E7%82%BA%E4%BB%80%E9%BA%BCfacebook%E6%9B%B4%E6%94%B9%E4%BA%86react-js%E7%9A%84%E6%8E%88%E6%AC%8A-a0ba6a2ef97f)、[五種開源授權規範的比較 (BSD, Apache, GPL, LGPL, MIT)](http://inspiregate.com/internet/trends/74-comparison-of-five-kinds-of-standard-open-source-license-bsd-apache-gpl-lgpl-mit.html) )

React 是用來建立使用者介面(UI, user interface)的 javascript 程式庫/套件，它讓我們可以用 `Component` 組成復雜的 UI。看以下[官方](https://reactjs.org/tutorial/tutorial.html#what-is-react)例子：

``` javascript
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}

// Example usage: <ShoppingList name="Mark" />
```
## 用JSX 語法建立 Component
首先，`<ShoppingList/>` 就是一個 Component，他是由「像是」 html tag 的 `<div/>`、`<hi/>`、`<ul/>`、`<li/>` 組成的。怎麼說「像是」，有注意嗎？我們 **不是** 寫文字的 `"<div>...</div>"`，反是直接寫 `<div>...</div>` 這類 XML-like，這就是 [`JSX語法`](https://blog.techbridge.cc/2016/04/21/react-jsx-introduction/)，它是個語法糖衣([Syntatic Sugar](https://en.wikipedia.org/wiki/Syntactic_sugar))，它會被轉譯器(transpiler)轉成 javascript

``` javascript
class ShoppingList extends React.Component {
  render() {
    return React.createElement(
      "div",
      { className: "shopping-list" },
      React.createElement(
        "h1",
        null,
        "Shopping List for ",
        props.name
      ),
      React.createElement(
        "ul",
        null,
        React.createElement(
          "li",
          null,
          "Instagram"
        ),
        React.createElement(
          "li",
          null,
          "WhatsApp"
        ),
        React.createElement(
          "li",
          null,
          "Oculus"
        )
      )
    );
  }
}
```
`React.createElement()` 會回傳 [React element](https://reactjs.org/docs/rendering-elements.html)，它們是組成 React app 最小的區塊，在 React 會維護自己的虛擬 DOM (React DOM)，可以想像成如同 HTML DOM tree 一樣，然後 React DOM 會比較 tree 的差異，進而更新 HTML DOM。

> React 沒有規定一定要用 JSX 語法，只是推薦使用，畢竟最後還是轉成 javascript `React.createElement()`。

## `ShoppingList` component 類別
接下來，`ShoppingList` 他是一個 `Class`，他繼承 `React.Component`，所以就享有 `Component` 的功能，像是、Props、State 和 生命周期(Lifecycle)。

建立 instance(即 React element) 不是用直接 `new ShoppingList()`，而是透過 JSX 語法建立：
``` javascript
<ShoppingList name="Mark" />;
```
將會轉譯
``` javascript
React.createElement(ShoppingList, { name: "Mark" });
```
他會把 `ShoppingList` class 送入，交給 React 建立 `ShoppingList` instance 和遞迴地建立  `render()` 裡面的所有 `<XXX />`。

## 誰是入口點？
最後的問題是：誰來用 `<ShoppingList name="Mark" />`？誰是 React app 的入口點？

在網頁讀取完後，一般會立刻執行 `ReactDOM.render()`，引發進入 React app 的渲染(render)，而更新 HTML DOM。 類似下面

``` html
<html>
<body>
  <div id='root'></div>
  <script type="text/javascript">
    const element = <ShoppingList name="Mark" />;
    ReactDOM.render(element, document.getElementById('root'));
  </script>
</body>
</html>
```
`ReactDOM.render()` 會渲染 **react element** 進 HTML DOM 中的 `<div id='root' >` 容器內，把 React app 和 HTML DOM 串連起來。

## 誰來轉譯 JSX?

轉譯器有很多（見 [Transpilers ](https://facebook.github.io/jsx/#transpilers)），較常見的是 [babel](https://babeljs.io/)，它可以把 JSX 語法轉成 javascript。

先等等，為什麼 babel 會知道
`<ShoppingList name="Mark" />` 轉成 `React.createElement(ShoppingList, { name: "Mark" })`，`React.createElement()` 是哪設定的，以後有 `MIT.createElement()` 可不可以？這是因為在轉譯前有設定 babel 的 `preset`，見下圖
![Screen Shot 2018-10-10 at 11.13.28 AM.png](resources/1D32D5AD248DE023A8F999624123C8DF.png =1387x596)
preset 選擇 `react` ，所以 babel 才知道要怎麼轉譯。再存細看一下，還有看到其它的 preset, es20XX 之類的，這表示 babel 可以認得的語法更多更新，都可以轉成更廣範使用/更低階/更原生的 javascript 語法，像是我們用的 ES6 語法中的 `Class`，可以用 babel 轉換成 更原生 javascript
![Screen Shot 2018-10-10 at 1.14.03 PM.png](resources/98A851947554B2F99984D0DED7D7FE76.png =1434x707)
這表示 **在當代瀏覽器就算沒支援太新的語法，還是可以透過轉譯器轉換成當代支援的 javascript**。早期 ES6 語法未被瀏覽器支援前就是如此，開發者使用較方便的語法開發，再轉換成可以支援javascript。

> 轉譯器也可以說是 **編譯器(compiler)**，因為它的行為就是從一個語言換成另一個語言，就像 C++ 的編譯器一樣把原始碼轉轉二進碼。

----

# 第一個前端 React 網頁

經過前一章的解釋有沒有發現，建立一個 React app 很不容易？要準備 component 原始碼、轉譯器、還要連結 React 和 DOM…等，一堆設定會使人怯步。

拜很多大神所賜，我們現在開發 React app 只要專注在 component，其它的有人會幫你設好。我們採用 [`create-react-app`](https://github.com/facebook/create-react-app)，這也是 Facebook 的專案，使得 React app 的建立更方便。

## 用 `create-react-app` 建立 `hello-react`
1. 安裝 `create-react-app` 指令
    ``` shell
    sudo npm install -g create-react-app
    ```
1. 開一個 `hello-react` 的資料夾，並移入
    ``` shell
    mkdir hello-react
    cd hello-react
    ```
1. 建立 `hello-react` 的 React app 專案
    ``` shell
    create-react-app .
    ```
    ![Screen Shot 2018-10-10 at 1.44.46 PM.png](resources/61717E0DB8D73499B2E8DB978D8CA4D3.png =1106x632)
    
    會建立以下的檔案
    ![Screen Shot 2018-10-10 at 1.46.36 PM.png](resources/4FDEE00233C5D67C458D4F0C2D61531B.png =216x411)
    
    這就是 Node.js 的專案，`src` 資料夾放的就是前端的 React 程式碼。
1. 執行開發用伺服器
    ``` shell
    npm start
    ```
    這時開啟開發伺服器，自動開啟一個網頁 `http://localhost:3000/`
    ![Screen Shot 2018-10-10 at 1.52.40 PM.png](resources/2DE7709B8566EB3247DF984B2657589A.png =663x576)
    會看到這結果，就是一個 React app。

## `create-react-app` 幫我們設定了什麼？
除了基本的 React app 專案的設定，還設定了一些開發時實用的東西

1. 開發用網頁伺服器
1. 程式碼異動監控
1. 語法檢查
1. 網頁除錯模式
1. 測試模式
1. 發佈靜態檔案
1. 脫離 `create-react-app` 提供的開發環境

### 開發用網頁伺服器
當執行 `npm start` 就可以執行開發伺服器。因為單單只有網頁原始碥是沒法在網頁看到結果，所以執行開發伺服器會進行之前所提的轉譯…等，並自動的開啟 React app 的網頁。

### 程式碼異動監控
不只如此，當程式碼發生異動時，還會自動重讀網頁

把 `.src/App.js` 改成如下：
``` javascript
import React, { Component } from 'react';
import './App.css';

class App extends Component {
  render() {
    const foo = 'foo';
    return (
      <div className="App">
        Hello React
      </div>
    );
  }
}

export default App;
```
存檔後回到網頁上看，網頁會自動重新整理

![Screen Shot 2018-10-10 at 2.02.42 PM.png](resources/F995E3680FCFAA1444DB5C027EB83DC5.png =200x80)

### 語法檢查
也設定了 ESLint，就如同我們在 [Day 6 - 一周目- 程式碼品質工具ESLint，照顧程式碼風格](https://ithelp.ithome.com.tw/articles/10200208) 中設定 Node.js 專案的程式碼品質監控。

![Screen Shot 2018-10-10 at 2.40.15 PM.png](resources/0BE41C153E8249BC0A8E682319BD9F38.png =665x330)

只是把 ESLint 組態設定放在 `package.json` 裡的 `eslintConfig`。這裡 ESLint 也可以找的到組態設定。
![Screen Shot 2018-10-10 at 2.44.35 PM.png](resources/507B523F5972DB9F197F58BACB54F03C.png =338x174)

### 網頁除錯模式
它也幫我們設定可以在 Chrome devTools 裡進行除錯。

打開 Chrome devTools 後，選 Source 頁籤，按下 command + p (Mac) / ctrl + p(windows)，輸入 `App` 過濾找到 App.js 源始碼。接下來你就可以像 VSCode debug 模式一樣下中斷點除錯。記得下完中斷點要重讀網頁才會觸發網頁 javascript 重新執行。

![Screen Recording 2018-10-10 at 2.49.10 PM.gif](resources/97BC936E4343967E8F2187A988BDE960.gif =274x370)

### 測試模式
另開啟一個 ternimal 執行 `npm run test` ，就可以開啟測試監控。它會掃描專案中的所有測式檔案並執行測試，當程式異動時它還會自動重新執行測試。未來我們會在二周目再做說明。

### 發佈靜態檔案
寫好的 React app 透過 `npm run build`，就會把原始碼轉譯、打包成少數js檔案，產生 `build` 資料夾，裡面就是純前端網頁的所有靜態檔案，把它們放在任何網頁伺服器(ex: nginx) 就可以透過瀏覽器讀取。

![Screen Shot 2018-10-10 at 3.09.15 PM.png](resources/0B8180604FE30F4EB899FB05508E05D3.png =275x181)

### 脫離 `create-react-app` 提供的開發環境
不喜歡它提供的開發環境嗎？ `npm run eject` 脫離他的開發環境。
會專案會變成
![Screen Shot 2018-10-10 at 3.25.56 PM.png](resources/D4835650E37F433A1EEBB51670314EE7.png =939x656)
這就是他的開發環境本來的面貌：用到 [Webpack](https://webpack.js.org/), [Babel](https://babeljs.io/), [ESLint](https://eslint.org/), [Jest](https://jestjs.io/)，和腳本…等。所有的工具都是要安裝、設定才能用在開發環境，對於新手來說超不友善，所以 `create-react-app` 幫我們省去了不少麻煩，專心開發 React app。

本主題不會脫離 `create-react-app` 的開發環境，所以不需要執行`npm run eject`。

> 這過程是不可還原的，所以用之前確認清楚。

> 我自己測試時發現，`create-react-app` 在建立 React app 時有開 local git repository，如果沒 commit 的異動檔案，`npm run eject` 不能運作。
![Screen Shot 2018-10-10 at 3.24.06 PM.png](resources/F9B14BCA2B7F4D3D1E256980C4A05FA5.png =718x276)


# 總結
今天我們介紹了 React 和使用在裡面的 JSX 語法，也用 `create-react-app` 建立了第一個 React app並說明他提供的開發環境。
