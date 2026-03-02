# 前置知识

## 一、JavaScript 模块化规范详解

### 1. CommonJS (CJS)
**使用场景**：Node.js 服务端

**特点**：
- 同步加载模块
- 运行时加载（值的拷贝）
- 每个文件是一个模块

**语法**：
```javascript
// 导出
module.exports = { add: (a, b) => a + b };
exports.count = 10;

// 导入
const math = require('./math');
const { add } = require('./math');
```

### 2. AMD (Asynchronous Module Definition)
**使用场景**：浏览器端

**特点**：
- 异步加载模块
- 依赖前置，提前执行
- 代表库：RequireJS

**语法**：
```javascript
// 定义模块
define(['jquery', 'underscore'], function($, _) {
  return {
    method: function() {}
  };
});

// 使用模块
require(['module1', 'module2'], function(m1, m2) {
  m1.method();
});
```

### 3. UMD (Universal Module Definition)
**使用场景**：通用环境（浏览器 + Node.js）

**特点**：
- 兼容 CommonJS 和 AMD
- 可以在多种环境下运行
- 通常用于第三方库

**语法**：
```javascript
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define([], factory);
  } else if (typeof module === 'object' && module.exports) {
    // CommonJS
    module.exports = factory();
  } else {
    // 浏览器全局变量
    root.myModule = factory();
  }
}(this, function () {
  return { method: function() {} };
}));
```

### 4. ES6 Modules (ESM) ⭐ 推荐
**使用场景**：现代浏览器和 Node.js

**特点**：
- JavaScript 官方标准
- 静态导入，编译时加载
- 支持 Tree Shaking（摇树优化）
- 值的引用（而非拷贝）

**语法**：
```javascript
// 导出
export const name = 'value';
export function add(a, b) { return a + b; }
export default function() {}

// 导入
import { name, add } from './module.js';
import defaultExport from './module.js';
import * as module from './module.js';
```

### 5. CMD (Common Module Definition)
**使用场景**：浏览器端（已淘汰）

**特点**：
- SeaJS 推广的规范
- 依赖就近，延迟执行
- 现在基本不再使用

---

## 二、ES6 Modules vs CommonJS 核心区别详解

### 2.1 值的引用 vs 值的拷贝

这是 **ES6 Modules** 和 **CommonJS** 的一个关键区别。

#### 核心区别

| 模块系统 | 导出机制 | 特点 |
|---------|---------|------|
| **CommonJS** | 值的拷贝 📸 | 导出时复制一份值，互不影响 |
| **ES6 Modules** | 值的引用（绑定）📹 | 导出的是引用，指向同一个值 |

#### CommonJS - 值的拷贝（拍照片）

```javascript
// counter.js (CommonJS)
let count = 0;

function increment() {
  count++;
}

function getCount() {
  return count;
}

module.exports = {
  count,      // 导出的是 count 的拷贝
  increment,
  getCount
};
```

```javascript
// main.js
const counter = require('./counter.js');

console.log(counter.count);      // 0
counter.increment();
console.log(counter.count);      // 还是 0 ❌（因为是拷贝，不会更新）
console.log(counter.getCount()); // 1 ✅（通过函数访问到最新值）
```

**说明**：
- `counter.count` 是导出时的**拷贝值**
- 内部 `count` 变化了，但导出的拷贝**不会更新**
- 就像复印了一份文件，原件改了，复印件不会变

#### ES6 Modules - 值的引用（视频直播）

```javascript
// counter.js (ES6)
export let count = 0;

export function increment() {
  count++;
}

export function getCount() {
  return count;
}
```

```javascript
// main.js
import { count, increment, getCount } from './counter.js';

console.log(count);      // 0
increment();
console.log(count);      // 1 ✅（实时更新！）
console.log(getCount()); // 1 ✅
```

**说明**：
- `count` 是一个**实时绑定**（引用）
- 内部 `count` 变化，外部**立即看到变化**
- 就像一个**指针**，始终指向同一个值

