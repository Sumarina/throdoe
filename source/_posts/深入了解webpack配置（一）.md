---
title: 深入了解webpack配置（一）
toc: true
comments: true
copyright: true
date: 2020-03-27 15:19:19
tags: ['webpack']
categories: '程序员'
---

## webpack 是什么

webpack 是一个静态资源构建工具，主要是把我们的应用程序所需要的模块构建成一个或者多个 bundles。在构建过程中，webpack 会递归循环创建依赖关系图。

## webpack 的核心概念

1. entry（入口）：指示 webpack 构建的入口文件，也就是构建依赖关系图的开始。（默认值为`./src`）
1. output（输出）：输出构建之后的 bundles（可以修改输出的文件名，默认值为`./dist`）
1. loader:将 webpack 不能识别的模块转为能够识别的有效模块
1. plugins(插件)：在 webpack 整个构建过程中注入扩展逻辑来改变构建结果或做你想要做的事情

## demo 配置

`使用 webpack 必须安装 webpack 和 webpack-cli（webpack 使用的是 4.x 的版本）`
在官网很明确的指出 webpack v4.0.0 后的版本是开箱即用。但仍还是高度可配置的。
在 src 文件新建一个 index.js 文件【webpack 默认入口，如若放其他目录，需修改配置文件中 entry 的值】。index 文件的内容如下：

```bash
const newArr = [
  ...[1, 2, 3].map(value => {
    return value * 2;
  })
];
```

执行`npx webpack --mode=development` (未在 package.json 中配置命令,直接使用 npx 代替 npm,在 npm 5.2 之后的版本增加 npx 命令，方便用户能够调用项目内部安装的模块)
构建完成后 index 里面的内容变成如下代码：

```bash
/******/ ({
/*! no static exports found */
(function(module, exports) {
eval("const newArr = [\n  ...[1, 2, 3].map(value => {\n    return value * 2;\n  })\n];\n//# sourceURL=[module]\n//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9zcmMvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9zcmMvaW5kZXguanM/YjYzNSJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zdCBuZXdBcnIgPSBbXG4gIC4uLlsxLCAyLCAzXS5tYXAodmFsdWUgPT4ge1xuICAgIHJldHVybiB2YWx1ZSAqIDI7XG4gIH0pXG5dO1xuIl0sIm1hcHBpbmdzIjoiQUFBQTtBQUNBO0FBQ0E7QUFDQTtBQUNBOyIsInNvdXJjZVJvb3QiOiIifQ==\n//# sourceURL=webpack-internal:///./src/index.js\n");

 })
});
```

index 中的内容并没有如我们所想转化为低版本的代码，这里就需要`babel-loader`
安装 babel-loader 以及相关依赖@babel/core、@babel/preset-env @babel/plugin-transform-runtime、@babel/runtime、@babel/runtime-corejs3。
新建 webpack.config.js 文件。

```bash
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: ['babel-loader'],
        options: {
          presets: ['@babel/preset-env'],
          plugins: [
            [
              '@babel/plugin-transform-runtime',
              {
                corejs: 3
              }
            ]
          ]
        },
        exclude: /node_modules/ //排除node_modules文件
      }
    ]
  }
```

也可以新建.babelrc 文件，添加配置。

```bash
{
  "presets": ["@babel/preset-env"],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": 3
      }
    ]
  ]
}
```

配置完成后，再次执行构建命令，输出内容如下：

```bash
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _babel_runtime_corejs3_core_js_stable_instance_map__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! @babel/runtime-corejs3/core-js-stable/instance/map */ \"./node_modules/_@babel_runtime-corejs3@7.9.2@@babel/runtime-corejs3/core-js-stable/instance/map.js\");\n/* harmony import */ var _babel_runtime_corejs3_core_js_stable_instance_map__WEBPACK_IMPORTED_MODULE_0___default = /*#__PURE__*/__webpack_require__.n(_babel_runtime_corejs3_core_js_stable_instance_map__WEBPACK_IMPORTED_MODULE_0__);\n/* harmony import */ var _babel_runtime_corejs3_helpers_toConsumableArray__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(/*! @babel/runtime-corejs3/helpers/toConsumableArray */ \"./node_modules/_@babel_runtime-corejs3@7.9.2@@babel/runtime-corejs3/helpers/toConsumableArray.js\");\n/* harmony import */ var _babel_runtime_corejs3_helpers_toConsumableArray__WEBPACK_IMPORTED_MODULE_1___default = /*#__PURE__*/__webpack_require__.n(_babel_runtime_corejs3_helpers_toConsumableArray__WEBPACK_IMPORTED_MODULE_1__);\n\n\n\nvar _context;\n\nvar newArr = _babel_runtime_corejs3_helpers_toConsumableArray__WEBPACK_IMPORTED_MODULE_1___default()(_babel_runtime_corejs3_core_js_stable_instance_map__WEBPACK_IMPORTED_MODULE_0___default()(_context = [1, 2, 3]).call(_context, function (value) {\n  return value * 2;\n}));//# sourceURL=[module]\n//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9zcmMvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9zcmMvaW5kZXguanM/YjYzNSJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zdCBuZXdBcnIgPSBbXG4gIC4uLlsxLCAyLCAzXS5tYXAodmFsdWUgPT4ge1xuICAgIHJldHVybiB2YWx1ZSAqIDI7XG4gIH0pXG5dO1xuIl0sIm1hcHBpbmdzIjoiOzs7Ozs7Ozs7O0FBQUE7QUFFQTtBQUNBIiwic291cmNlUm9vdCI6IiJ9\n//# sourceURL=webpack-internal:///./src/index.js\n");

```

