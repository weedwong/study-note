# webpack使用手册

## webpack是什么

webpack是JavaScript项目最流行的打包器，可以把非js文件打包成模块给js调用。

## 怎么使用？？？

最简单的是安装之后直接在命令行使用

1. 安装

   ```bash
   npm install webpack -g
   ```

2. 使用

   先新建一个工程项目`webpack-first`

   ```bash
   npm init webpack-first
   ```

   接下来我们在工程中创建三个文件：`index.html`、`main.js` 和 `module.js`。
   index.html

   ```html
   <html lang="zh-CN">
     <body>
       <script src="dist/bundle.js"></script>
     </body>
   </html>
   ```

   main.js
   
   ```javascript
   import myModule from './module.js';
   document.write('this is main.js');
   myModule();
   ```
   
   module.js
   
   ```javascript
   export default function() {
      document.write('this is module.js');
   }
   ```

   执行 Webpack 命令来生成我们的第一个打包结果：
   
   ```bash
   webpack main.js dist/bundle.js
   ```

   然后在浏览器打开`index.html`,`main.js` 和 `module.js`已经被打包到`dist/bundle.js`中并被`index.html`文件加载了
<!-- 3. 

4. Gggggg -->