#### 形象比喻

**CommonJS - 拍照片 📸**
```
原始文件（counter.js）：
┌─────────┐
│ count:0 │  → 导出时 → ┌─────────┐ 照片（main.js）
│ ↓       │             │ count:0 │
│ count:1 │             │         │
└─────────┘             └─────────┘
                        照片不会自动更新！
```

**ES6 Modules - 视频直播 📹**
```
原始文件（counter.js）：
┌─────────┐
│ count:0 │  ←═══════ 实时连接 ═══════┐
│ ↓       │                         │
│ count:1 │  ←═══════ 同步更新 ═════════┤ main.js
└─────────┘                         │
                                    看到实时值！
```

#### 为什么这很重要？

**1. 状态管理更直观**
```javascript
// store.js (ES6)
export let state = { count: 0 };

export function updateState(newState) {
  state = newState;
}

// app.js
import { state } from './store.js';
console.log(state);  // ✅ 始终是最新的 state
```

**2. 避免常见陷阱**
```javascript
// ❌ CommonJS 的常见错误
const config = require('./config');
config.setDebug(true);
console.log(config.isDebug);  // 还是 false ❌（拷贝不更新）

// ✅ ES6 的正确行为
import { isDebug } from './config.js';
setDebug(true);
console.log(isDebug);  // true ✅（引用自动更新）
```

**3. 注意：ES6 模块导入的值是只读的**
```javascript
// counter.js
export let count = 0;

// main.js
import { count } from './counter.js';
count = 10;  // ❌ TypeError: Assignment to constant variable

// 正确做法：通过导出的函数修改
export function setCount(value) {
  count = value;  // ✅ 在模块内部可以修改
}
```

#### 总结对比

| 特性 | CommonJS | ES6 Modules |
|------|----------|-------------|
| **导出机制** | 值的拷贝 📸 | 值的引用 📹 |
| **更新机制** | 不会自动更新 | 实时同步 |
| **外部修改** | 可以修改导入的对象 | 只读，不能修改 |
| **适合场景** | 服务端同步加载 | 现代应用，静态分析 |

---

### 2.2 Tree Shaking（摇树优化）详解

#### 什么是 Tree Shaking？

**Tree Shaking** 是一个术语，通常用于描述**移除 JavaScript 代码中未使用（dead code）的过程**。

**形象比喻**：

- 🌳 **树** = 你的整个代码库
- 🍃 **绿叶** = 实际使用的代码（活代码）
- 🍂 **枯叶** = 未使用的代码（死代码）

**摇树（Shaking）** = 摇动这棵树，让**枯叶（未使用的代码）掉落**，只保留**绿叶（使用的代码）**

#### Tree Shaking 的作用

**优化前（没有 Tree Shaking）**：
```javascript
// math.js - 导出多个函数
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }

// main.js - 只使用了 add
import { add } from './math.js';
console.log(add(1, 2));
```

```javascript
// bundle.js - 包含所有函数（浪费）
function add(a, b) { return a + b; }
function subtract(a, b) { return a - b; }  // ❌ 未使用但被打包
function multiply(a, b) { return a * b; }  // ❌ 未使用但被打包
function divide(a, b) { return a / b; }    // ❌ 未使用但被打包
console.log(add(1, 2));
```

**优化后（有 Tree Shaking）**：
```javascript
// bundle.js - 只包含使用的函数（优化）
function add(a, b) { return a + b; }  // ✅ 使用了，保留
console.log(add(1, 2));
```

**效果**：
- 📦 减小打包体积（可能减少 50%+ 的代码）
- ⚡ 加快加载速度
- 🚀 提升性能

#### Tree Shaking 的工作原理

**1. 依赖 ES6 Modules（ESM）**

Tree Shaking **只对 ES6 模块（import/export）有效**，对 CommonJS（require/module.exports）**无效**。

