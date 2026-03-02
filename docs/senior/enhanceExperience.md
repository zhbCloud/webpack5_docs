# 提升开发体验

## SourceMap

### 1.为什么

开发时我们运行的代码是经过 webpack 编译后的，例如下面这个样子：

```js
/*
 * ATTENTION: The "eval" devtool has been used (maybe by default in mode: "development").
 * This devtool is neither made for production nor for readable output files.
 * It uses "eval()" calls to create a separate source file in the browser devtools.
 * If you are trying to read the output file, select a different devtool (https://webpack.js.org/configuration/devtool/)
 * or disable the default devtool with "devtool: false".
 * If you are looking for production-ready output files, see mode: "production" (https://webpack.js.org/configuration/mode/).
 */
/******/ (() => { // webpackBootstrap
/******/ 	"use strict";
/******/ 	var __webpack_modules__ = ({

/***/ "./node_modules/css-loader/dist/cjs.js!./node_modules/less-loader/dist/cjs.js!./src/less/index.less":
/*!**********************************************************************************************************!*\
  !*** ./node_modules/css-loader/dist/cjs.js!./node_modules/less-loader/dist/cjs.js!./src/less/index.less ***!
  \**********************************************************************************************************/
/***/ ((module, __webpack_exports__, __webpack_require__) => {

eval("__webpack_require__.r(__webpack_exports__);\n/* harmony export */ __webpack_require__.d(__webpack_exports__, {\n/* harmony export */   \"default\": () => (__WEBPACK_DEFAULT_EXPORT__)\n/* harmony export */ });\n/* harmony import */ var _node_modules_css_loader_dist_runtime_noSourceMaps_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ../../node_modules/css-loader/dist/runtime/noSourceMaps.js */ \"./node_modules/css-loader/dist/runtime/noSourceMaps.js\");\n/* harmony import */ var _node_modules_css_loader_dist_runtime_noSourceMaps_js__WEBPACK_IMPORTED_MODULE_0___default = /*#__PURE__*/__webpack_require__.n(_node_modules_css_loader_dist_runtime_noSourceMaps_js__WEBPACK_IMPORTED_MODULE_0__);\n/* harmony import */ var _node_modules_css_loader_dist_runtime_api_js__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(/*! ../../node_modules/css-loader/dist/runtime/api.js */ \"./node_modules/css-loader/dist/runtime/api.js\");\n/* harmony import */ var _node_modules_css_loader_dist_runtime_api_js__WEBPACK_IMPORTED_MODULE_1___default = /*#__PURE__*/__webpack_require__.n(_node_modules_css_loader_dist_runtime_api_js__WEBPACK_IMPORTED_MODULE_1__);\n// Imports\n\n\nvar ___CSS_LOADER_EXPORT___ = _node_modules_css_loader_dist_runtime_api_js__WEBPACK_IMPORTED_MODULE_1___default()((_node_modules_css_loader_dist_runtime_noSourceMaps_js__WEBPACK_IMPORTED_MODULE_0___default()));\n// Module\n___CSS_LOADER_EXPORT___.push([module.id, \".box2 {\\n  width: 100px;\\n  height: 100px;\\n  background-color: deeppink;\\n}\\n\", \"\"]);\n// Exports\n/* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = (___CSS_LOADER_EXPORT___);\n\n\n//# sourceURL=webpack://webpack5/./src/less/index.less?./node_modules/css-loader/dist/cjs.js!./node_modules/less-loader/dist/cjs.js");

/***/ }),
// 其他省略
```

所有 css 和 js 合并成了一个文件，并且多了其他代码。此时如果代码运行出错那么提示代码错误位置我们是看不懂的。一旦将来开发代码文件很多，那么很难去发现错误出现在哪里。

所以我们需要更加准确的错误提示，来帮助我们更好的开发代码。

### 2.是什么

SourceMap（源代码映射）是一个用来生成源代码与构建后代码一一映射的文件的方案。

它会生成一个 xxx.map 文件，里面包含源代码和构建后代码每一行、每一列的映射关系。当构建后代码出错了，会通过 xxx.map 文件，从构建后代码出错位置找到映射后源代码出错位置，从而让浏览器提示源代码文件出错位置，帮助我们更快的找到错误根源。

### 3.怎么用

