---
title: Webpack打包原理分析
date: 2020-09-05 14:18:21
tags:
  - webpack
categories: 
  - webpack
cover: 
abbrlink: 202011
---

## 前言

这两天学习了webpack的相关内容，在此总结一下webpack的打包原理。

## webpack模块

首先先来了解一下webpack的核心概念。

### 入口(entry)

入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。每个依赖项随即被处理，最后输出到称之为 bundles 的文件中。

### 出口(output)

output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。

### loader

loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。

### 插件(plugins)

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

## 思路

我们使用webpack的默认配置打包文件时会在dist文件夹下生产main.js，main.js内部就包含了bundle.js

```js
(function (modules) {
  var installedModules = {};

  function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    });
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    );
    module.l = true;
    return module.exports;
  }
  return __webpack_require__((__webpack_require__.s = "./index.js"));
})({
    "./index.js": function (module, exports) {
      eval(
          '// import a from "./a";\n\nconsole.log("hello word");\n\n\n//#sourceURL=webpack:///./index.js?'
        ),
        "./a.js": function (module, exports) {
          eval(
              '// import a from "./a";\n\nconsole.log("hello word");\n\n\n//#sourceURL=webpack:///./index.js?'
            ),
            "./b.js": function (module, exports) {
              eval(
                '// import a from "./a";\n\nconsole.log("hello word");\n\n\n//#sourceURL=webpack:///./index.js?'
              );
            }
        });
```

去掉注释之后大概就是这样的，首先这是一个自执行函数，传入参数其实就是我们打包文件的相关依赖的对应路径已经代码。

⼤概的意思就是，我们实现了⼀个webpack_require 来实现⾃⼰的模块化，把代码都缓存在installedModules⾥，代码⽂件以对象传递进来，key是路径，value是包裹的代码字符串，并且代码内部的require，都被替换成了webpack_require。

所以我们实现的步骤就是:

webpack的配置⽂件（默认的）
  + ⼊⼝（⼊⼝模块的位置）
  + 出⼝（⽣成bundle⽂件位置）

创建⼀个webpack
  + 接收⼀份配置（webpack.config.js）
  + 分析出⼊⼝模块位置
    + 读取⼊⼝模块的内容，分析内容
    + 哪些是依赖
    + 哪些是源码
      + 需要编译 -> 浏览器能够执⾏
    + 分析其他模块
  + 拿到对象数据结构
    + 模块路径
    + 处理好的内容
  + 创建bundle.js
    + 启动器函数，来补充代码⾥有可能出现的module exports require，让浏览器能够顺利的执⾏

## 主菜