**为什么？**

| 特性 | ES6 Modules | CommonJS |
|------|------------|----------|
| **加载时机** | 编译时（静态） | 运行时（动态） |
| **结构** | 静态结构，可分析 | 动态结构，不可预测 |
| **Tree Shaking** | ✅ 支持 | ❌ 不支持 |

**示例对比**：
```javascript
// ✅ ES6 - 静态导入，可以 Tree Shaking
import { add } from './math.js';

// ❌ CommonJS - 动态导入，无法 Tree Shaking
const math = require('./math.js');
const add = math.add;

// ❌ 动态导入，无法静态分析
const moduleName = './math.js';
const math = require(moduleName);
```

**2. 静态分析**

Webpack 在**编译阶段**会：
1. 分析代码的导入导出关系
2. 标记哪些导出被使用了
3. 删除未使用的导出

```javascript
// utils.js
export function usedFunction() { }    // ✅ 被使用
export function unusedFunction() { }  // ❌ 未使用

// main.js
import { usedFunction } from './utils.js';
usedFunction();

// Webpack 分析：
// - usedFunction 被导入并使用 → 保留
// - unusedFunction 未被导入 → 删除
```

#### 如何启用 Tree Shaking

**在 webpack 5 中（自动启用）**

```javascript
// webpack.config.js
export default {
  mode: 'production',  // 生产模式自动启用 Tree Shaking
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  }
};
```

**关键点**：
- ✅ `mode: 'production'` - 自动启用
- ✅ 使用 ES6 模块（import/export）
- ✅ 确保代码无副作用

**package.json 配置（可选）**

```json
{
  "name": "my-project",
  "sideEffects": false  // 告诉 webpack 所有文件都没有副作用（需谨慎：确保项目中无 CSS 导入等副作用代码）
}
```

或者指定有副作用的文件：
```json
{
  "name": "my-project",
  "sideEffects": [
    "*.css",              // CSS 文件有副作用
    "*.scss",
    "./src/polyfill.js"   // 某些特定文件有副作用
  ]
}
```

#### sideEffects 配置的影响对比

**核心区别**：

| 配置 | Tree Shaking 粒度 | 删除范围 |
|------|-----------------|---------|
| **未设置 sideEffects: false** | 成员级别 | 只删除文件内未使用的导出 |
| **设置 sideEffects: false** | 文件级别 + 成员级别 | 可删除整个未使用的文件 |

**示例对比**：

```javascript
// utils.js
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }
console.log('utils.js loaded!');  // 副作用代码

// main.js
import { add } from './utils.js';
console.log(add(1, 2));
```

**未设置 sideEffects: false 时**：
```javascript
// 打包结果
console.log('utils.js loaded!')  // ✅ 保留（副作用代码）
function add(a, b) { return a + b }  // ✅ 保留（被使用）
// subtract 和 multiply 被删除 ❌（未使用的导出）
console.log(add(1, 2))
```
✅ 删除未使用的导出成员  
✅ 保留副作用代码  
✅ 保留文件本身

**设置 sideEffects: false 时**：
```javascript
// colors.js（完全未被导入）
export const RED = '#ff0000';
export const BLUE = '#0000ff';

// main.js
import { add } from './utils.js';
// colors.js 完全未使用
```

打包结果：
- `utils.js` - 保留使用的部分
- `colors.js` - ✅ 整个文件被删除（因为确认无副作用）

**可视化对比**：

```
未设置 sideEffects: false：
文件级别:    [utils.js 文件] ✅ 保留
              ├─ add()      ✅ 保留（被使用）
              ├─ subtract() ❌ 删除（未使用）
              └─ multiply() ❌ 删除（未使用）
副作用代码:    console.log() ✅ 保留

设置 sideEffects: false：
文件级别:    [colors.js 文件] ❌ 整个文件删除
              └─ 所有导出都未使用时，直接删除整个文件
```

