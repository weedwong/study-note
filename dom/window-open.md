# window.open 打开新窗口被拦截

做项目的时候经常会用到wind.open打开新窗口，用来预览文件或者下载文档等。可是很多时候会被浏览器当成广告弹层给拦截了，故此记录一下原因以及解决方案。

原因是当浏览器检测到非用户操作产生的新弹出窗口，则会对其进行阻止。因为浏览器认为这可能是一个广告，不是一个用户希望看到的页面。

基于此原因，我们解决方案便是想法子骗过浏览器，让其以为是用户主动触发即可。

### 使用a标签跳转
该方法通过创建一个a标签，再模拟点击事件，触发链接跳转

```javascript
function newWin(url, id) {  
  var a = document.createElement(‘a‘);  
  a.setAttribute(‘href‘, url);  
  a.setAttribute(‘target‘, ‘_blank‘);  
  a.setAttribute(‘id‘, id);  
  // 防止反复添加  
  if(!document.getElementById(id)) {                       
      document.body.appendChild(a);  
  }  
  a.click();  
}  

function openUrl(url) {
  var a = $('<a href="'+url+'" target="_blank"></a>')[0];
  var e = document.createEvent('MouseEvents');
  e.initEvent('click', true, true);
  a.dispatchEvent(e);
}
```

```
这种方法也只能在同步代码中使用，若是异步代码还得想其他方法。
```

### 用window.open打开新窗口，然后修改此窗口地址，达到重定向效果

```javascript
xx.addEventListener(‘click‘, function () {
  // 打开页面，此处最好使用提示页面
  var newWin = window.open(‘loading page‘);

  ajax().done(function() {
    // 重定向到目标页面
    newWin.location.href = ‘target url‘;
  });
});
```

### 变异步为同步执行

```javascript
function openUrl() {
  var a = document.createElement(‘a‘); 
  a.setAttribute(‘target‘, ‘_blank‘);  
  a.setAttribute(‘id‘, id); 
  ajax().done(function(url) {
    // 重定向到目标页面
    a.setAttribute(‘href‘, url);
  });
  // 防止反复添加  
  if(!document.getElementById(id)) {                       
      document.body.appendChild(a);  
  }  
  a.click(); 
}
```