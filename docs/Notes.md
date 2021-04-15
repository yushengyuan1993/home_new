- [01. new操作符做了什么](#01-new操作符做了什么)
- [02. script标签的defer属性和async属性](#02-script标签的defer属性和async属性)
- [03. 手写防抖debounce和节流throttle](#03-手写防抖debounce和节流throttle)
- [04. 手写Promise](#04-手写promise)
- [05. 手写数字格式化，比如输入9999999，输出9,999,999](#05-手写数字格式化比如输入9999999输出9999999)
- [06. 手写一个sleep函数](#06-手写一个sleep函数)
- [07. 手写call、apply和bind](#07-手写callapply和bind)
- [08. 手写一个双向数据绑定](#08-手写一个双向数据绑定)
- [09. 手写浅拷贝和深拷贝](#09-手写浅拷贝和深拷贝)
- [10. 手写函数柯里化](#10-手写函数柯里化)

## 01. new操作符做了什么
  > `new` 共经历了四个过程。
  ```js
  var Fn = function () { };
  var fnObj = new Fn();
  ```
  1. 创建一个空对象
   ```js
   var obj = new Object();
   ```
  2. 设置原型链
   ```js
   obj.__proto__ = Fn.prototype;
   ```
  3. 让Fn的this指向obj，并执行Fn的函数体
   ```js
   var result = Fn.call(obj);
   ```
  4. 判断Fn的返回值类型，如果是值类型，就返回obj。如果是引用类型，就返回这个引用类型的对象
   ```js
   if (typeof result === 'object') {
     fnObj = result;
   } else {
     fnObj = obj;
   }
   // return typeof result === 'object' ? result : obj;
   ```

## 02. script标签的defer属性和async属性
  1. 当HTML页面解析到以下标签时：
  - `<script>`，脚本没有 defer 或 async，浏览器会立即加载并执行指定的脚本，也就是说不等待后续载入的文档元素，读到就加载并执行。执行结束后，HTML解析继续。
  - `<script defer>`，defer 属性表示延迟执行引入的 JavaScript，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。当整个 document 解析完毕后再执行脚本文件，在 DOMContentLoaded 事件触发之前完成。多个脚本按顺序执行。
  - `<script async>`，async 属性表示异步执行引入的 JavaScript，与 defer 的区别在于，如果已经加载好，就会开始执行，也就是说它的执行仍然会阻塞文档的解析，只是它的加载过程不会阻塞。多个脚本的执行顺序无法保证。

  注意，在没有src属性的情况下，async和defer属性会被忽略。

  2. 延伸问题，为何把CSS`<link>`标签放在`<head></head>`之间，把JS`<script>`标签放在`</body>`之前？

  - **把`<link>`标签放在`<head></head>`之间**

    把`<link>`标签放在`<head></head>`之间是规范要求如此。另外，这种写法可以让页面有序的逐步呈现，提高了用户体验。如果将样式文件放在文档底部，会造成HTML的样式渲染延后，用户可能会看到空白页或者一堆没有样式的页面。
  
  - **把`<script>`标签恰好放在`</body>`之前** 

    脚本在下载和执行期间会阻止HTML的解析。把`<script>`标签放在底部，能保证HTML首先完成解析，将页面内容尽早呈现给用户。

    但是，当脚本里包含`document.write()`时，不应放在底部，但是现在规范要求不使用`document.write()`。同时，将`<script>`标签放在底部时，在整个文档（`document`）被解析之前，浏览器不会开始下载脚本。也许，一个比较好的方法是，在`<script>`添加`defer`属性，然后将其放在`<head>`标签中。

## 03. 手写防抖debounce和节流throttle
  1. 防抖debounce
  - 原理：在事件被触发的n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
  - 适用场景：
    1. 按钮提交：防止多次提交，至执行最后一次提交动作；
    2. 搜索联想：防止联想发送请求，只发送最后一次输入的请求。
  - 简易版实现：
    ```js
    function debounce(func, wait, immediate) {
      let timeout = null;

      return function() {
        const that = this;
        const args = arguments;
        
        if (timeout) clearTimeout(timeout);

        if (immediate) {
          const callNow = !timeout;
          timeout = setTimeout(() => {
            timeout = null
          }, wait);

          if (callNow) func.apply(that, args);
        } else {
          timeout = setTimeout(() => {
            func.apply(that, args);
          }, wait);
        }
      }
    }
    ```
  2. 节流throttle
  - 原理：规定在一个单位时间内，只能触发一次回调函数，如果这个单位时间内触发了多次回调函数，则只有一次生效。
  - 适用场景：
    1. 拖拽场景：固定时间内值触发一次，避免短时间内高频触发
    2. 缩放场景：浏览器窗口的 resize
  - 简易版实现：
    ```js
    // 1. 使用时间戳
    function throttle(func, wait) {
      let that, args;
      let previous = 0;

      return function() {
        let now = + new Date();
        that = this;
        args = arguments;

        if (now - previous > wait) {
          func.apply(that, args);
          previous = now;
        }
      }
    }

    // 2. 使用定时器
    function throttle(func, wait) {
      let timeout = null;

      return function() {
        const that = this;
        const args = arguments;

        if (!timeout) {
          timeout = setTimeout(() => {
            timeout = null;
            func.apply(that, args);
          }, wait);
        }
      }
    }
    ```

## 04. 手写Promise
  ```js
  function myPromise(constructor) {
    const that = this;
    that.status = "pending"; // 定义状态改变前的初始状态
    that.value = undefined;  // 定义状态为resolved的时候的状态
    that.reason = undefined; // 定义状态为rejected的时候的状态
    function resolve(value) {
      // 两个==="pending"，保证了状态的改变是不可逆的
      if (that.status === "pending") {
        that.value = value;
        that.status = "resolved";
      }
    }
    function reject(reason) {
      // 两个==="pending"，保证了状态的改变是不可逆的
      if (that.status === "pending") {
        that.reason = reason;
        that.status = "rejected";
      }
    }
    // 捕获构造异常
    try {
      constructor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }
  myPromise.prototype.then = function(onFullfilled, onRejected) {
    const that = this;
    switch (that.status) {
      case "resolved": onFullfilled(that.value); break;
      case "rejected": onRejected(that.reason); break;
      default: 
    }
  }

  // 测试
  const p = new myPromise(function(resolve, reject) {
    if (Math.random() > 0.5) {
      resolve('success');
    } else {
      reject('fail');
    }
  });
  p.then(function(x) {
    console.log(x);
  }, function(y) {
    console.error(y);
  });
  ```

## 05. 手写数字格式化，比如输入9999999，输出9,999,999
  ```js
  function formatNumber(num) {
    //str.split('').reverse() => ["0", "9", "8", "7", "6", "5", "4", "3", "2", "1"]
    return num.toString().split('').reverse().reduce((prev,next,index) => {
      return ((index % 3) ? next : (next + ',')) + prev;
    })
  }

  console.log(formatNumber(1234567890)); // 1,234,567,890
  ```

## 06. 手写一个sleep函数
  - Promise 实现
    ```js
    const sleep = time => {
      return new Promise((resolve) => setTimeout(resolve, time));
    }
    sleep(1000).then(() => {
      console.log('hello');
    })
    ```
  - async 实现
    ```js
    function sleep(time) {
      return new Promise((resolve) => setTimeout(resolve, time))
    }

    async function output() {
      const out = await sleep(1000);
      
      console.log('hello');
      //return out;
    }

    output();
    ```
  - ES5 实现
    ```js
    function sleep(callback, time) {
      if (typeof callback === 'function') {
        setTimeout(callback, time);
      }
    }

    sleep(() => {
      console.log('hello');
    }, 1000);
    ```

## 07. 手写call、apply和bind
  - call
    ```js
    Function.prototype.myCall = function(context) {
      // 判断调用对象
      if (typeof this !== "function") {
        console.error("type error");
      }

      // 获取参数
      let args = [...arguments].slice(1),
        result = null;

      // 判断 context 是否传入，如果未传入则设置为 window
      context = context || window;

      // 将调用函数设为对象的方法
      context.fn = this;

      // 调用函数
      result = context.fn(...args);

      // 将属性删除
      delete context.fn;

      return result;
    };
    ```
  - apply
    ```js
    Function.prototype.myApply = function(context) {
      // 判断调用对象是否为函数
      if (typeof this !== "function") {
        throw new TypeError("Error");
      }

      let result = null;

      // 判断 context 是否存在，如果未传入则为 window
      context = context || window;

      // 将函数设为对象的方法
      context.fn = this;

      // 调用方法
      if (arguments[1]) {
        result = context.fn(...arguments[1]);
      } else {
        result = context.fn();
      }

      // 将属性删除
      delete context.fn;

      return result;
    };
    ```
  - bind
    ```js
    Function.prototype.myBind = function(context) {
      // 判断调用对象是否为函数
      if (typeof this !== "function") {
        throw new TypeError("Error");
      }

      // 获取参数
      var args = [...arguments].slice(1),
        fn = this;

      return function Fn() {
        // 根据调用方式，传入不同绑定值
        return fn.apply(
          this instanceof Fn ? this : context,
          args.concat(...arguments)
        );
      };
    };
    ```

## 08. 手写一个双向数据绑定
  - HTML
    ```html
    手机号： <input id="tel" type="tel" />
    <p>输入的手机号为：<span id="num"></span></p>
    ```
  - Object.defineProperty()
    ```js
    let obj = {
      tel: null
    };
    let newObj = JSON.parse(JSON.stringify(obj));

    Object.defineProperty(obj, 'tel', {
      get() {
        return newObj.tel;
      },
      set(val) {
        if (val === newObj.tel) return;

        newObj.tel = val;

        observer();
      }
    });

    function observer() {
      document.getElementById('num').innerHTML = obj.tel;
      document.getElementById('tel').value = obj.tel;
    }

    observer();

    setTimeout(() => {
      obj.tel = '15122222222';
    }, 1000);

    document.getElementById('tel').oninput = function () {
      obj.tel = this.value;
    }
    ```
  - Proxy
    ```js
    let obj = {};

    obj = new Proxy(obj, {
      get(target, prop) {
        return target[prop];
      },
      set(target, prop, value) {
        target[prop] = value;
        observer();
      }
    });

    function observer() {
      document.getElementById('num').innerHTML = obj.tel;
      document.getElementById('tel').value = obj.tel;
    }

    observer();

    setTimeout(() => {
      obj.tel = '15122222222';
    }, 1000);

    document.getElementById('tel').oninput = function () {
      obj.tel = this.value;
    }
    ```

## 09. 手写浅拷贝和深拷贝
  - 浅拷贝
    ```js
    function shallowCopy(object) {
      // 只拷贝对象
      if (!object || typeof object !== "object") return;

      // 根据 object 的类型判断是新建一个数组还是对象
      let newObject = Array.isArray(object) ? [] : {};

      // 遍历 object，并且判断是 object 的属性才拷贝
      for (let key in object) {
        if (object.hasOwnProperty(key)) {
          newObject[key] = object[key];
        }
      }

      return newObject;
    }
    ```
  - 深拷贝
    ```js
    function deepCopy(object) {
      if (!object || typeof object !== "object") return;

      let newObject = Array.isArray(object) ? [] : {};

      for (let key in object) {
        if (object.hasOwnProperty(key)) {
          newObject[key] = typeof object[key] === "object" ? deepCopy(object[key]) : object[key];
        }
      }

      return newObject;
    }
    ```

## 10. 手写函数柯里化
  函数柯里化指的是一个函数，它接收函数 A，并且能返回一个新的函数，这个新的函数能够处理函数 A 的剩余参数。
  - ES5实现
    ```js
    function curry(fn, args) {
      // 获取函数需要的参数长度
      let length = fn.length;

      args = args || [];

      return function() {
        let subArgs = args.slice(0);

        // 拼接得到现有的所有参数
        for (let i = 0; i < arguments.length; i++) {
          subArgs.push(arguments[i]);
        }

        // 判断参数的长度是否已经满足函数所需参数的长度
        if (subArgs.length >= length) {
          // 如果满足，执行函数
          return fn.apply(this, subArgs);
        } else {
          // 如果不满足，递归返回柯里化的函数，等待参数的传入
          return curry.call(this, fn, subArgs);
        }
      };
    }
    ```
  - ES6实现
    ```js
    function curry(fn, ...args) {
      return fn.length <= args.length ? fn(...args) : curry.bind(null, fn, ...args);
    }
    ```