**总结**：
- 未设置时 = 保守策略，保留文件和副作用，只删除未使用的导出
- 设置后 = 激进优化，可以删除整个未使用的文件，体积更小

#### Tree Shaking 实际效果

**示例项目**：
```javascript
// lodash-es (支持 Tree Shaking)
import { debounce } from 'lodash-es';

// 打包结果：只包含 debounce 相关代码（~5KB）
// 而不是整个 lodash 库（~70KB）
// 节省了 92% 的体积！
```

```javascript
// lodash (CommonJS，不支持 Tree Shaking)
const { debounce } = require('lodash');

// 打包结果：包含整个 lodash 库（~70KB）❌
```

#### Tree Shaking 的限制

**1. 副作用代码**
```javascript
// math.js
export function add(a, b) { return a + b; }

// 这行代码有副作用（修改全局对象）
window.globalVar = 'something';

// webpack 不敢删除整个文件，即使 add 没被用到
```

**2. 动态导入**
```javascript
// ❌ 无法 Tree Shaking
const moduleName = condition ? './a.js' : './b.js';
import(moduleName);
```

**3. CommonJS 混用**
```javascript
// ❌ 无法 Tree Shaking
const utils = require('./utils.js');
export { utils };
```

#### 最佳实践

✅ **推荐做法**：
- 使用 ES6 模块（import/export）
- 生产模式构建（mode: 'production'）
- 避免副作用代码
- 使用支持 Tree Shaking 的库（如 lodash-es）

❌ **避免**：
- 混用 CommonJS 和 ES6 模块
- 在模块顶层执行有副作用的代码
- 使用默认导出整个对象

**示例**：
```javascript
// ❌ 不利于 Tree Shaking
export default {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};

// ✅ 有利于 Tree Shaking
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
```

#### 常见误区 ⚠️

**误区：需要设置 `"type": "module"` 才能使用 Tree Shaking**

❌ **错误认知**：
```json
{
  "type": "module"  // 以为这个影响 Tree Shaking
}
```

✅ **正确理解**：

| 配置项 | 影响范围 | 是否影响 Tree Shaking |
|--------|---------|---------------------|
| `"type": "module"` | Node.js 如何解析文件 | ❌ **不影响** |
| 源代码用 `import/export` | webpack 的静态分析 | ✅ **直接影响** |
| `mode: "production"` | webpack 优化开关 | ✅ **直接影响** |

**Tree Shaking 的真正条件**：

```javascript
// ✅ 源代码用 ES6 模块 - 这个才是关键！
import { add } from './math.js';
```

```javascript
// webpack.config.js
module.exports = {
  mode: "production",  // ✅ production 模式
};
```

```json
// package.json - 不设置 type 也完全可以！
{
  "name": "my-project",
  "version": "1.0.0"
  // 不需要 "type": "module"
}
```

**实测验证**（未设置 type）：

```javascript
// math.js - 定义 5 个函数
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }  // 未使用
export function divide(a, b) { return a / b; }    // 未使用
export function unused() { /* 大量代码 */ }       // 未使用

// main.js - 只导入 2 个
import { add, subtract } from './math.js';

// 打包结果（production 模式）
// ✅ 只包含 add 和 subtract
// ✅ multiply、divide、unused 全部被删除
// ✅ Tree Shaking 完全生效！
```

**为什么不设置 type 反而更好？**

| 配置 | 源代码 import | Tree Shaking | 灵活性 |
|------|--------------|--------------|--------|
| **不设置 type** ⭐ | `"./math"` 简洁 ✅ | ✅ 支持 | 最高 |
| **"type": "module"** | `"./math.js"` 必须写 ⚠️ | ✅ 支持 | 受限 |

**结论**：
- Tree Shaking 看的是**源代码的 import/export**
- 与 package.json 的 `"type"` 配置**无关**
- webpack 会自动处理模块转换
- **不设置 type** 是最灵活的选择

---

