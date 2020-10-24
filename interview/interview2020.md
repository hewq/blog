# 2020前端面试题汇总（2020.05-2020.10）
> 2020 年 5 月份选择了裸辞，这是一个不理智的决定，面试之后才发现自己有太多的不足，再加上疫情之后的经济环境，历经 5 个月终于才找到了一份工作。现在把这 5 月所有的面试题记录下来做一次总结，在此感谢在这 5 个月里关心我和帮我推荐工作的人们。

## web 性能优化？
- 资源优化
  - CDN
    减少网络延迟，加快加载速度。
  - 预加载
    预先加载资源，提高用户体验，一般在 loading 页上处理；
    preloadjs 可做预加载，phaserjs 的 preload 方法可做预加载。
  - 懒加载
    按需加载，节省资源，提高用户体验；
    vue 按需加载路由 /* webpackChunkName: "xxx" */
  - 资源压缩
    - 图片压缩
        利用 webpack url-loader 的 limit 参数将小图直接转为 base64;
        在线压缩地址：https://tinypng.com/
    - 字体压缩
        字蛛+：https://github.com/allanguys/font-spider-plus
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
        css 会先遍历所有的`a`标签，再找到`.nav`下的`a`标签。
    */
  - 减少 dom 操作的次数。
  - 注意内存回收，尤其是再 dom 节点或 dom 节点绑定的方法赋值个某个变量时。
  - 减少绑定方法的次数。
  - 尽量使用 transform 或者 opacity 来实现动画效果。（相关阅读：https://fed.taobao.org/blog/taofed/do71ct/performance-composite/）
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
## 什么是强缓存和协商缓存？
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
## 怎么处理 CSRF、XSS 漏洞？



## 怎么处理跨域问题？
- 1.服务器设置响应头：`access-control-allow-origin`，放开跨域问题；
- 2.利用 JSONP 处理，因为 JSONP 是通过创建 `script` 标签的方式发送请求，所以只能用于 `get` 方法；
- 3.通过服务器的反向代理方式。
## localStorage、sessionStorage 和 cookie 的区别？
## 排序算法
## 实现一个浮点数相加的方法，要保证精度？（0.1 + 0.2 = 0.3）
## rem 和 em
rem 是根据根节点的 `font-size` 变化，em 是根据父级节点的 `font-size` 变化。
## css 选择器优先级

## flex 1
## 实现上下左右居中
## get 和 post 的区别
## 清除浮动的几种方式
## 1px 问题
## 深拷贝
## 浏览器的渲染过程
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
