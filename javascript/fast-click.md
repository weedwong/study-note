# FastClick.js

## FastClick.js是什么

    用来解决移动端点击事件延迟300ms触发的问题；

    移动端浏览器在点击时要进行一次缩放判断，如果你是快速的双击，就应该去缩放页面，所以会有300ms延迟来进行该判断。

## 除此之外还可以通过以下方法处理延迟问题

1. 设置<meta name="viewport" content="user-scalable=no" />

通过此设置禁用浏览器缩放功能，缺点是浏览器无法缩放，双指缩放也会失效

2. 设置<meta name="viewport" content="width=device-width, initial-scale=1">

较高级的浏览器都会在看到这个meta标签的时候都会移除300ms的点击判断。
缺点大概就是兼容性；如果你需要兼容很低版本的浏览器，最好先测试一下这个标签能不能有效果

3. 使用zepto的tap事件

从原理上说是监听touchstart和touchend 在结束的时候模拟一个tap事件并触发他，来获得与touchend同时相应的效果。
会有点击穿透的问题（这个问题可以百度google一下，不详细说了）就是因为并没有做事件的取消，导致上层tap后，300ms过后点击事件作用在了下层元素上。
并且不利于做移动pc端的统一，需要给两个端绑定不同的事件，比较麻烦

4. 使用fastclick库

从原理上说也是监听touchstart和touchend事件，在结束的时候模拟一个click事件并立即触发他，同时取消300ms后应该触发的click事件。
这个库很棒，但是会增加项目体积。也会出现一些focus事件上的小bug。


## 使用fastclick.js

```js
if ('addEventListener' in document) {
    document.addEventListener('DOMContentLoaded', function() {
        FastClick.attach(document.body);
    }, false);
}
```

## FastClick核心原理

监听元素的touch事件，阻止原生的click事件，并模拟合成一个click事件，通过`dispatchEvent`触发