## 三、package.json 中的 "type" 配置详解

### 配置选项

| type 配置值 | 作用 | .js 文件处理方式 | .mjs 文件 | .cjs 文件 |
|------------|------|----------------|----------|----------|
| **不设置（默认）** | 默认 CommonJS | CommonJS | ESM | CommonJS |
| **"commonjs"** | 明确指定 CommonJS | CommonJS（强制） | ESM | CommonJS |
| **"module"** | 启用 ESM | ESM（严格模式） | ESM | CommonJS |

### 1. 不设置 type（默认行为）⭐⭐⭐⭐⭐ 最推荐

```json
{
  "name": "webpack5_code",
  "dependencies": {
    "webpack": "^5.105.2",
    "webpack-cli": "^6.0.1"
  }
}
```

**行为**：
- `.js` 文件默认为 **CommonJS** 模块
- `.mjs` 文件为 **ES6** 模块
- `.cjs` 文件为 **CommonJS** 模块

**webpack.config.js 实测结果（webpack-cli 6.x）**：
```javascript
// ✅ CommonJS 语法 - 可以
module.exports = {
  mode: 'development'
};

// ✅ ESM 语法 - 也可以！（webpack-cli 6.x 智能加载）
export default {
  mode: 'development'
};
```

**源代码**：

```javascript
// src/main.js
import count from "./js/count";  // ✅ 不需要 .js
const sum = require("./js/sum"); // ✅ 也可以用 require
```

**优点**：
- ✅ 最灵活
- ✅ import 不需要扩展名
- ✅ webpack-cli 6.x 智能支持两种语法
- ✅ 兼容性最好

### 2. 设置 "type": "commonjs"（显式声明）

```json
{
  "name": "webpack5_code",
  "type": "commonjs",
  "dependencies": {
    "webpack": "^5.105.2",
    "webpack-cli": "^6.0.1"
  }
}
```

**行为**：
- `.js` 文件**强制**为 **CommonJS** 模块
- Node.js 严格要求使用 CommonJS 语法
- **效果与不设置 type 不同**（更严格）

**webpack.config.js 实测结果**：
```javascript
// ✅ CommonJS 语法 - 可以
module.exports = {
  mode: 'development'
};

// ❌ ESM 语法 - 不可以！会报错
export default {
  mode: 'development'
};
// 错误: SyntaxError: Unexpected token 'export'
```

**源代码**：
```javascript
// src/main.js
import count from "./js/count";  // ✅ 不需要 .js（webpack 处理）
const sum = require("./js/sum"); // ✅ 也可以
```

**适用场景**：
- 明确表示项目使用 CommonJS
- 需要严格限制只使用 CommonJS
- 防止团队成员在配置文件中误用 ESM

### 3. 设置 "type": "module"（启用 ESM）

```json
{
  "name": "webpack5_code",
  "type": "module",
  "dependencies": {
    "webpack": "^5.105.2",
    "webpack-cli": "^6.0.1"
  }
}
```

**行为**：
- `.js` 文件视为 **ES6 模块**（严格模式）
- `.mjs` 文件为 **ES6** 模块
- `.cjs` 文件为 **CommonJS** 模块

**限制**：
- import 必须写完整扩展名（`.js`）
- 不能使用 CommonJS 语法（`require`）

**webpack.config.js**：
```javascript
// ✅ ESM 语法 - 可以
export default {
  mode: 'development'
};

// ❌ CommonJS 语法 - 不可以
module.exports = {};

// 或者改名为 webpack.config.cjs 使用 CommonJS
```

**源代码**：
```javascript
// src/main.js
// ✅ 必须写完整扩展名
import count from "./js/count.js";  // 必须加 .js
import sum from "./js/sum.js";

// ❌ 不能用 CommonJS
const module = require("./module");  // 会报错
```

**错误示例**：
```
ERROR in ./src/main.js 1:0-31
Module not found: Error: Can't resolve './js/count'
BREAKING CHANGE: The extension in the request is mandatory for it to be fully specified.
Add the extension to the request.
```