通过查看[Webpack DevTool 文档](https://webpack.docschina.org/configuration/devtool/)可知，SourceMap 的值有很多种情况.

但实际开发时我们只需要关注两种情况即可：

#### **开发环境**

- 开发模式：`cheap-module-source-map`
  - 优点：打包编译速度快，只包含行映射
  - 缺点：没有列映射

```js
module.exports = {
  // 其他省略
  mode: "development",
  devtool: "cheap-module-source-map",
};
```

#### 生产环境

在生产环境中，是否配置 SourceMap 取决于**公司对性能、安全和调试便利性**的权衡。目前主流的做法有以下几种：

##### 1. 推荐：配置 `source-map`（最常见）

大多数中大型项目的 webpack.prod.js 都会配置 `devtool: 'source-map'`。

- **做法**：打包后会生成主 JS 文件（如 `main.js`）和一个对应的 `.map` 文件。

- **优点：**
  - **错误溯源**：当线上系统报错时，监控系统（如 Sentry, Fundebug）可以利用 `.map` 文件将压缩后的堆栈信息还原成源码行数。
  - **主包体积不受影响**：`.map` 文件是物理独立的，只有当开发者打开控制台并手动加载它时，浏览器才会下载，普通用户访问时不会下载。
  
- **缺点**：**打包编译速度更慢，源码暴露风险**。由于 `.map` 文件通常也部署在服务器上，懂行的用户可以通过控制台直接查看到你的前端源码。

  ![image-20260228165327466](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228165327466.png)

  `//# sourceMappingURL` 的核心作用就是：**告诉浏览器去哪里下载对应的 SourceMap 文件，从而实现生产代码到开发源码的精准映射。**

---

##### 2. 追求安全：`hidden-source-map`

如果你既想在后台监控错误，又不希望用户在浏览器控制台看到源码，这是最佳方案。

- **特点**：会生成 `.map` 文件，但打包后的 JS 文件结尾**不会产生** `//# sourceMappingURL=...` 的引用注释。
- **效果**：浏览器控制台不会自动关联源码，报错只显示压缩后的行数。但你可以把生成的 `.map` 文件手动上传到 Sentry 等私有监控平台，实现内部调试。

---

##### 3. 极致安全：不配置（`none`）

如果项目非常敏感（比如金融、保密项目），或者是一个极简的小项目，可以选择完全不配置。

- **特点**：不生成任何映射文件。
- **效果**：压缩后的代码完全无法调试。如果线上出 bug，只能通过“盲猜”或在本地模拟数据来复现。

---

##### 4. 常见的线上排查方案（大厂方案）

很多公司采用 **“服务器加锁/权限控”** 的方案：

1. **打包生成 SourceMap**。
2. **不随代码上传到 CDN/公网服务器**。
3. **内网部署**：只把 `.map` 文件发布到公司内网的一台专用调试服务器，或者直接上传到私有的错误监控平台。

---

##### 总结建议

- **中小型项目/后台管理系统**：直接用 `source-map`，调试效率第一。
- **大型互联网产品/C端应用**：用 `hidden-source-map` + Sentry/私有监控，兼顾安全与调试。
- **个人练习项目**：建议配置 `source-map`。

## hidden-source-map 源码溯源

> 本方案将指导你如何在 Webpack 5 项目中集成 Sentry，实现生产环境hidden-source-map下的自动错误捕获和源码还原。
>

### 1. 准备工作

#### 1.1 注册 Sentry 账号

访问 [Sentry.io](https://sentry.io/) 首页进行注册。

#### 1.2 创建项目（Project）

选择 **Browser JavaScript**（根据自己实际项目类型选择）。

![image-20260228151819711](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228151819711.png)

![image-20260228152001432](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228152001432.png)

#### 1.3 设置 Sentry SDK

![image-20260228152054610](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228152054610.png)

#### 1.4 获取 DSN

DSN 格式类似 `https://xxxxxx@sentry.io/12345`。

![image-20260228152309729](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228152309729.png)

或者选择项目后，将鼠标悬停在设置图标上查看：
![image-20260228155828037](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228155828037.png)

---

### 2. 安装依赖

在项目根目录下执行：

```bash
npm install @sentry/browser
npm install @sentry/webpack-plugin --save-dev
```

---

### 3. 代码集成

#### 3.1 SDK 初始化

在入口文件（如 `src/main.js`）的顶部引入并初始化。

**`main.js`**

```js
import * as Sentry from "@sentry/browser";

Sentry.init({
  dsn: "你的 DSN 地址",
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration(),
  ],
  // 性能监控采样率
  tracesSampleRate: 1.0,
  // Session Replay 采样率
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});
```

---

### 4. Webpack 配置自动上传 SourceMap

这是生产环境最重要的步骤，确保 Sentry 能根据混淆代码还原源码。

**`webpack.prod.js`**

```js
const { sentryWebpackPlugin } = require("@sentry/webpack-plugin");

module.exports = {
  mode: "production",
  devtool: "hidden-source-map", // ✨ 安全推荐：生成 map 但不添加关联注释，防止浏览器控制台泄露源码
  plugins: [
    // ... 其他插件
    sentryWebpackPlugin({
      authToken: "你的 Sentry Token",
      org: "你的组织名",
      project: "你的项目名",
      sourcemaps: {
        assets: ["./dist/**"],
        ignore: ["node_modules", "config"],
      },
    }),
  ],
};
```

#### 4.1 authToken

1. **推荐使用：Personal Tokens (个人令牌)**
   这是最通用的选择。在 `sentryWebpackPlugin` 配置中，你只需要提供一个 `authToken` 即可。
   - **权限要求**：创建时至少勾选 `project:write`、`project:releases` 和 `org:read`。

2. **企业级选择：Organization Tokens (组织令牌)**
   通常用于 CI/CD 自动化流水线。与个人账号解耦，即使人员离职也不影响自动化部署。

![image-20260228155519933](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228155519933.png)

![image-20260228154656332](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228154656332.png)

#### 4.2 org 组织名

![image-20260228155436973](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228155436973.png)

#### 4.3 project 项目名

![image-20260228155646709](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228155646709.png)

---

### 5. 验证

在代码中手动抛出一个错误：

```js
const sum = (...args) => {
  return args.reduce((pre, cur) => pre + cur, 0)();
};
console.log(sum(1, 2, 3, 4, 5));
```

执行 `npm run build` 打包并运行代码。片刻后，你将在 Sentry 后台看到这个错误，并且它能精准定位到源代码的具体行数。

![image-20260228160213461](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228160213461.png)

**对比说明：**

- **浏览器控制台**：由于配置了 `devtool: "hidden-source-map"`，报错时无法关联到原始文件。
- **Sentry 后台**：通过上传的 `.map` 文件，可以清晰地看到源码文件及报错行。

![image-20260228160425665](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228160425665.png)
