+++
title = "Day 4 - 一周目- 用VSCode debug 模式，玩玩 ES6 常用語法"
date = "2018-10-04"
description = "debug模式更細部說明，以及基本ES6：變數、內建物件、邏輯、迴圈、條件"
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

昨天介紹了用VSCode執行debug模式，今天要來看debug模式可以做什麼？

<!--more-->

# 回憶
昨天介紹了用VSCode執行debug模式，今天要來看debug模式可以做什麼？

# 目標

* debug模式更細部說明
* 程式在debug模式下觀察程式
* 基本ES6：變數、內建物件、邏輯、迴圈、條件

# VSCode 偵錯(debug) 模式
大部的程式執行環境都可以透過編輯器或[整合開發環境 IDE](https://zh.wikipedia.org/zh-tw/%E9%9B%86%E6%88%90%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83) 支援debug模式。 Node.js 支援 debug模式，我們可以使用 VSCode 方便介面的操作，對執行的程式debug。

# debug 模式
debug 模式就是可查看程式的執行歷程，進而進行干涉。

## debug 模式可以做什麼？

* 觀察程式執行路徑，下中斷點(breakpoint)、操作Step over/Step in/Step out、看執行函式堆疊
* 觀察/修改變數的值

## 準備偵錯模式的遊樂場(Playground) 
1. 建立**hello-es6** 專案，因為資料夾就是一個專案，所以可以直接把昨天的 **hello-nodejs** 資料夾拷貝一份再改名為 **hello-es6**，就完成了。順手改  ***package.json*** 內的專案名稱 ***name***
2. 把 `index.js`內容改成
    ``` javascript
    function fool(a) {
        console.log('a is:' + a + ' in fool()');
    }
    
    const a = 1;
    console.log('a is:' + a);
    fool(a);
    ```
3. 執行debug模式：選 **Debug** 裡的 **Start Debugging**　或選好執行 index.js 的執行組態再按綠色箭頭
![插圖2.png](resources/660975DFE5A4DE5F812B27A91262C649.png =601x88)
4. 看到執行結果
![Screen Shot 2018-10-04 at 8.57.03 AM.png](resources/417B925800BA0A3D61E283ECC8CA3C01.png =661x112)

## 玩轉 deubg
1. 在第2, 7行下中斷點
![Screen Recording 2018-10-04 at 8.58.32 AM.gif](resources/B6F33809CF89A43375BFAA4913CEA04E.gif =400x183)
1. 執行debug模式，看看發生什麼事？
![Screen Shot 2018-10-04 at 9.06.39 AM.png](resources/9B8437405A58FBD30F114D15EC77E9B2.png =573x205)
沒錯!程式停下來了、停在第2行~莫慌，我們接下去
1. 按下 Contiune (往右的箭頭)
![Screen Recording 2018-10-04 at 9.12.36 AM.gif](resources/487EC3C5C47736BBB73EC99CDFDF3BBA.gif =400x248)
按第一下 Contiune，就停在第7行，再按一次才會執行完並印出完整的結果

## debug 模式常用功能區塊
第一小節，我們看到 **中斷點** 可以使程式停在這位置，而 **Contiune**(往右的箭頭) 繼續執行直到下一個中斷點或執行結束。

接下來，我們來看看常用功能區塊
![Screen Shot 2018-10-04 at 9.41.12 AM.png](resources/4B5A013B3C0E9EA551A78FA79F89C1FB.png =866x772)
1. VARIABLES：變數值
    執行時可以存取到的變數和他們的值。你甚至可以連點兩下修改他們，進而影響後續執行。
2. WATCH：自定義程式表達式(expression)
    圖中：`a+1` 會被執行出結果，常用於查看特定的值，像是物件的屬性值object1.property1。
3. CALL STACK：函數的執行堆疊(stack)
    圖中程式執行到第二行，所以會看到最上面是`fool() 2:5`，表示現在正在 `fool()` 函數中，第 `二` 行第 `五` 個字。你可以點擊第二個 `(anonymous function)` 會看到是從哪進來的，且會 **VARIABLES** 是進來前的狀態。
4. BREAKPOINTS：中斷點的位置
    勾勾可以開關它們並保留下它們。
5. DEBUG CONSOLE：**condoel.log()** 印出的結果。
    condoel.log()印到標準輸出[**stdout**](https://nodejs.org/dist/latest-v10.x/docs/api/console.html#console_console_log_data_args)
6. 程式流程控制儀表板(panel)：依次是 Contiune, Step Over, Step In, Step Out, Restart, Stop
    1. Contiune：繼續執行
    2. Step Over：不離開當前執行函數，單步執行到下一行
    3. Step In：單步執行到下一行。若有函數就進入
    4. Step Out：離開當前函數後，停留在下一行
    5. Restart：整個 deubg 重新執行
    6. Stop：終止整個 deubg 模式
7. EVALUATE EXPRESSION: 執行程式碼片斷
    這名字我是從 WebStorm 來的，你可以在這直接寫程式執行。把下面這行貼入按Enter執行
    ``` javascript
    function sayHi(a) {
      console.log('Hi! a is ' + a + ' in syhHi()');
    }
    sayHi(a);
    ```
    
> 你可以多玩玩程式流程控制儀表板和 `VARIABLES`和 `WATCH` 區塊，這對於除錯有莫大的幫助

> 我發現動態的改值或執行程式碼片斷，可能會導致 `VARIABLES` 和 `WATCH` 的值殘留舊值。不過，在我的除錯經驗中，真的很少改值。

-----

# ES6 (ECMAScript6) 快速學習
上一章，我們已學會用 VSCode 在 debug 模式下運行，這不僅是用來除錯，還可以拿來習學用的。

ES6 是 [ECMAScript6](https://zh.wikipedia.org/wiki/ECMAScript)的簡稱，它只是個 **規格(specification)** ，而 javascript 就是實現他的語言。

然而，ES6 的特性不一定 javascript會實現，更重要是能不能在 Node.js 或瀏覽器使用
* Node.js 可以見官方文件[ECMAScript 2015 (ES6) and beyond](https://nodejs.org/en/docs/es6/#ecmascript-2015-es6-and-beyond)
* 瀏覽器就有點麻煩了，有瀏覽器要怎麼查？可以用[Can I use?](https://caniuse.com/)查網頁技術的支援程度

我覺得不管在什麼環境，你跑跑看就知道了。好消息是 ES6 常用的語法，大部分的環境都支援了。

## 變數(Variable)：用來存取值的名稱
變數是程式存取值的名稱，例如：
``` javascript
const a = 1;
```
這行看來簡單，其實有很多小細節在裡面：
1. 宣告：程式在執行前要先配置記體憶空間 (實際上，在`{. . . }`內所有宣告的變數都會在此區塊被執行前一起配置)
2. 變數範圍/範籌/作用域(Variable Scope)：這個變數被修飾為 `const` ，就是一但賦值後「變數的值」再也不能被修改。Scope 還有：`let`、`var`、`const`。
3. 賦值： `=` 是賦值符號

## 基本型別/原始型別 (Primitives)：有哪些類型的值？
1. Boolean - true / false
    ``` javascript
    const a = true;
    ```
1. Null - "沒有值"的值
    ``` javascript
    const a = null;
    ```
1. Undefined - 被宣告過的變數，但沒有設定值
    ``` javascript
    const a = undefined;
    ```
1. Number - 數字，像：整數、浮點數
    ``` javascript
    const a = 1;
    const a = 1.1;
    ```
1. String - 字元陣例
    ``` javascript
    const a = 'hello';
    ```
1. Symbol - 全局執行過程中唯一的值，不可能有一樣的
    ``` javascript
    const a = Symbol('foo');
    const b = Symbol('foo');
    a === b // false
    ```
    這是 ES6 才引入，用途見 [ES6 In Depth: Symbols](https://hacks.mozilla.org/2015/06/es6-in-depth-symbols/)，但我們不會用到
1. 除了上述之外，其它值的都是 `Object` 型別。

## 常用的內建物件型別：Object, Array, String, Date, Error
這些怎麼用去 google 都可以查的到，所以我只列出常用的操作，想知道有哪函操作可以查看 [Standard built-in objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)。

假如我打開 [Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)，會看到如下：
![Screen Shot 2018-10-04 at 9.54.56 PM.png](resources/E3F07A9C2CBCFDFA500856DC4070FD30.png =807x492)
文件中會先給簡述，再來是Demo，也可以立刻執行玩看看，之後才是語法或更詳細的說明。先執行程式看看效果，對於學習是很有幫助的，有興趣的人再去深入查看。

接下來的內容看起來很無聊，也沒有必要學會每個函數、更不用背它們，當要用的時候在上 google 查就好了。儘量保持問問題的想法，像是：
* Q: 怎麼串連子串陣列？A:  `['a', 'b', 'c'].join(',')`
* Q: Java 的字串有 split 方法，那 javascript 有沒有？A: `'a,b,c'.split(',')`

這樣學起來有趣多了。

### Array
``` javascript
// 宣告
const words = ['a', 'b', 'c'];

// 設定值
words[0] = 'd';

// 取值
console.log(words[0]);

// 轉換
words.map(word => words + '->');　// a->b->c->
```

### Object

``` javascript
// 宣告
const person = {
  name: 'May'
};

// 設定值
person.name = 'Billy';

// 取值
console.log(person.name);

// 動態塞key-value
person.fax = '0000000000'
console.log(person.fax);

// 取keys
Object.keys(person);

// 取value
Object.values(person);
```

### String
``` javascript
// 宣告
const msg = 'hi!';

// 串接 string
const msg = msg + ' May.';

// 樣版字串(Template literals)
const msg2 = `${msg} Bye!`

// 宣告(含特殊字元)
const msg3 = "Hi!\nMay"
```

### Date
``` javascript
// 現在時間
const now = new Date();
```
雖然有內建的 Date ，但實務上用 [moment](https://momentjs.com/) 套件比好用。

### Error
```
// 宣告
const e = new Error('wrong password');

// 拋出例外(exception)
throw e;
```

還有其它像：Promise，我想獨立出來說。

## 邏輯運算
邏輯運算有 `==`, `!=`, `===`, `!==`, `&&`, `||`

這裡我們需要 `==` 和 `===` 是不同的，直接看例子
```
console.log(1 == '1'); // true
console.log(1 === '1'); // false
```

javascript 是會 type coercion(會自動的隱式轉型)，所以 `===` 就是不做隱式轉型，型別也要一樣。建議使用 `===`, `!==` 有較強的比較結果，避免隱式轉型造成的不可預測的情況。

## 迴圈(loop)
``` javascript
// 陣列
const a = ['a', 'b', 'c'];
for (let i = 0; i < a.length; i++) {
    console.log(a[i]);
}
a.forEach(element => {
    console.log(element);
});
for (const element of a) {
    console.log(element);
}
for (const key in a) {
    console.log(key, a[key]);
}

// 物件
const object = {first: 1, second: 2};
for (const key in object) {
    console.log(key, object[key]);
}
for (const key of Object.keys(object)) {
    console.log(key, object[key]);
}

// 另外還有 while loop，可以自己查看看
```
是不是被 `for in`、`for of`、`forEach` 弄的很亂阿？可以使用 [lodash forEach](https://lodash.com/docs/4.17.10#forEach) 統一寫法
``` javascript
const _ = require('lodash');
_.forEach([1, 2], (value) => {
  console.log(value);
});
 
_.forEach({ 'a': 1, 'b': 2 }, (value, key) => {
  console.log(key);
});
```

## 條件符(condition)
有 `if`, `switch`, `()?X:Y`(三元運算ternary operation)

``` javascript
const a = '1';
if(a === '1') {
  console.log('euqal');
}

switch(a){
    case '1':
    case '2': {
      console.log('a');
      break;
    }
    case '3': {
      console.log('b');
      break;
    }
    default: {
      console.log('default');
      break;
    }
}

console.log(a === '1' ? 'equal': 'not equal');
```

利用 `&&` 和 `||` 可做到類似條件運算，這技巧在渲染react component 很好用，可以少寫判斷程式碼
``` javascript
// || 直到有值
const a1 = undefined;
const b1 = null;
const c1 = a1 || b1 || 'default c1';
console.log(c1); // default c1

const a2 = undefined;
const b2 = 'b2';
const c2 = a2 || b2|| 'default c2';
console.log(c2); // b2

// && 前面是 true，才會設後面的值，否則是false
const a = true;
const b = a && 'set b';
const c = !a && 'set c';
console.log(b, c); // set b, false
```

## 關於 javascript 的觀察
1. 變數宣告不用指明 **變數型態**， 不用像 C++ `int a = 1;`，反而 Scope 的修飾更為重要
2. 特別小心 [type coercion](https://medium.freecodecamp.org/js-type-coercion-explained-27ba3d9a2839), 否則會出現有趣的情況(見[What the f*ck JavaScript?](https://github.com/denysdovhan/wtfjs))，`typeof`只有少許[可能的回傳值](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Operators/typeof#Description)，這函數不是真的回傳型態
    ``` javascript
    const a = [];
    console.log(typeof a); // object
    ```
3. lodash 也提供方便的查型態函數，但還是有可能有無法預期的結果，使用前一定要測測看
![Screen Shot 2018-10-05 at 9.07.11 AM.png](resources/8FE09BFF11BB6967033F7D00A3901414.png =276x699)

# 回顧
介紹了VSCode debug 模式和操作，幫助我們學習 javascript 語言。同時，了解到javascript有很多因為 type coercion 產生的好笑的情況。

# 參考連結
* [JavaScript Essentials: Types & Data Structures](https://codeburst.io/javascript-essentials-types-data-structures-3ac039f9877b)