**适用场景**：
- 纯 ESM 项目
- Node.js 原生 ESM 应用

---

## 四、webpack-cli 版本影响（实测结果）

### webpack-cli 6.x 智能加载机制 🎉

| package.json 配置 | webpack.config.js 可用语法 | 源代码 import | 源代码 require | 实测结果 |
|------------------|-------------------------|--------------|---------------|---------|
| **不设置 type** ⭐ | **CJS ✅ / ESM ✅** | 不需要 .js ✅ | 可以 ✅ | ✅ 通过 |
| **"type": "commonjs"** | CJS ✅ / ESM ❌ | 不需要 .js ✅ | 可以 ✅ | ⚠️ ESM 报错 |
| **"type": "module"** | CJS ❌ / ESM ✅ | **必须加 .js** ⚠️ | 不可以 ❌ | ⚠️ import 受限 |

### webpack-cli 6.x 的核心机制

webpack-cli 6.x 有一个**智能加载机制**：

```javascript
// webpack-cli 内部逻辑（简化版）
async tryRequireThenImport(configPath) {
  try {
    // 先尝试 CommonJS require
    return require(configPath);
  } catch (err) {
    // 关键：同时处理两种错误！
    if (
      err.code === 'ERR_REQUIRE_ESM' ||  // .mjs 或 "type":"module" 的 .js
      err instanceof SyntaxError           // 不设置 type 时 .js 里写了 export default
    ) {
      // 自动改用 import() 加载
      return await import(configPath);
    }
    throw err;
  }
}
```

**这就是为什么**：

| 情况 | 结果 | 原因 |
|------|------|------|
| **不设置 type** + `export default` | ✅ 成功 | require 抛出 SyntaxError → 被捕获 → 自动改用 import() 成功加载 |
| **"type": "commonjs"** + `export default` | ❌ 失败 | 显式标记为 CJS → require 和 import() 都将 .js 视为 CJS → `export` 语法始终非法，无法挽救 |
| **"type": "module"** + `export default` | ✅ 成功 | require 抛出 ERR_REQUIRE_ESM → 被捕获 → 自动改用 import() 成功加载 |

### webpack-cli 版本对比

| webpack-cli 版本 | 不设置 type | "type": "commonjs" | "type": "module" |
|-----------------|------------|-------------------|-----------------|
| **< 6.x** | 只能用 CJS | 只能用 CJS | ESM 或改名 .cjs |
| **>= 6.x** 🎉 | **CJS 或 ESM 都可以** ⭐ | 只能用 CJS | ESM 或改名 .cjs |

---

## 五、实测验证记录

### 测试环境
- webpack: 5.105.2
- webpack-cli: 6.0.1
- Node.js: v20+

### 测试 1：不设置 type + export default ✅

**配置**：
```json
// package.json
{
  "name": "webpack5_code",
  "dependencies": {
    "webpack": "^5.105.2",
    "webpack-cli": "^6.0.1"
  }
  // 不设置 type
}
```

```javascript
// webpack.config.js
export default {
  mode: "development",
  entry: "./src/main.js"
};
```

**结果**：
```bash
$ npx webpack --mode=production
✅ webpack 5.105.2 compiled successfully in 186 ms
```

### 测试 2：type: "commonjs" + export default ❌

**配置**：
```json
// package.json
{
  "name": "webpack5_code",
  "type": "commonjs",
  "dependencies": { ... }
}
```

```javascript
// webpack.config.js
export default {
  mode: "development"
};
```

**结果**：
```bash
$ npx webpack --mode=production
❌ SyntaxError: Unexpected token 'export'
[webpack-cli] Failed to load webpack.config.js
```

### 测试 3：type: "module" + export default ⚠️

**配置**：
```json
// package.json
{
  "name": "webpack5_code",
  "type": "module",
  "dependencies": { ... }
}
```

