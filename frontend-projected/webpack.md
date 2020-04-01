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
   webpack main.js -o dist/bundle.js

   <!-- main.js 是入口文件， bundle.js 是打包出来的文件 -->
   <!-- 因为webpack版本不一样，命令后面后的 -o 有时是必须有时不是 -->
   ```

   然后在浏览器打开`index.html`,可以看到`main.js` 和 `module.js`已经被打包到`dist/bundle.js`中并被`index.html`文件加载了

## 配置文件  

   上面我们用命令行实现了简单的打包操作，实际项目中会用到很复杂的操作，就需要用到配置文件了。
   首先要在工程中创建一个`webpack.config.js`文件
   ```javascript
   const path = require('path');

   module.exports = {
      entry: './main.js',
      output: {
         path: path.join(__dirname, 'dist'),
         filename: 'bundle.js'
      }
   }

   /*
   *  entry属性是入口文件
   *  output属性是打包出来的文件
   */ 
   
   ```
   定义好配置文件后，在工程中执行打包命令 `webpack`，跟上的命令执行结果一样
   
## 配置本地开发服务

   在进行本地开发的时候通过`webpack-dev-server`起一个本地服务，可以实现监听本地文件的改动，重新进行打包，并实时刷新浏览器等功能
   
   首先通过`npm install webpack-dev-server --save-dev`命令，安装`webpack-dev-server`，然后修改配置文件为：
   
   ```javascript
   const path = require('path');

   module.exports = {
      entry: './main.js',
      output: {
         path: path.join(__dirname, 'dist'),
         filename: 'bundle.js'
      },
      devServer: {
         port: 3000, // 服务端口
         publicPath: "/dist/" // 打包后资源路径
      }
   }
   
   ```

   接着执行命令 `node_modules/.bin/webpack-dev-server` 我们就可以在浏览器访问 `localhost:3000`了

## 配置加载器

>  一切皆模块,加载器就是把所有的资源加载为js模块，所有加载器是配置在`module`属性中（我猜的）

   如果项目中需要打包css,图片等文件，这个时候就需要用到加载器`loader`,`loader` 相当于扩展了`webpack`的能力，让它能处理更多类型的文件

   现在项目中新建一个 `style.css`

   ```css
   body {
      text-align: center;
      padding: 100px;
      color: #fff;
      background-color: #ff9;
   }
   ```
   
   更改 `main.js`

   ```javascript
      import myModule from './module.js';
      import './style.css';
      document.write('this is main.js');
      myModule();
   ```

   由于 `loader` 是独立的功能，所以处理不同类型的文件需要在项目中安装对应的加载器。这里通过`npm i css-loader --save-dev` 安装 `css-loader`，是专门处理`.css` 的加载器。

   然后修改配置文件 `webpack.config.js`

   ```javascript
   const path = require('path');

   module.exports = {
      entry: './main.js',
      output: {
         path: path.join(__dirname, 'dist'),
         filename: 'bundle.js'
      },
      module: {
         loaders: [
               {
                  test: /\.css$/, // 匹配文件类型
                  loader: 'css-loader' // 处理器名字
               }
         ]
      }
   }
   ```

   重新打包后，刷新页面，其实样式并没有应用上，这是因为 `css-loader` 只处理了 `.css` 文件，并未将样式应用倒也没中。我们还需要一个加载器 `style-loader` 才能解决这个问题.

   先通过 `npm i style-loader --save-dev` 命令安装 `style-loader`，然后修改配置文件`webpack.config.js` 为

   ```javascript
   const path = require('path');

   module.exports = {
      entry: './main.js',
      output: {
         path: path.join(__dirname, 'dist'),
         filename: 'bundle.js'
      },
      module: {
         loaders: [
               {
                  test: /\.css/,
                  loader: 'style-loader!css-loader'
               }
         ]
      }
   }
   ```

   这样一个最基本的依赖打包配置就完成了

## 配置插件

>  插件可以对原有代码做一些特殊处理，比如代码压缩

   这里先配置一个插件用来做资源压缩

   首先用命令`npm i uglifyjs-webpack-plugin --save-dev`

   接着修改配置文件

   ```javascript
   const path = require('path');
   const UglifyJSPlugin = require('uglifyjs-webpack-plugin')

   module.exports = {
      entry: './main.js',
      output: {
         path: path.join(__dirname, 'dist'),
         filename: 'bundle.js'
      },
      module: {
         loaders: [
               {
                  test: /\.css/,
                  loader: 'style-loader!css-loader'
               }
         ]
      },
      plugins: [
         new UglifyJSPlugin()
      ]
   }
   ```

   这样我们打包出来的文件就是压缩过的了，代码的可读性差但是体积小很多

## 按需加载

   `Webpack` 支持异步加载模块的特性，从原理上说其实很简单——就是动态地向页面中插入 script 标签。在代码层面，Webpack 支持两种方式进行异步模块加载，一种是 CommonJS 形式的 require.ensure，一种是 ES6 Module 形式的异步 import()。

   首先加载一个新的模块`log.js`,

   ```javascript
   export const log = function() {
    console.log('this is log.js.'); // 异步加载要注意 不能直接使用 document.write
   }
   ```

   修改`main.js`：

   ```javascript
   import myModule from './module.js';
   import('./log.js').then(module => {
    module.log();
   }).catch(error => 'An error occurred while loading the module');
   document.write('this is main.js');
   myModule();

   ```

   修改配置文件

   ```javascript
   const path = require('path');
   const UglifyJSPlugin = require('uglifyjs-webpack-plugin')

   module.exports = {
      entry: './main.js',
      output: {
         path: path.join(__dirname, 'dist'),
         publicPath: './dist/', // 工程中有按需加载以及图片和文件等外部资源时就需要它来配置这些资源的路径
         filename: 'bundle.js'
      },
      module: {
         loaders: [
               {
                  test: /\.css/,
                  loader: 'style-loader!css-loader'
               }
         ]
      },
      plugins: [
         new UglifyJSPlugin()
      ]
   }
   ```

   重新打包之后你会发现打包结果中多出来一个 `0.bundle.js`，这里面就是将来会被异步加载进来的内容

## 其他配置

   1. tree-shaking
   2. scope-hoisting
   3. devServer.proxy
   4. 公共模块
   5. 多页应用打包
   6. loader option
   7. sourceMap