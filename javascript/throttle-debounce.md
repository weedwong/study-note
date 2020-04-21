# JavaScript 节流和防抖函数

## 节流 (throttle)

> 对于一个多次连续不间断调用的函数，让其按照一定的时间间隔调用，以避免调用过于频繁，称之节流

### throttle 应用场景

- DOM 元素的拖拽功能实现（mousemove）
- 射击游戏的 mousedown/keydown 事件（单位时间只能发射一颗子弹）
- 计算鼠标移动的距离（mousemove）
- Canvas 模拟画板功能（mousemove
- 搜索联想（keyup
- 监听滚动事件判断是否到页面底部自动加载更多：给 scroll 加了 debounce 后，只有用户停止滚动后，才会判断是否到了页面底部；如果是 throttle 的话，只要页面滚动就会间隔一段时间判断一次

### throttle 实现

函数节流就是让连续执行的函数，变为固定时间段间断地执行。 大概有两种方式实现。

其一使用时间戳来判断是否已经到回调执行时间，记录上次执行的时间戳，然后每次触发事件时执行回调，回调中判断当前时间戳距离上次执行时间戳的时间间隔是否有*s，如果是，则执行，并更新上次执行的时间戳，如此循环。

我们以scroll事件为例，探究如何是实现滚动一次窗口打印一个hello world 字符串。 如果不对其节流或者去抖：

```js
window.onscroll = function () {
  console.log('hello world');
}
```

这样每滚动一次，实际上会打印多个 hello world。我们通过节流函数，十七每隔1000ms执行一次；

```js
var throttle = function(action, delay) {
  var last = 0;
  return function() {
    var curr = new Date();
    if (curr - last > delay) {
      action.apply(this, arguments);
      last = curr;
    }
  }
}
```

第二种方法是使用定时器，比如，当scroll事件刚触发时，打印一个hello world ，然后设置一个1000ms的定时器，此后每次触发scroll事件，触发回调，如果已经存在定时器，则回调不执行方法，直到定时器出发，handler被清除，然后重新设置定时器。

```js
function throttlePro(action, delay) {
  var tId;
  return function () {
    var context = this;
    var arg = arguments;
    if (tId) return;
    tId = setTimeout(function () {
      action.apply(context, arg);
      clearTimeout(tId);
      // setTimeout 返回一个整数，clearTimeout 之后，tId还是那个整数,setInterval同样如此
      tId = null;
    }, delay);
  }
}
```

## 防抖 (debounce)

### debounce 应用场景

- 每次 resize/scroll 触发统计事件
- 文本输入的验证（连续输入文字后发送 AJAX 请求进行验证，验证一次就好）

> 对于一个连续多次不间断调用的函数，只让其执行一次，称之防抖

### debounce 实现

函数去抖背后的思路是指，某些代码不可能在没有间断的情况下连续执行。创建一个定时器，在指定的时间间隔之后运行代码。当第二次调用该函数时，它会清除前一次的定时器并设置另一个。如果前一个定时器已经执行过了，这个操作就没有任何意义。然而，如果前一个定时器尚未执行，其实就是将其替换为一个新的定时器。目的是只有在执行函数的请求停止了一段时间之后才执行。

经典案例：

```js
function debounce(method, context) {
  clearTimeout(method.tId);
  method.tId = setTimeout(function() {
    method.call(context);
  }, 1000);
}

function print() {
  console.log('hello world');
}

window.onscroll = function() {
  debounce(print);
};
```

利用闭包实现：

```js
function debounce(delay, action) {
  var tId;
  return function () {
    var context = this;
    var arg = arguments;
    if (tId) clearTimeout(tId);
    tId = setTimeout(function () {
      action.apply(context, arg);
    }, delay);
  }
}

window.onscroll = debounce(1000, print);
```

参考：[JavaScript 函数节流和函数去抖]()