项目目录如下
![wERVw4.png](https://s1.ax1x.com/2020/09/05/wERVw4.png)

bundle就是我们webpack构建的入口
```js
// bundle.js
//  执行webpack构建的入口
//  拿到webpack.config.js配置
const options = require("./webpack.config.js");
const webpack = require("./lib/webpack.js");

new webpack(options).run();
```

接下来简单的实现一下webpack.js
```js
const fs = require("fs");
module.exports = class webpack {
  constructor(options) {
    const { entry, output } = options;
    this.entry = entry;
    this.output = output;
    this.modules = [];
  }
  // 启动函数
  run(){
    // 分析入口模块内容
    const info = this.parse(this.entry);
  }
  // 分析入口模块的内容
  parse(entryFile){
    const content = fs.readFileSync(entryFile, "utf-8");
  }
}  
```
我们在bundle中new webpack传入了webpack.config.js给我们的webpack.js，我们通过解构得到入口entry和出口output，保存在内部。在parse函数中可以使用node中的fs读取模块的内容。

我们要拿到文件的依赖，这里推荐使用⽤@babel/parser，这是babel7的⼯具，来帮助我们分析内部的语法，包括es6，返回⼀个ast抽象语法树。
npm install @babel/parser --save
[@babel/parser](https://babeljs.io/docs/en/babel-parser)

所以我们安装并引入之后@babel/parser在parse方法内部添加
```js
const ast = parser.parse(content, {
  sourceType: "module",
});
```
就可以拿到ast了。大概就是这个样子
```
Node {
  type: 'File',
  start: 0,
  end: 90,
  loc: SourceLocation {
    start: Position { line: 1, column: 0 },
    end: Position { line: 4, column: 0 }
  },
  errors: [],
  program: Node {
    type: 'Program',
    start: 0,
    end: 90,
    loc: SourceLocation { start: [Position], end: [Position] },
    sourceType: 'module',
    interpreter: null,
    body: [ [Node], [Node], [Node] ],
    directives: []
  },
  comments: []
}
```

接下来我们使用⼀个模块@babel/traverse，来遍历ast
npm install @babel/traverse --save
引入时需要这么引入
```js
const traverse = require("@babel/traverse").default;
```

在parse内部加入traverse，内部保存一下我们后面需要到的路径。
```js
traverse(ast, {
  // 这是ast的类型名，表示import语句
  ImportDeclaration({ node }) {
    // 保存路径
    //   "./a.js" => "./src/a.js"
    const newPathName =
      "./" + path.join(path.dirname(entryFile), node.sourcvalue);
    // console.log(newPathName);
    
    dependencies[node.source.value] = newPathName;
  },
});
```

路径处理完成之后，我们把代码处理成浏览器可运⾏的代码，需要借助@babel/core，和@babel/preset-env，把ast语法树转换成合适的代码

```js
const babel = require("@babel/core");
const { code } = babel.transformFromAst(Ast, null, {
  // ast内容处理
  presets: ["@babel/preset-env"]
});
```
然后我们返回处理好的信息
```js
return {
  filename,
  dependencies,
  code
};
```
现在code就是处理好的，浏览器识别的代码。但浏览器还不是完全识别，后面我们再继续说这个问题。

根据上面的步骤，到现在为止，我们已经走完了以上几步
创建⼀个webpack
  + 接收⼀份配置（webpack.config.js）
  + 分析出⼊⼝模块位置
    + 读取⼊⼝模块的内容，分析内容
    + 哪些是依赖
    + 哪些是源码
      + 需要编译 -> 浏览器能够执⾏

接下来我们要分析其他模块，其实就是递归一下，比如说我们的index.js依赖a.js，但我们不知道a.js是不是还有依赖，所以要递归一下。
在run内部加入
```js
//递归分析其他的模块
this.modules.push(info);
for (let i = 0; i < this.modules.length; i++) {
  const item = this.modules[i];
  const { dependencies } = item;
  if (dependencies) {
    for (let j in dependencies) {
      // 使用parse处理dependencies
      this.modules.push(this.parse(dependencies[j]));
    }
  }
}
```
然后再把得到的数组转换成对象
```js
const obj = {};
this.modules.forEach(item => {
  obj[item.entryFile] = {
    dependencies: item.dependencies,
    code: item.code,
  };
});
this.file(obj);
```
最后我们创建file来处理require，module，exports，处理成浏览器可以完全理解的代码，最后在dist目录中生成文件。
```js
file(code) {
  //创建自运行函数，处理require,module,exports
  //生成main.js = >dist/main.js
  const filePath = path.join(this.output.path, this.outputfilename);
  // console.log(filePath);
  //require("./a.js")
  // this.entry = "./src/index.js"
  const newCode = JSON.stringify(code);
  const bundle = `(function(graph){
      function require(module){
          // 因为代码内部的require的路径为./a.js这种，但对象中key并不是这种形式
          // 处理code中的require路径，把./a.js处理成./src/a.js
          function reRequire(relativePath){
              return require(graph[module].dependencie[relativePath]) 
          }
          // 定义exports
          var exports = {};
          (function(require,exports,code){
              eval(code)
          })(reRequire,exports,graph[module].code)
          return exports;
      }
      // 传入入口模块
      require('${this.entry}')
  })(${newCode})`;
  // 把bundle和参数code写入文件
  fs.writeFileSync(filePath, bundle, "utf-8");
}
```
看注释就能看明白了，最后我们只需要node bundle.js就可以生成main.js了!!!

webpack.js完整代码
```js
const fs = require("fs");
const path = require("path");
// 拿到⽂件中依赖，这⾥我们不推荐使⽤字符串截取，引⼊的模块名越多，就越麻烦，不灵活，这⾥
// 我们推荐使⽤@babel/parser，这是babel7的⼯具，来帮助我们分析内部的语法，包括es6，返回
// ⼀个ast抽象语法树
const parser = require("@babel/parser");
// 接下来我们就可以根据body⾥⾯的分析结果，遍历出所有的引⼊模块，但是⽐较麻烦，这⾥还是
// 推荐babel推荐的⼀个模块@babel/traverse，来帮我们处理。
const traverse = require("@babel/traverse").default;

const { transformFromAst } = require("@babel/core");

module.exports = class webpack {
  constructor(options) {
    const { entry, output } = options;
    this.entry = entry;
    this.output = output;
    this.modules = [];
  }
  run() {
    //开始分析入口模块的内容
    const info = this.parse(this.entry);

    //递归分析其他的模块
    this.modules.push(info);
    for (let i = 0; i < this.modules.length; i++) {
      const item = this.modules[i];
      // 遍历
      const { dependencies } = item;
      if (dependencies) {
        for (let j in dependencies) {
          // 使用parse处理dependencies
          this.modules.push(this.parse(dependencies[j]));
        }
      }
    }
    const obj = {};
    this.modules.forEach(item => {
      obj[item.entryFile] = {
        dependencies: item.dependencies,
        code: item.code,
      };
    });
    // console.log(obj);
    this.file(obj);
  }
  parse(entryFile) {
    const content = fs.readFileSync(entryFile, "utf-8");
    
    // 分析内部的语法
    const ast = parser.parse(content, {
      sourceType: "module",
    });
    console.log(ast);
    
    // 以对象的形式保存路径
    const dependencies = {};

    // 遍历出所有的引⼊模块
    traverse(ast, {
      // 这是ast的类型名，表示import语句
      ImportDeclaration({ node }) {
        // 保存路径
        //   "./a.js" => "./src/a.js"
        const newPathName =
          "./" + path.join(path.dirname(entryFile), node.source.value);
        // console.log(newPathName);
        
        dependencies[node.source.value] = newPathName;
      },
    });
    const { code } = transformFromAst(ast, null, {
      presets: ["@babel/preset-env"],
    });

    return {
      entryFile,
      dependencies,
      code,
    };
  }
  file(code) {
    //创建自运行函数，处理require,module,exports
    //生成main.js = >dist/main.js
    const filePath = path.join(this.output.path, this.output.filename);
    // console.log(filePath);
    //require("./a.js")
    // this.entry = "./src/index.js"
    const newCode = JSON.stringify(code);
    const bundle = `(function(graph){
        function require(module){
            // 因为代码内部的require的路径为./a.js这种，但对象中的key并不是这种形式
            // 处理code中的require路径，把./a.js处理成./src/a.js
            function reRequire(relativePath){
                return require(graph[module].dependencies[relativePath]) 
            }
            // 定义exports
            var exports = {};
            (function(require,exports,code){
                eval(code)
            })(reRequire,exports,graph[module].code)
            return exports;
        }
        // 传入入口模块
        require('${this.entry}')
    })(${newCode})`;
    // 把bundle和参数code写入文件
    fs.writeFileSync(filePath, bundle, "utf-8");
  }
};
```

项目地址[webpack-bundle.js](https://github.com/Starlogzzz/webpack-bundle.js)




