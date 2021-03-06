## Official document

- 仅推荐英文官网 [https://webpack.js.org/](https://webpack.js.org/)
- 截止目前的版本 [Latest release](https://github.com/webpack/webpack/releases/tag/v5.14.0)

## Tutorial materials

- 阮一峰老师 [ruanyf/webpack-demos](https://github.com/ruanyf/webpack-demos)

## Books and blogs

- 电子书和实体书 [gwuhaolin/dive-into-webpack/深入浅出 Webpack](https://github.com/gwuhaolin/dive-into-webpack)
- Webpack 4 和单页应用入门 [wallstreetcn/webpack-and-spa-guide](https://github.com/wallstreetcn/webpack-and-spa-guide)

## Table of Contents

1. [about webpack & webpack-cli](#demo-01--about-webpack--webpack-cli-source-code)

## Demo 01 : about webpack & webpack-cli ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo01)) [:top:](#table-of-contents)

```sh
ls
LICENSE  package.json  README.md
```

```sh
// 新建demo01
mkdir demo01
yarn init --y
```

初始化项目，新建工具方法，分别使用`esm` `commonjs`

```sh
mkdir src && cd src && touch commonjs.util.js && touch esm.util.js
```

```js
// commonjs.util.js
const add = (a, b) => a + b;

module.exports = {
  add,
};
```

```js
// esm.util.js
const sum = (x, y) => x + y;

export default sum;
```

In `index.html` We can load

```html
<body>
  <script src="./src/index.js"></script>
</body>
```

will `error`

```js
// xx Uncaught SyntaxError: Cannot use import statement outside a module
```

and

```js
// xx index.js:1 Uncaught ReferenceError: require is not defined
// at index.js:1
```

这时候借助 `webpack` 和 `webpack-cli` 来打包不同的`模块化语法` 可以参考[https://javascript.info/modules](https://javascript.info/modules)

```sh
cd demo01 && yarn add webpack -D
```

will `info`

```shell
webpack
CLI for webpack must be installed.
  webpack-cli (https://github.com/webpack/webpack-cli)

We will use "yarn" to install the CLI via "yarn add -D webpack-cli".
Do you want to install 'webpack-cli' (yes/no):
```

源码对应的位置

```js


/**
 * @typedef {Object} CliOption
 * @property {string} name display name
 * @property {string} package npm package name
 * @property {string} binName name of the executable file
 * @property {boolean} installed currently installed?
 * @property {string} url homepage
 */

/** @type {CliOption} */
const cli = {
	name: "webpack-cli",
	package: "webpack-cli",
	binName: "webpack-cli",
	installed: isInstalled("webpack-cli"),
	url: "https://github.com/webpack/webpack-cli"
};

if (!cli.installed) {
	const path = require("path");
	const fs = require("graceful-fs");
	const readLine = require("readline");

	const notify =
		"CLI for webpack must be installed.\n" + `  ${cli.name} (${cli.url})\n`;

	console.error(notify);

	let packageManager;

	if (fs.existsSync(path.resolve(process.cwd(), "yarn.lock"))) {
		packageManager = "yarn";
	} else if (fs.existsSync(path.resolve(process.cwd(), "pnpm-lock.yaml"))) {
		packageManager = "pnpm";
	} else {
		packageManager = "npm";
	}

	const installOptions = [packageManager === "yarn" ? "add" : "install", "-D"];

	console.error(
		`We will use "${packageManager}" to install the CLI via "${packageManager} ${installOptions.join(
			" "
		)} ${cli.package}".`
	);
// and so on
```

成功之后的打包结果

```shell
asset main.js 210 bytes [compared for emit] [minimized] (name: main)
orphan modules 53 bytes [orphan] 1 module
./src/index.js + 1 modules 175 bytes [built] [code generated]
./src/utils/commonjs.util.js 64 bytes [built] [code generated]

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/configuration/mode/

webpack 5.14.0 compiled with 1 warning in 209 ms
```

```js
// root/dist/main.js
(() => {
  var r = {
      35: (r) => {
        r.exports = { add: (r, t) => r + t };
      },
    },
    t = {};
  function e(s) {
    if (t[s]) return t[s].exports;
    var o = (t[s] = { exports: {} });
    return r[s](o, o.exports, e), o.exports;
  }
  (() => {
    "use strict";
    const { add: r } = e(35);
    r(1, 2);
  })();
})();
```

替换`index.html` 中的脚本

```html
<body>
  <!-- <script src="./src/index.js" type="module"></script> -->
  <script src="./dist/main.js"></script>
</body>
```

---

## Demo 02 : webpack.config.js & handle css module ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo02)) [:top:](#table-of-contents)

默认是`index.js` 指定入口, `api/cli` 通过命令行的方式

```zsh
npx webpack --entry ./src/main.js --output-path ./build
```

以上的方式也是可以的，但是不太方便

`webpack.config.js`

```js
module.exports = {
  entry: "./src/main.js",
  output: {
    // 把所有依赖的模块合并输出到一个 bundle.js 文件
    filename: "出口的文件名.js",
    // 输出文件都放到 dist 目录下(输出文件存放的本地目录必须是决定路径string类型)
    path: "", // 绝对路径
  },
};
```

**ERROR**

```shell
Invalid configuration object. Webpack has been initialized using a configuration object that does not match the API schema.
 - configuration.output.path: The provided value "" is not an absolute path!   -> The output directory as **absolute path** (required).
```

提供配置文件的路径 `new.config.js`

```json
{
  "build": "webpack --config new.config.js"
}
```

没有使用过的文件是不会被打包进去的,接着我们处理`CSS` 文件

**ERROR**

```shell
ERROR in ./src/styles/index.css 1:4
Module parse failed: Unexpected token (1:4)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process
this file. See https://webpack.js.org/concepts#loaders
```

我们需要`css-loader` 的能力来处理 css 文件。这样可以正常引入`css`文件 **但是并没有应用到页面文档中** 还需要`style-loader` 样式才会生效

```shell
yarn add css-loader style-loader -D
```

通常情况下我们是使用`预处理器` 譬如`sass(scss)` ,预处理文件

```scss
div {
  span {
  }
}
```

处理这种文件需要对应的编译

```sh
yarn add sass -D
yarn add sass-loader -D

```

```shell
ERROR in ./src/styles/index.scss 1:3
Module parse failed: Unexpected token (1:3)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process
this file. See https://webpack.js.org/concepts#loaders> div{
|     span{
|
 @ ./src/main.js 2:0-28
```

## Demo 03 : postCSS & autoprefixer ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo03)) [:top:](#table-of-contents)

问：什么是`postcss`
答：通过`js` 代码对样式进行转换、适配
作用：自动添加浏览器的前缀、css 样式的重置

1. 构建工具中的扩展
2. 添加相关的插件

```sh
yarn add postcss postcss-cli autoprefixer -D
```

```sh
npx postcss --use autoprefixer -o result.css ./src/styles/index.css
```

通常在工程化的项目中，比如

```css
:root {
  --mainColor: #12345678;
}
```

把一些特定的样式描述转换为 rgba,并且还会添加样式的前缀

```css
/* becomes */

:root {
  --mainColor: rgba(18, 52, 86, 0.47059);
}
```

可以直接项目的根目录下`touch postcss.config.js`

```js
module.exports = {
  plugins: [require("postcss-preset-env")],
};
```

## Demo 04 : assets ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo04)) [:top:](#table-of-contents)

加载依赖的图片,**ERROR**

```sh
ERROR in ./src/assets/images/test.png 1:0
Module parse failed: Unexpected character '�' (1:0)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
(Source code omitted for this binary file)
 @ ./src/index.js 6:10-45
```

没有对应的文件处理`loader`

```js
const div = document.createElement("div");
const img = new Image();

img.src = require("./assets/images/test.png");

div.appendChild(img);
```

```js
GET http://127.0.0.1:5500/demo04/[object%20Module] 404 (Not Found)
```

把相关的资源文件复制到`dist` 目录下，然后进行重命名

```js

{
  // 对非文本文件采用 file-loader 加载
  test: /\.(gif|png|jpe?g|eot|woff|ttf|svg|pdf)$/,
  use: ["file-loader"],
},
```

## Demo 05 : assets module type ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo05)) [:top:](#table-of-contents)

资源模块类型子在`webpack5` 中新增了一种方式，我们不用使用之前的 loader 直接配置`type` 类型（资源的类型）

```js
{
    test:/\.(png|jpe?g|gif|svg)$/,
    type:'asset/resource',

    // loader:'file-loader'
    // use:[
    //     {
    //         loader:'file-loader',
    //         options:{
    //             name: 'img/[name].[hash:8].[ext]',
    //         }
    //     }

    // ]
}
```

对于`字体元素` file-loader 可以加载

```js
{
  // 对非文本文件采用 file-loader 加载
  test: /\.(gif|png|jpe?g|eot|woff|ttf|svg|pdf)$/,
  use: ["file-loader"],
},
```

## Demo 06 : about plugins ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo06)) [:top:](#table-of-contents)

通过插件生成`index.html` 文件

## Demo 07 : Modular principle & mode ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo07)) [:top:](#table-of-contents)

允许使用各种模块化（esModule commonjs）

- commonjs 模块化的原理
- esm 的原理
- commonjs 加载 esm
- esm 加载 commonjs

### commonjs 的原理

```js
// dist/bundle.js
(() => {
  var __webpack_modules__ = {
    "./src/js/commonjs.js": (module) => {
      function print() {
        console.log(`commonjs`);
      }
      // 将导出的变量放入到module对象exports
      module.exports = {
        print,
      };
    },
  };
  // The module cache
  var __webpack_module_cache__ = {};

  // The require function
  function __webpack_require__(moduleId) {
    // 判断缓存中是否加载过对应moduleId的模块
    if (__webpack_module_cache__[moduleId]) {
      return __webpack_module_cache__[moduleId].exports;
    }
    // 创建一个module对象，并把它放入缓存中
    var module = (__webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {},
    });

    // 加载对应的模块
    // 参数一
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // 导出一个对象 {}
    return module.exports;
  }

  (() => {
    /*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
    __webpack_require__(/*! ./js/commonjs */ "./src/js/commonjs.js");
  })();
})();
```

### ems 模块化原理

```js
(() => {
  // 启动脚本
  var __webpack_modules__ = {
    "./src/index.js": (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      "use strict";
      __webpack_require__.r(__webpack_exports__);
      var _js_esm_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(
        /*! ./js/esm.js */ "./src/js/esm.js"
      );
      /* harmony import */ var _js_esm_js__WEBPACK_IMPORTED_MODULE_0___default = /*#__PURE__*/ __webpack_require__.n(
        _js_esm_js__WEBPACK_IMPORTED_MODULE_0__
      );
      // require('./js/commonjs')
    },

    "./src/js/esm.js": () => {
      const print = () => {
        console.log(`esm`);
      };
    },
  };
  // 模块缓存处理
  var __webpack_module_cache__ = {};

  //  __webpack_require__ webpack 加载函数
  function __webpack_require__(moduleId) {
    // 检查模块是否缓存
    if (__webpack_module_cache__[moduleId]) {
      return __webpack_module_cache__[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = (__webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {},
    });

    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;
  }

  (() => {
    // getDefaultExport function for compatibility with non-harmony modules
    __webpack_require__.n = (module) => {
      var getter =
        module && module.__esModule ? () => module["default"] : () => module;
      __webpack_require__.d(getter, { a: getter });
      return getter;
    };
  })();

  /* webpack/runtime/define property getters */
  (() => {
    // define getter functions for harmony exports
    __webpack_require__.d = (exports, definition) => {
      for (var key in definition) {
        if (
          __webpack_require__.o(definition, key) &&
          !__webpack_require__.o(exports, key)
        ) {
          Object.defineProperty(exports, key, {
            enumerable: true,
            get: definition[key],
          });
        }
      }
    };
  })();

  /* webpack/runtime/hasOwnProperty shorthand */
  (() => {
    __webpack_require__.o = (obj, prop) =>
      Object.prototype.hasOwnProperty.call(obj, prop);
  })();

  /* webpack/runtime/make namespace object */
  (() => {
    // define __esModule on exports
    __webpack_require__.r = (exports) => {
      if (typeof Symbol !== "undefined" && Symbol.toStringTag) {
        Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
      }
      Object.defineProperty(exports, "__esModule", { value: true });
    };
  })();

  // startup
  // Load entry module
  __webpack_require__("./src/index.js");
  // This entry module used 'exports' so it can't be inlined
})();
//# sourceMappingURL=bundle.js.map
```

我们仍然可以交替加载

```js
// index.js
const commonjs = require("./js/commonjs");

import esm from "./js/esm";
```

## Demo 08 : source-map & Devtool([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo08)) [:top:](#table-of-contents)

当 webpack 打包源代码时，可能会很难追踪到 error(错误) 和 warning(警告) 在源代码中的原始位置。

- [https://webpack.docschina.org/configuration/devtool/#devtool](https://webpack.docschina.org/configuration/devtool/#devtool)

不同的值，打包的速度不尽相同

```json
{
  "version": 3, // 版本
  "sources": ["webpack://demo08/./src/index.js"], // 来自哪里
  "names": [], // 转换之前的变量集合、属性名称
  "mappings": ";;;;AAAA,c", // source-map
  "file": "bundle.js", // 打包之后的文件
  "sourcesContent": ["console.log(a)"], // 源代码
  "sourceRoot": "" // 相对的目录
}
```

- false 不生成 source-map 文件
- eval 转成字符串 方便在开发环境下调试错误的代码等等 (函数中)
- none 生产环境下默认值 (none)（省略 devtool 选项） - 不生成 source map。
- inline-source-map

```js
//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly9kZW1vMDgvLi9zcmMvaW5kZXguanMiXSwibmFtZXMiOltdLCJtYXBwaW5ncyI6Ijs7OztBQUFBLGMiLCJmaWxlIjoiYnVuZGxlLmpzIiwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coYSkiXSwic291cmNlUm9vdCI6IiJ9
```

在开发阶段推荐使用`cheap-module-source-map` 测试环境也可以使用 `sorce-map` 或者是`cheap-module-source-map`

在发布阶段的话可以不写

## Demo 09 : babel ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo09)) [:top:](#table-of-contents)

Babel 是 JavaScript 编译器

```sh
  npm install --save-dev @babel/cli @babel/core @babel/preset-env
```

其中

- @babel/core babel @babel/cli 是 Babel 命令行转码工具，如果我们使用命令行进行 Babel 转码就需要安装它
- @babel/cli 脚手架方便 我们使用在终端使用命令 @babel/cli 依赖@babel/core，因此也需要安装@babel/core 这个 Babel 核心 npm 包
- @babel/preset-env 预设 集合了转换箭头函、es6 语法等等@babel/preset-env 这个 npm 包提供了 ES6 转换 ES5 的语法转换规则，我们在 Babel 配置文件里指定使用它。如果不使用的话，也可以完成转码，但转码后的代码仍然是 ES6 的，相当于没有转码

使用预设进行**编译**

```shell
npx babel index.js -o compiled.js --presets=@babel/env
```

### 插件和预设

- `@babel/plugin-transform-arrow-functions` 将所有的箭头函数转换为 ES5 函数表达式

- `@babel/preset-env` 此预设将包括所有插件以支持现代 JavaScript（ES2015，ES2016 等）。但是预设也可以选择。而不是从终端传递 cli 和预设选项
  一般情况下 我们是设置单独的配置文件，目前是采用多包管理

### polyfill

```js
   {
        "targets": {

        },
        /**
         * false 不用任何相关的代码
         *
         * usage 代码中用到什么引入什么
         * entry 入口文件导入
         */
        "useBuiltIns": "usage",
        "corejs": "3.6.5"
      }
```

## Demo 11 : typescript ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo11)) [:top:](#table-of-contents)

**error**

```sh
ERROR in ./src/index.ts
[tsl] ERROR
      TS18002: The 'files' list in config file 'tsconfig.json' is empty.
```

ts 的项目需要新建`tsconfig.json` `tsc --init`

## Demo 12 : eslint ([Source-code](https://github.com/yayxs/webpack-learn/tree/main/demo12)) [:top:](#table-of-contents)

生成一个文件（es 的配置文件）

## Loaders

### css-loader

其主要的作用就是当我们在文件中引入`css` 文件中的时候

- 内联的方式
- cli 的方式
- 配置的方式

```js
// webpack.config.js
module: {
  rules: [
    {
      test: /\.css$/i,
      use: [
    {
      loader:'style-loader'
    },
        {
          loader: "css-loader",
          options: {

          },
        },
      ],
    },
  ],
},
```

### style-loader

处理顺序为从后到前，即先交给 css-loader 处理，再把结果交给 style-loader。

```js
// webpack.config.js
use: [
  {
    loader: "style-loader",
  },
  // 处理顺序为从后到前，即先交给 css-loader 处理，
  // 再把结果交给style-loader。
  {
    loader: "css-loader",
    options: {},
  },
];
```

### sass-loader

```js

// webpack.config.js
{
    // 命中css预处理文件 .sass .scss
    test: /\.s[ac]ss$/i,
    use:[
      {
        loader:'style-loader'
      },
      {
        loader:'css-loader'
      },
      {
        loader:'sass-loader'
      }
    ]
  },
```

### postcss-loader

> Options for PostCSS as we reference these options twice
> Adds vendor prefixing based on your specified browser support in >package.json

```js

{
  loader: "postcss-loader",
  options: {
      postcssOptions:{
        plugins:[
          require('autoprefixer')
        ]
      }
  },
},
```

### file-loader

处理`requie` 引入的图片

> "file" loader makes sure those assets get served by WebpackDevServer.
> When you `import` an asset, you get its (virtual) filename.
> In production, they would get copied to the `build` folder.
> This loader doesn't use a "test" so it will catch all modules
> that fall through the other loaders.

```js
//
module: {
    rules: [
        {
            test:/\.(png|jpe?g|gif|svg)$/,
            loader:'file-loader'
        }
    ]
},
```

### url-loader

```js
// "url" loader works like "file" loader except that it embeds assets
// smaller than specified limit in bytes as data URLs to avoid requests.
// A missing `test` is equivalent to a match.
```

直接转换为`base64` 嵌入到打包后的`js`文件。

- 小的图片可以直接转换为`base64`

## babel-loader

```js
{
        // 命中 JavaScript 文件
        // test: /\.js$/,
        test: /\.m?js$/,
        // 用 babel-loader 转换 JavaScript 文件
        // ?cacheDirectory 表示传给 babel-loader 的参数，用于缓存 babel 编译结果加快重新编译速度
        // use: ["babel-loader?cacheDirectory"],
        exclude: /(node_modules|bower_components)/,
        use: [
          {
            loader: "babel-loader",
            options: {
              cacheDirectory: true, // 默认是false
              presets: ["@babel/preset-env"],
              plugins: ['@babel/plugin-transform-runtime']
            },
            // // enforce:'post' 的含义是把该 Loader 的执行顺序放到最后
            // // enforce 的值还可以是 pre，代表把 Loader 的执行顺序放到最前面
            // enforce: "post",
          },
        ],
        // 只命中src目录里的js文件，加快 Webpack 搜索速度
        include: path.resolve(__dirname, "src"),
      },
```

```js
rules: [{
  //
  test:/\.js$/,
  use:{
      loader:'babel-loader',
      options:{
          presets:[
              "@babel/preset-env"
          ]
      }
  }
}],
```

> Stage 0 - Strawman: just an idea, possible Babel plugin.
> Stage 1 - Proposal: this is worth working on.
> Stage 2 - Draft: initial spec.
> Stage 3 - Candidate: complete spec and initial browser >implementations.
> Stage 4 - Finished: will be added to the next yearly release.

## ts-loader

```js
  rules: [
      {
        test: /\.tsx?$/,
        loader: "ts-loader",
      },
    ],
```

## eslint-loader

## Plugins

## clean-webpack-plugin

```js
  plugins: [new CleanWebpackPlugin()],

```

## html-webpack-plugin

```js
// Generates an `index.html` file with the <script> injected.
```

## DefinePlugin

创建全局的常量 内置的插件

## Other

- 浏览器市场占有率 [can i use](https://www.caniuse.com/usage-table)

```js
// .browserslistrc
> 1%
last 2 versions
```

- 浏览器样式前缀 [autoprefixer](https://autoprefixer.github.io/)

- 转化现代的 css 特性 [postcss-preset-env](https://github.com/csstools/postcss-preset-env)

## typescript 配置

可以通过`ts-loader` 与 `babel-loader` 都可以编译 `ts`代码

[Babel vs tscfor TypeScript](https://www.typescriptlang.org/docs/handbook/babel-with-typescript.html#babel-vs-tsc-for-typescript)

- ts-loader 会有相关的类型校验
- babel-loader 不会
