---
title: Tree shaking
date: 2021-08-23 15:02:44
tags:
---
当 Javascript 项目达到一定体积时，将代码分成模块会更易于管理。但是，当这样做时，我们最终可能会导入实际上未使用的代码。Tree Shaking 是一种通过消除最终文件中未使用的代码来优化体积的方法。
我们来举个例子，下面是一个简单的 Javascript 文件，命名为 mathUtils.js，主要实现了基础的数学运算。
export function add(a, b) {
    console.log("add");
    return a + b;
}
export function minus(a, b) {
    console.log("minus");
    return a - b;
}
export function multiply(a, b) {
    console.log("multiply");
    return a * b;
}
export function divide(a, b) {
    console.log("divide");
    return a / b;
}
在 index.js 里，我们通过如下方式调用该文件：
import { add } from "./mathUtils";
add(1, 2);
假设我们正在使用像 webpack 这样的工具来打包 mathUtils.js，即使仅导入并使用了add()功能，我们也会看到文件中的所有功能都包含在最终输出中。
/***/ "./src/mathUtils.js":
/*!**************************!*\
  !*** ./src/mathUtils.js ***!
  \**************************/
/*! exports provided: add, minus, multiply, divide */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"add\", function() { return add; });\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"minus\", function() { return minus; });\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"multiply\", function() { return multiply; });\n/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, \"divide\", function() { return divide; });\nfunction add(a, b) {\n    console.log(\"add\");\n    return a + b;\n}\n\nfunction minus(a, b) {\n    console.log(\"minus\");\n    return a - b;\n}\n\nfunction multiply(a, b) {\n    console.log(\"multiply\");\n    return a * b;\n}\n\nfunction divide(a, b) {\n    console.log(\"divide\");\n    return a / b;\n}\n\n\n//# sourceURL=webpack:///./src/mathUtils.js?");
/***/ })
Tree Shaking 是如何工作的
虽然 Tree Shaking 的概念早在1990年代就已经被提出。
Dead code elimination in dynamic languages is a much harder problem than in static languages. The idea of a "treeshaker" originated in LISP in the 1990s.—— wikipedia
但当真正作用到 Javascript 中，是在 ES6 模块规范被提出之后，因为只有模块是通过 static 方式引用时，Tree Shaking 才会起作用。
在 ES6 模块规范之前，我们使用require()语法的 CommonJS 模块规范。这些模块是 dynamic 动态加载的，这意味着我们可以根据代码中的条件导入新模块。
var myDynamicModule;
if (condition) {
    myDynamicModule = require("foo");
} else {
    myDynamicModule = require("bar");
}
CommonJS 模块的这种 dynamic 性质意味着无法应用 Tree Shaking，因为在实际运行代码之前无法确定需要哪些模块。
在 ES6 中，引入了模块的新语法，这是 static 的。使用import语法，我们不再能够动态导入模块。
如下所示的代码是不被允许的：
if (condition) {
    import foo from "foo";
} else {
    import bar from "bar";
}
相反，我们必须在任何条件之外定义全局范围内的所有导入。
import foo from "foo";
import bar from "bar";
if (condition) {
    // do stuff with foo
} else {
    // do stuff with bar
}
除其他好处外，这种新语法还可以有效地 Tree Shaking，因为可以确定导入后使用的任何代码，而无需先运行这些代码。
Tree Shaking 究竟做了些什么
Tree Shaking 在 Webpack 中的实现，是用来尽可能的删除没有被使用过的代码，一些被 import 了但其实没有被使用的代码。
import { add, multiply } from "./mathUtils";
add(1, 2);
在上面的示例中，multiply()函数从未使用过，将从最终的打包文件中删除。
甚至从从未访问过的导入对象中删除特定属性。
myInfo.js
export const myInfo = {
    name: "Ire Aderinokun",
    birthday: "2 March"
}
index.js
import { myInfo } from "./myInfo.js";
console.log(myInfo.name);
在上面的示例中，birthday属性不会被输出到最终打包文件中，因为从未实际使用过。
但是，Tree Shaking 并不能消除 所有 未使用的代码。消除和不消除的细节不在本文讨论范围之内，但应注意的是，使用 Tree Shaking 并不能完全解决未使用代码的问题。
副作用
一个副作用是：有一些代码，是在 import 时执行了一些行为，这些行为不一定和任何导出相关。例如 polyfill ，Polyfills 通常是在项目中全局引用，而不是在 index.js 中使用导入的方式引用。
Tree Shaking 并不能自动判断哪些脚本是副作用，因此手动指定它们非常重要。
如何使用
Tree Shaking 通常是和打包工具配合使用，例如 Webpack，只需在配置文件中设置mode即可。
webpack.production.config.js
module.exports = {
    ...,
    mode: "production",
    ...,
};
要将某些文件标记为副作用，我们需要将它们添加到package.json文件中。
{
    ...,
    "sideEffects": [
        "./src/polyfill.js"
    ],
    ...,
}
删除代码的原理：webpack基于ES6提供的模块系统，对代码的依赖树进行静态分析，把import & export标记为3类：
• 所有import标记为/* harmony import */
• 被使用过的export标记为/harmony export([type])/，其中[type]和webpack内部有关，可能是binding，immutable等；
• 没有被使用的export标记为/* unused harmony export [FuncName] */，其中[FuncName]为export的方法名，之后使用Uglifyjs（或者其他类似的工具）进行代码精简，把没用的都删除。