现在成功构建成低版本的代码。
webpack 自身只理解 JavaScript，并不能够有效的识别其他文件，如`.css`以及图片。
这里就需要安装`style-loader、css-loader、postcss-loader、less-loader`。安装完成后，在 webpack.config.js 中配置，代码如下：

```bash
 module: {
    rules: [
      {
        test: /\.(le|c)ss$/,//
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              plugins: function() {
                return [
                  require('autoprefixer')({
                    overrideBrowserslist: ['>0.25%', 'not dead']
                  })
                ];
              }
            }
          },
          'less-loader'
        ],
        exclude: /node_module/
      }
    ]
  }
```

style-loader 动态创建 style 标签将 css 插入到 head 中。
css-loader 负责处理 @import 等语句。
postcss-loader 和 autoprefixer，自动生成浏览器兼容性前缀。
less-loader 负责处理编译 .less 文件,将其转为 css。
loader 的执行顺序是从右往左，执行顺序 less-loader->postcss-loader->css-loader->style-loader。

```bash
@color: red;
body {
  background-color: @color;
}
```

构建完成后如上样式代码会输出如下`background-color: red`

```bash
eval("// Imports\nvar ___CSS_LOADER_API_IMPORT___ = __webpack_require__(/*! ../../node_modules/_css-loader@3.4.2@css-loader/dist/runtime/api.js */ \"./node_modules/_css-loader@3.4.2@css-loader/dist/runtime/api.js\");\nexports = ___CSS_LOADER_API_IMPORT___(false);\n// Module\nexports.push([module.i, \"body {\\n  background-color: red;\\n}\\n\", \"\"]);\n// Exports\nmodule.exports = exports;\n//# sourceURL=[module]\n//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9ub2RlX21vZHVsZXMvX2Nzcy1sb2FkZXJAMy40LjJAY3NzLWxvYWRlci9kaXN0L2Nqcy5qcyEuL25vZGVfbW9kdWxlcy9fcG9zdGNzcy1sb2FkZXJAMy4wLjBAcG9zdGNzcy1sb2FkZXIvc3JjL2luZGV4LmpzPyEuL25vZGVfbW9kdWxlcy9fbGVzcy1sb2FkZXJANS4wLjBAbGVzcy1sb2FkZXIvZGlzdC9janMuanMhLi9zcmMvY3NzL21haW4ubGVzcy5qcyIsInNvdXJjZXMiOlsid2VicGFjazovLy8uL3NyYy9jc3MvbWFpbi5sZXNzP2VhMTEiXSwic291cmNlc0NvbnRlbnQiOlsiLy8gSW1wb3J0c1xudmFyIF9fX0NTU19MT0FERVJfQVBJX0lNUE9SVF9fXyA9IHJlcXVpcmUoXCIuLi8uLi9ub2RlX21vZHVsZXMvX2Nzcy1sb2FkZXJAMy40LjJAY3NzLWxvYWRlci9kaXN0L3J1bnRpbWUvYXBpLmpzXCIpO1xuZXhwb3J0cyA9IF9fX0NTU19MT0FERVJfQVBJX0lNUE9SVF9fXyhmYWxzZSk7XG4vLyBNb2R1bGVcbmV4cG9ydHMucHVzaChbbW9kdWxlLmlkLCBcImJvZHkge1xcbiAgYmFja2dyb3VuZC1jb2xvcjogcmVkO1xcbn1cXG5cIiwgXCJcIl0pO1xuLy8gRXhwb3J0c1xubW9kdWxlLmV4cG9ydHMgPSBleHBvcnRzO1xuIl0sIm1hcHBpbmdzIjoiQUFBQTtBQUNBO0FBQ0E7QUFDQTtBQUNBO0FBQ0E7QUFDQTsiLCJzb3VyY2VSb290IjoiIn0=\n//# sourceURL=webpack-internal:///./node_modules/_css-loader@3.4.2@css-loader/dist/cjs.js!./node_modules/_postcss-loader@3.0.0@postcss-loader/src/index.js?!./node_modules/_less-loader@5.0.0@less-loader/dist/cjs.js!./src/css/main.less\n");

```

同样，如果我们在 css 中使用了 image，那 webpack 无法识别怎么处理？那么我们需要安装`url-loader`，注意`url-loader`依赖`file-loader`,安装完 url-loader 和 file-loader 后，继续配置。

```bash
module: {
    rules: [
      {
        test: /\.(png|jpg|gif|jpeg|webp|svg|eot|ttf|woff|woff2)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              outputPath: 'assets',
              limit: 10240,//设置最大10kb，小于10kb的资源会转化为base64，大于10kb的资源则会拷贝到dist文件下
              esModule: false,
              name: '[name]_[hash:6].[ext]'
            }
          }
        ],
        exclude: /node_module/
      }
    ]
  }
```

至此，算是完成了一些基本配置。