```javascript
// webpack.config.js
export default {
  mode: "development",
  entry: "./src/main.js"
};
```

```javascript
// src/main.js
import count from "./js/count";  // ❌ 没有 .js
```

**结果**：
```bash
$ npx webpack --mode=production
❌ Module not found: Can't resolve './js/count'
BREAKING CHANGE: The extension in the request is mandatory
```

---

## 六、最佳实践建议（基于实测）

### 推荐配置 ⭐⭐⭐⭐⭐

```json
// package.json - 不设置 type
{
  "name": "webpack5_code",
  "version": "1.0.0",
  "dependencies": {
    "webpack": "^5.105.2",
    "webpack-cli": "^6.0.1"
  }
}
```

```javascript
// webpack.config.js - 使用 ESM 语法（现代化）
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default {
  mode: 'development',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

> **注意**：ESM 中没有 `__dirname`，需要通过 `import.meta.url` 手动构造。
> 如果不需要自定义输出路径，可以省略 `output.path`（webpack 默认输出到 `dist` 目录）：
> ```javascript
> export default {
>   mode: 'development',
>   entry: './src/main.js',
>   output: { filename: 'bundle.js' }
> };
> ```

```javascript
// src/main.js - 使用 ES6 模块
import count from "./js/count";  // 简洁，不需要 .js ✅
import sum from "./js/sum";

console.log(count(3, 1));
console.log(sum(1, 2, 3, 4));
```

### 为什么这是最佳配置？

1. **不设置 type**：
   - ✅ webpack-cli 6.x 智能加载，配置文件可用 ESM
   - ✅ 源代码 import 不需要写扩展名
   - ✅ 灵活性最高
   - ✅ 兼容性最好

2. **配置文件用 ESM**：
   - ✅ 与源代码风格统一
   - ✅ 更现代化
   - ✅ webpack-cli 6.x 智能支持

3. **源代码用 ESM**：
   - ✅ JavaScript 官方标准
   - ✅ 支持 Tree Shaking
   - ✅ 静态分析
   - ✅ 更好的编辑器支持

---

## 七、配置选择流程图

```
需要配置 package.json 的 type 吗？
    │
    ├─→ 是否使用 webpack-cli 6.x？
    │   │
    │   ├─→ 是 → 推荐不设置 type ⭐⭐⭐⭐⭐
    │   │   └─ 配置文件可用 export default
    │   │   └─ import 不需要 .js
    │   │   └─ 最灵活、最简洁
    │   │
    │   └─→ 否 (< 6.x) → 配置文件必须用 CommonJS
    │       └─ module.exports = {}
    │
    ├─→ 是否需要严格限制只用 CommonJS？
    │   ├─→ 是 → "type": "commonjs"
    │   │   └─ 配置文件只能用 module.exports
    │   │
    │   └─→ 否 → 继续
    │
    └─→ 是否纯 ESM 项目（Node.js 原生）？
        ├─→ 是 → "type": "module"
        │   └─ 注意：import 必须加 .js
        │
        └─→ 否 → 不设置 type ⭐ 最推荐
