# 2020前端面试题汇总（2020.05-2020.10）

> 2020 年 5 月份选择了裸辞，这是一个不理智的决定，面试之后才发现自己有太多的不足，再加上疫情之后的经济环境，历经 5 个月终于才找到了一份工作。现在把这 5 月所有的面试题记录下来做一次总结，在此感谢在这 5 个月里关心我和帮我推荐工作的人们。

<!-- TOC -->

- [2020前端面试题汇总（2020.05-2020.10）](#2020前端面试题汇总202005-202010)
  - [web 性能优化](#web-性能优化)
  - [CommonJS 和 ES6 Module 的区别](#commonjs-和-es6-module-的区别)
  - [手写 call、apply、bind 方法](#手写-callapplybind-方法)
  - [实现一个宽高比 2 : 1 的 div](#实现一个宽高比-2--1-的-div)
  - [position: sticky 实现粘性布局](#position-sticky-实现粘性布局)
  - [给斐波那契函数实现缓存功能](#给斐波那契函数实现缓存功能)
  - [事件循环](#事件循环)
  - [什么是强缓存和协商缓存](#什么是强缓存和协商缓存)
  - [怎么处理 CSRF、XSS 漏洞](#怎么处理-csrfxss-漏洞)
  - [怎么处理跨域问题](#怎么处理跨域问题)
  - [排序算法](#排序算法)
  - [实现一个浮点数相加的方法，要保证精度（0.1 + 0.2 = 0.3）](#实现一个浮点数相加的方法要保证精度01--02--03)
  - [rem 和 em](#rem-和-em)
  - [css 选择器优先级](#css-选择器优先级)
  - [flex 1](#flex-1)
  - [实现上下左右居中](#实现上下左右居中)
  - [清除浮动的几种方式](#清除浮动的几种方式)
  - [移动端 1px 问题](#移动端-1px-问题)
  - [深拷贝](#深拷贝)
  - [css布局：左边固定右边自适应和左右固定中间自适应](#css布局左边固定右边自适应和左右固定中间自适应)
  - [代码题](#代码题)

<!-- /TOC -->

## web 性能优化

- 资源优化
  - CDN
    减少网络延迟，加快加载速度。
  - 预加载
    预先加载资源，提高用户体验，一般在 loading 页上处理；
    preloadjs 可做预加载，phaserjs 的 preload 方法可做预加载。
  - 懒加载
    按需加载，节省资源，提高用户体验；
    vue 按需加载路由 /*webpackChunkName: "xxx"*/
  - 资源压缩
    - 图片压缩
        利用 webpack url-loader 的 limit 参数将小图直接转为 base64;
        在线压缩地址：<https://tinypng.com/>
    - 字体压缩
        字蛛+：<https://github.com/allanguys/font-spider-plus>
    - 代码文件压缩
    - 服务器端 gzip 压缩
  - 缓存
    服务器端做相应的缓存技术，强缓存和协商缓存。
  - 资源分割
    图片或文件过大时，应该做分割处理；
    webpack optimization.SplitChunks 分割代码。
  - 减少请求
    使用雪碧图减少 http 请求；
    合并代码减少请求。
- 代码优化
  - 减少 dom 层级，减少 css 选择器层级；层级越深，渲染性能越低。
  - css 有多个层级时，尽量不要直接使用标签名，因为 css 的遍历是从内到外（从右到左）的。例如：

    ```css
    .nav a {
        color: red;
    }
    /*
        css 会先遍历所有的 a 标签，再找到 .nav 下的 a 标签。
    */

  - 减少 dom 操作的次数。
  - 注意内存回收，尤其是再 dom 节点或 dom 节点绑定的方法赋值个某个变量时。
  - 减少绑定方法的次数。
  - 尽量使用 transform 或者 opacity 来实现动画效果。相关阅读：<https://fed.taobao.org/blog/taofed/do71ct/performance-composite/>
  - for 循环体先将循环次数放到变量中，避免多次取值影响性能

    ```javascript
      // 错误
      for (let i = 0; i < arr.length; i++) {}
      // 正确
      for (let i = 0, len = arr.length; i < len; i++) {}

  - 避免使用全局变量
    因为原型链的存在，使用全局变量要比局部变量开销更大。
  - if 语句分支过多时，应当使用 switch；switch 可以直接定位到指定条件。

## CommonJS 和 ES6 Module 的区别

- CommonJS 模块依赖关系的建立发生在代码**运行**阶段；ES6 Module 模块依赖关系的建立发生在代码**编译**阶段。
- CommonJS require 的模块路径可以动态指定，支持传入一个表达式，也可以使用 if 语句判断是否加载某个模块；ES6 Module 的导入、导出语句都是声明式的，不支持导入的路径是一个表达式，并且导入、导出语句必须位于模块的顶层作用域（比如不能放在 if 语句中）。
- 在导入一个模块时，CommonJS 获取的是一份导出值的拷贝；ES6 Module 获取的是值的动态映射，并且这个映射是只读的。

## 手写 call、apply、bind 方法

```javascript
// call
Function.prototype.myCall = function () {
  context = context || window;
  let fn = Symbol();
  context[fn] = this;
  let args = [...arguments].slice(1);
  let ret = context[fn](...args);
  delete context[fn];
  return ret;
}

// apply
Function.prototype.myApply = function () {
  context = context || window;
  let fn = Symbol();
  context[fn] = this;
  let args = [...arguments][1] || [];
  let ret = context[fn](...args);
  delete context[fn];
  return ret;
}

// bind
Function.prototype.myBind = function (context) {
  context = context || window;
  let fn = this;
  let args = [...arguments].slice(1);
  return function () {
    let exeArgs = [...arguments]
    fn.apply(context, [...args, ...exeArgs])
  }
}
```

## 实现一个宽高比 2 : 1 的 div

```css
.div {
  box-size: border-box;
  width: 100%;
  padding-bottom: 50%
}
```

## position: sticky 实现粘性布局

```css
.nav {
  position: -webkit-sticky;
  position: sticky;
  width: 100%;
  height: 100px;
  background-color: #f00;
  top: 0;
  left: 0;
}
```

## 给斐波那契函数实现缓存功能

```javascript
let fibCache = (function() {
    let cache = {};
    return function fib(n) {
        if (n < 2) {
            return n;
        }

        if (cache[n]) {
            return cache[n];
        } else {
            return cache[n] = fib(n - 1) + fib(n - 2);
        }
    }
})();
fibCache(10);
```

## 事件循环

  JavaScript 是单线程的，所有的任务都需要排队，任务又分为同步任务和异步任务。
  所有的同步任务都在主线程中排队执行，只有前一个任务执行完毕才会去执行下一个任务。异步任务则放在任务队列中排队，分为宏任务队列和微任务队列。
  **只有主线程执行栈为空时，才会读取微任务队列中的任务到执行栈中执行，只有当微任务队列为空时，才会读取宏任务队列中的任务到执行栈中执行。**

## 什么是强缓存和协商缓存

  浏览器请求一次资源后，可通过服务器设置响应头来控制资源的缓存方式。
  浏览器命中强缓存时，直接从本地读取资源，并返回 200 状态码，不会向服务器发送请求。
  浏览器向服务器发送请求，命中协商缓存时，直接从本地读取资源，并返回 304 状态码。

- 强缓存: 通过响应头 `expires` 或 `cache-control` 设置
  - expires(http1.0): 值为 GMT 格式的时间，在此时间内则命中强缓存。
  - cache-control(http1.1)：取值 max-age=xxx，xxx 数值，单位秒，在该有效期内则命中强缓存。
  - 当 `expires` 和 `cache-control` 同时存在时，`cache-control` 优先级高于 `expires`。
- 协商缓存：通过字段 `last-modified/if-modified-since` 和 `etag/if-none-match` 设置
  - last-modified/if-modified-since(http1.0)：值为 GMT 格式时间，表示资源最后修改时间，当浏览器再次发送请求时，设置请求头 `if-modified-since` 的值为上次请求中 `last-modified` 的值，服务器比对 `last-modified` 和 `if-modified-since`，如果相同则命中协商缓存。
  - etag/if-none-match(http1.1)：值为一段字符串，表示文件的唯一标识，当浏览器再次发送请求时，设置请求头 `if-none-match` 的值为上次请求中 `etag` 的值，服务器比对 `etag` 和 `if-none-match`，如果相同则命中协商缓存。

## 怎么处理 CSRF、XSS 漏洞

- CSRF（Cross-site request forgery）跨站请求伪造。相关阅读：<https://tech.meituan.com/2018/10/11/fe-security-csrf.html>
  - 攻击
    - 用户登录 A 网站，并保留了登录凭证（cookie）
    - 用户访问了 B 网站
    - B 网站向 A 网站发送一个请求，浏览器会默认携带 cookie
    - A 网站对请求进行验证，确认是用户的凭证
    - A 网站执行 B 网站的请求
  - 防御
    - 同源检测：通过 http 请求头中的`origin`或`referer`字段，确定请求的来源域名
    - 添加 token
    - 双重 cookie 验证
- XSS（Cross-Site Scripting）跨站脚本攻击。相关阅读：<https://tech.meituan.com/2018/09/27/fe-security.html>
  - 攻击：攻击者将恶意代码插入到页面中
  - 防御：过滤敏感字段
  
## 怎么处理跨域问题

- 1.服务器设置响应头：`access-control-allow-origin`，放开跨域问题；
- 2.利用 JSONP 处理，因为 JSONP 是通过创建 `script` 标签的方式发送请求，所以只能用于 `get` 方法；
- 3.通过服务器的反向代理方式。

## 排序算法

- 冒泡排序

  ```JavaScript
    // 冒泡排序
    function bubbleSort(arr) {
        let len = arr.length;
        for (let i = 0; i < len; i++) {
            for (let j = 0; j < len - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                }
            }
        }
        return arr;
    }
  ```

- 选择排序

  ```javascript
    function selectionSort(arr) {
      let len = arr.length;
      let indexMin;
      for (let i = 0; i < len - 1; i++) {
        indexMin = i;
        for (let j = i + 1; j < len; j++) {
          if (arr[j] < arr[indexMin]) {
              indexMin = j
          }
        }

        if (i !== indexMin) {
          [arr[i], arr[indexMin]] = [arr[indexMin], arr[i]]
        }
      }
      return arr;
    }
  ```

- 插入排序

  ```javascript
    function insertSort(arr) {
      let len = arr.length;
      let temp;
      for (let i = 1; i < len; i++) {
        temp = arr[i];
        let j = i - 1;
        while (j >= 0 && temp < arr[j]) {
          arr[j + 1] = arr[j];
          j--;
        }
        arr[j + 1] = temp;
      }
      return arr;
    }
  ```

- 快速排序
  
  ```javascript
    function quickSort(arr) {
      function quick(arr, left, right) {
        if (arr.length < 2) return arr;
        let piovt = arr[Math.floor((left + right) / 2)];
        let indexLeft = left;
        let indexRight = right;
        while (indexLeft <= indexRight) {
          while (arr[indexLeft] < piovt) {
            indexLeft++;
          }
          while (arr[indexRight] > piovt) {
            indexRight--;
          }

          if (indexLeft <= indexRight) {
            [arr[indexLeft], arr[indexRight]] = [arr[indexRight], arr[indexLeft]];
            indexLeft++;
            indexRight--;
          }
        }
        if (left < indexLeft - 1) {
          quick(arr, left, indexLeft - 1);
        }
        if (right > indexRight + 1) {
          quick(arr, indexRight + 1, right);
        }
      }
      quick(arr, 0, arr.length - 1);
      return arr;
    }
  ```

- 归并排序

  ```javascript
    function mergeSort(arr) {
      let len = arr.length;
      if (len > 1) {
        let mid = Math.floor(len / 2);
        let left = mergeSort(arr.slice(0, mid));
        let right = mergeSort(arr.slice(mid, len));
        arr = merge(left, right);
      }

      function merge(left, right) {
        let indexLeft = 0;
        let indexRight = 0;
        let res = [];
        while (indexLeft < left.length && indexRight < right.length) {
          if (left[indexLeft] < right[indexRight]) {
            res.push(left[indexLeft++]);
          } else {
            res.push(right[indexRight++]);
          }
        }
        if (indexLeft < left.length) {
          return [...res, ...left.slice(indexLeft)];
        } else {
          return [...res, ...right.slice(indexRight)];
        }
      }
      return arr;
    }
  ```

## 实现一个浮点数相加的方法，要保证精度（0.1 + 0.2 = 0.3）

```javascript
  function sum(num1, num2) {
    let str1 = num1.toString();
    let str2 = num2.toString();
    let point1 = str1.indexOf('.');
    let point2 = str2.indexOf('.');
    let intNum1 = parseInt(str1.split('.').join(''), 10);
    let intNum2 = parseInt(str2.split('.').join(''), 10);
    let offset = Math.abs(point1 - point2);
    let ret;

    if (point1 > point2) {
        intNum1 = intNum1 * Math.pow(10, offset);
        ret = (intNum1 + intNum2) / Math.pow(10, point1);
    } else if (point1 < point2) {
        intNum2 = intNum2 * Math.pow(10, offset);
        ret = (intNum1 + intNum2) / Math.pow(10, point2);
    } else {
        ret = (intNum1 + intNum2) / Math.pow(10, point1);
    }
    return ret
  }
```

## rem 和 em

rem 是根据根节点的 `font-size` 变化，em 是根据父级节点的 `font-size` 变化。

## css 选择器优先级

!important > 内联 style > ID 选择器 > 类选择器 > 标签选择器 > 通配符选择器

## flex 1

`flex`是`flex-grow`、`flex-shrink`和`flex-basis`的缩写，`flex: 1`即是`flex: 1 1 auto`的缩写。
相关阅读：
<http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html>
<https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex>

## 实现上下左右居中

```css
/* 方式一 */
.box {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* 方式二 */
.box {
  position: absolute;
  width: 100px;
  height: 100px;
  top: 50%;
  left: 50%;
  margin-top: -50px;
  margin-left: -50px;
}
/* 方式三 */
.box {
  display: flex;
  justify-content: center;
  align-item: center;
}
```

## 清除浮动的几种方式

- `clear: both`
- 伪类：`&:after {content: "";clear: both;display: block;}`
- 给父级容器设置高度
- BFC 方式，详见：<https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context>

## 移动端 1px 问题

- 使用图片
- box-shadow
- `transform: scale(0.5)`
- 设置`viewport`的`scale`值

## 深拷贝

- `JSON.parse(JSON.stringify())`，不够全面。

- ```javascript
  function clone(target, memoMap = new WeakMap()) {
    if (typeof target === 'object') {
      let cloneTarget = Array.isArray(target) ? [] : {};
      if (memoMap.get(target)) {
        return map.get(target);
      }
      memoMap.set(target, cloneTarget);
      for (const key in target) {
        cloneTarget[key] = clone(target[key], map);
      }
      return cloneTarget;
    } else {
      return target;
    }
  }
  ```

相关阅读：<https://juejin.im/post/6844903929705136141#heading-14>

## 浏览器的渲染过程

- 根据`html`构建 DOM
- 根据`css`构建 CSSOM
- 将 DOM 和 CSSOM 合并成 Render Tree
- 根据 Render Tree 进行布局
- 最后绘制

## 判断数组的方法

```javascript
let arr = [];
// Array.isArray()
if (Array.isArray(arr)) console.log('is array');
// instanceof
if (arr instanceof Array) console.log('is array');
// toString()
if (Object.prototype.toString.apply(arr).toLowerCase() === '[object array]') console.log('is array');
```

## css布局：左边固定右边自适应和左右固定中间自适应

- 左边固定右边自适应
  - `position`

    ```css
      .box {
        width: 200px;
        position: relative;
      }
      .left {
        position: absolute;
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
      }
      .right {
        margin-left: 50px;
        background-color: bisque;
      }
    ```

  - `float`

    ```css
      .box {
        width: 200px;
      }
      .left {
        float: left;
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
      }
      .right {
        background-color: bisque;
      }
      ```

  - `flex`

    ```css
      .box {
        width: 200px;
        display: flex;
      }
      .left {
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
      }
      .right {
        flex: 1;
        margin-left: 50px;
        background-color: bisque;
      }
    ```

- 左右固定中间自适应
  - `flex`

    ```css
      .box {
        width: 200px;
        display: flex;
      }
      .left {
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
      }
      .mid {
        flex: 1;
        background-color: chocolate;
      }
      .right {
        width: 80px;
        background-color: bisque;
      }
    ```

  - 浮动

    ```html
      <div class="box">
        <div class='left'>左边左边左边左边左边左边左边</div>
        <div class='right'>右边右边右边右边右边右边右边</div>
        <div class="mid">中间中间中间中间中间中间中间中间</div>
      </div>
    ```

    ```css
      .box {
        width: 200px;
      }
      .left {
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
        float: left;
      }
      .mid {
        background-color: chocolate;
        overflow: hidden;
      }
      .right {
        float: right;
        width: 50px;
        background-color: bisque;
      }
    ```

  - `table`布局

    ```css
      .box {
        width: 200px;
        display: table;
      }
      .left {
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
        display: table-cell;
      }
      .mid {
        background-color: chocolate;
        display: table-cell;
      }
      .right {
        width: 50px;
        background-color: bisque;
        display: table-cell;
      }
    ```

  - 绝对定位

    ```css
      .box {
        width: 200px;
        position: relative;
      }
      .left {
        width: 50px;
        height: 100px;
        background-color: aqua;
        opacity: 0.5;
        position: absolute;
        left: 0;
      }
      .mid {
        background-color: chocolate;
        position: absolute;
        left: 50px;
        right: 50px;
      }
      .right {
        width: 50px;
        background-color: bisque;
        position: absolute;
        right: 0;
      }
    ```
  
## 代码题

```javascript
// 请写出输出结果
let ret = [1,2,10].map(parseInt);
console.log(ret); // ret: [1, NaN, 2]

// 请写出输出结果
let s = new String('s');
switch (s) {
  case '':
    console.log(1);
    break;
  case 's':
    console.log(2);
    break;
  default:
    console.log(3);
    break;
}
// 输出结果： 3
```