```

---

## 八、关键要点总结

### 1. type 配置的影响

| 配置 | webpack.config.js | 源代码 import | 推荐度 |
|------|------------------|--------------|-------|
| **不设置** | CJS 或 ESM ✅ | 不需要 .js ✅ | ⭐⭐⭐⭐⭐ |
| **"commonjs"** | 只能 CJS ⚠️ | 不需要 .js ✅ | ⭐⭐ |
| **"module"** | 只能 ESM ✅ | 必须加 .js ⚠️ | ⭐⭐ |

### 2. 文件扩展名规则

| 扩展名 | 始终视为 | 不受 type 影响 |
|-------|---------|--------------|
| `.mjs` | ES Module | ✅ |
| `.cjs` | CommonJS | ✅ |
| `.js` | 取决于 type | ❌ |

### 3. webpack-cli 6.x 的核心特性

✨ **智能加载机制**：
- 不设置 type 时，配置文件可以用 `export default`
- webpack-cli 会自动尝试 require，失败后切换到 import
- 这是 webpack-cli 6.x 的重要新特性！

### 4. 实测验证的结论

✅ **推荐做法**：
- 不设置 type
- webpack.config.js 用 `export default`（ESM 语法）
- 源代码用 `import/export`
- import 不需要写 .js 扩展名

❌ **不推荐**：
- 显式设置 `"type": "commonjs"`（限制灵活性）
- 设置 `"type": "module"`（import 必须加 .js，麻烦）
- 配置文件用 CommonJS（虽然可以，但不现代）

---

## 九、常见问题 FAQ

### Q1: 不设置 type 能使用 Tree Shaking 吗？主流不是 ESM 吗？
- ✅ **完全可以！这是一个常见误区！**
- Tree Shaking 看的是**源代码的 import/export**，与 package.json 的 `"type"` 无关
- 你的代码已经在用 ESM（`import/export`），这才是关键

**正确理解**：
```
ESM 是主流 ✅          ≠         必须设置 "type": "module" ❌
（代码编写风格）                    （package.json 配置）
```

**实测证明**（不设置 type）：
```javascript
// math.js - 定义 5 个函数
export function add() {}        // ← 使用
export function subtract() {}   // ← 使用
export function multiply() {}   // ← 未使用
export function divide() {}     // ← 未使用
export function unused() {}     // ← 未使用

// main.js
import { add, subtract } from './math.js';

// 打包结果（production 模式）
// ✅ 只保留 add 和 subtract
// ✅ 其他 3 个函数全部删除
// ✅ Tree Shaking 完全生效！
```

**为什么不设置 type 更好**：
- ✅ 源代码用 ESM（主流）
- ✅ Tree Shaking 正常工作
- ✅ import 不需要写 .js（简洁）
- ✅ webpack 自动处理转换
- ✅ 最灵活、最方便

**结论**：不设置 type 既符合主流（用 ESM），又最灵活（无限制）！

---

## 十、总结

### 核心结论（实测验证）

1. **webpack-cli 6.x** 支持在**不设置 type** 的情况下使用 ESM 语法（`export default`）
2. **显式设置 "type": "commonjs"** 会禁用此特性，强制使用 CommonJS
3. **Tree Shaking** 与 `"type"` 配置**无关**，只看源代码是否用 `import/export`
4. **最佳实践**：不设置 type + 配置文件用 ESM + 源代码用 ESM

### 推荐配置一览

```json
// package.json ⭐⭐⭐⭐⭐
{
  "name": "webpack5_code",
  "version": "1.0.0",
  "dependencies": {
    "webpack": "^5.105.2",
    "webpack-cli": "^6.0.1"
  }
  // 不设置 type
}
```

```javascript
// webpack.config.js ⭐⭐⭐⭐⭐
export default {
  mode: 'development',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js'
  }
};
```

```javascript
// src/main.js ⭐⭐⭐⭐⭐
import count from "./js/count";  // 不需要 .js
import sum from "./js/sum";

console.log(count(3, 1));
```

这样的配置：
- ✅ 最灵活
- ✅ 最简洁（import 不需要 .js）
- ✅ 最现代化（全部使用 ESM）
- ✅ 兼容性最好
- ✅ Tree Shaking 完全支持
- ✅ 实测验证通过

### 关键认知纠正

**误区**：以为需要设置 `"type": "module"` 才能使用 Tree Shaking 或跟上 ESM 主流

**真相**：
```
源代码用 import/export（ESM 主流）✅
         +
mode: "production"（webpack 优化）✅
         =
    Tree Shaking 完全生效 ✅

与 "type" 配置无关！
```

**这就是 webpack 5 + webpack-cli 6.x 的最佳实践！** 🎯
