# 处理 js 资源

有人可能会问，js 资源 Webpack 不能已经处理了吗，为什么我们还要处理呢？

原因是 Webpack 对 js 处理是有限的，只能编译 js 中 ES 模块化语法，不能编译其他语法，导致 js 不能在 IE 等浏览器运行，所以我们希望做一些兼容性处理。

其次开发中，团队对代码格式是有严格要求的，我们不能由肉眼去检测代码格式，需要使用专业的工具来检测。

- 针对 js 兼容性处理，我们使用 Babel 来完成
- 针对代码格式，我们使用 Eslint 来完成

我们先完成 Eslint，检测代码格式无误后，在由 Babel 做代码兼容性处理

## Eslint

可组装的 JavaScript 和 JSX 检查工具。

这句话意思就是：它是用来检测 js 和 jsx 语法的工具，可以配置各项功能

我们使用 Eslint，关键是写 Eslint 配置文件，里面写上各种 rules 规则，将来运行 Eslint 时就会以写的规则对代码进行检查。

> **注意：** ESLint 在 v9.0.0 之后默认采用了全新的 **Flat Config（扁平化配置）**。为了兼顾不同项目，下面将分别介绍旧版配置和新版配置的写法。

### 1. 配置文件

配置文件有以下几种写法：

#### 旧版配置 (Legacy Config)
- `.eslintrc.*`：新建文件，位于项目根目录
  - `.eslintrc`
  - `.eslintrc.js`
  - `.eslintrc.json`
  - 区别在于配置格式不一样
- `package.json` 中 `eslintConfig`：不需要创建文件，在原有文件基础上写

#### 新版配置 (Flat Config)
- `eslint.config.js`：新建文件，位于项目根目录
- `eslint.config.mjs` 或 `eslint.config.cjs`

ESLint 会查找和自动读取它们，所以以上配置文件只需要存在一个即可。

### 2. 具体配置

#### 旧版配置 (.eslintrc.js)

我们以 `.eslintrc.js` 配置文件为例：

```js
module.exports = {
  // 解析选项
  parserOptions: {},
  // 具体检查规则
  rules: {},
  // 继承其他规则
  extends: [],
  // ...
  // 其他规则详见：https://eslint.bootcss.com/docs/user-guide/configuring
};
```

1. parserOptions 解析选项 

   告诉 ESLint 用的解析器：你的代码是哪种“语法/模块/特性”，这样它才能正确解析、不误报。

```js
parserOptions: {
  ecmaVersion: 6, // ES 语法版本
  sourceType: "module", // ES 模块化
  ecmaFeatures: { // ES 其他特性
    jsx: true // 如果是 React 项目，就需要开启 jsx 语法
  }
}
```

2. rules 具体规则

- `"off"` 或 `0` - 关闭规则
- `"warn"` 或 `1` - 开启规则，使用警告级别的错误：`warn` (不会导致程序退出)
- `"error"` 或 `2` - 开启规则，使用错误级别的错误：`error` (当被触发的时候，程序会退出)

```js
rules: {
  semi: "error", // 禁止使用分号
  'array-callback-return': 'warn', // 强制数组方法的回调函数中有 return 语句，否则警告
  'default-case': [
    'warn', // 要求 switch 语句中有 default 分支，否则警告
    { commentPattern: '^no default$' } // 允许在最后注释 no default, 就不会有警告了
  ],
  eqeqeq: [
    'warn', // 强制使用 === 和 !==，否则警告
    'smart' // https://eslint.bootcss.com/docs/rules/eqeqeq#smart 除了少数情况下不会有警告
  ],
}
```

更多规则详见：[规则文档](https://eslint.bootcss.com/docs/rules/)

3. extends 继承

开发中一点点写 rules 规则太费劲了，所以有更好的办法，继承现有的规则。

现有以下较为有名的规则：

- [Eslint 官方的规则](https://eslint.bootcss.com/docs/rules/)：`eslint:recommended`
- [Vue Cli 官方的规则](https://github.com/vuejs/vue-cli/tree/dev/packages/@vue/cli-plugin-eslint)：`plugin:vue/essential`
- [React Cli 官方的规则](https://github.com/facebook/create-react-app/tree/main/packages/eslint-config-react-app)：`react-app`

**旧版配置写法 (.eslintrc.js)：**

```js
// 例如在 Vue 项目中，我们可以这样写旧版配置
module.exports = {
  extends: ["plugin:vue/essential"],
  rules: {
    // 我们的规则会覆盖掉 vue/essential 的规则
    // 所以想要修改规则直接改就是了
    eqeqeq: ["warn", "smart"],
  },
};
```

**新版配置写法 (eslint.config.js)：**

在新版 Flat Config 中，引入 Vue 的官方规则（如 `eslint-plugin-vue`）的方式也有所改变：

```js
// 例如在 Vue 项目中，新版配置可以这样写
import pluginVue from "eslint-plugin-vue";

export default [
  // 继承 vue 的官方规则（对应旧版的 plugin:vue/essential）
  ...pluginVue.configs["flat/essential"],
  {
    rules: {
      // 我们的规则会覆盖掉 vue/essential 的规则
      // 所以想要修改规则直接改就是了
      eqeqeq: ["warn", "smart"],
    },
  },
];
```

#### 新版配置 (eslint.config.js)

在新版 Flat Config 中，配置是一个数组，每个对象代表一组配置。它抛弃了复杂的 `extends` 解析，采用直接引入模块的方式。

```js
import js from "@eslint/js";
import globals from "globals";
import pluginVue from "eslint-plugin-vue";

export default [
	// 1. // 继承 Eslint 官方推荐规则 (应用到所有受支持的文件，通常包含 .js, .mjs, .cjs)
	js.configs.recommended,

	// 2. Vue 推荐配置 (应用到 .vue 文件及相关解析设置)
	...pluginVue.configs["flat/essential"],

	// 3. 自定义配置：针对 .vue 文件的覆盖
	{
		files: ["**/*.vue"],
		rules: {
			"vue/multi-word-component-names": "off",
			"vue/no-unused-vars": "warn",
		},
	},

	// 4. 自定义配置：针对 .js 文件的特定设置
	{
		files: ["**/*.js"], //  "**/*.jsx"
		// 语言选项
		languageOptions: {
			ecmaVersion: "latest",
			sourceType: "module",
			globals: {
				...globals.node, // 启用 node 全局变量
				...globals.browser, // 启用浏览器全局变量
			  },
			parserOptions: {
				ecmaFeatures: {
					// jsx: true, // 只有这里显式开启，才能解析 JSX
				},
			},
		},
		rules: {
			// js.configs.recommended 已经包含了一些规则，这里可以覆盖或添加
			"no-unused-vars": "error",
			"no-undef": "error",
		},
	},
];
```

### 3. 在 Webpack 中使用

1. 下载包

```:no-line-numbers
npm i eslint-webpack-plugin eslint -D
```
*(注：如果使用新版 Flat Config，建议安装 `eslint@9` 或更高版本，以及 `globals` 包：`npm i globals -D`)*

2. 定义 Eslint 配置文件

可以根据项目需求选择**旧版**或**新版**配置：

- **旧版配置 (.eslintrc.js)**

```js
module.exports = {
  // 继承 Eslint 规则
  extends: ["eslint:recommended"],
  env: {
    node: true, // 启用node中全局变量
    browser: true, // 启用浏览器中全局变量
  },
  parserOptions: {
    ecmaVersion: 6,
    sourceType: "module",
  },
  rules: {
    "no-var": 2, // 不能使用 var 定义变量
  },
};
```

- **新版配置 (eslint.config.js)**

```js
import js from "@eslint/js";
import globals from "globals";

export default [
  js.configs.recommended,
  {
    languageOptions: {
      globals: {
        ...globals.node,
        ...globals.browser,
      },
      ecmaVersion: "latest",
      sourceType: "module",
    },
    rules: {
      "no-var": "error",
    },
  },
];
```

3. 修改 js 文件代码

- main.js

```js{11-14}
import count from "./js/count";
import sum from "./js/sum";
// 引入资源，Webpack才会对其打包
import "./css/iconfont.css";
import "./css/index.css";
import "./less/index.less";
import "./sass/index.sass";
import "./sass/index.scss";
import "./styl/index.styl";

var result1 = count(2, 1);
console.log(result1);
var result2 = sum(1, 2, 3, 4);
console.log(result2);
```

4. 配置

- webpack.config.js

```js{2,58-62}
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true, // 自动将上次打包目录资源清空
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.s[ac]ss$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
      {
        test: /\.styl$/,
        use: ["style-loader", "css-loader", "stylus-loader"],
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "src"),
      // 如果使用 Flat Config 且 eslint 版本较新，插件会自动识别 eslint.config.js
    }),
  ],
  mode: "development",
};
```

5. 运行指令

```:no-line-numbers
npx webpack
```

在控制台查看 Eslint 检查效果

### 4. VSCode Eslint 插件

打开 VSCode，下载 Eslint 插件，即可不用编译就能看到错误，可以提前解决。

但是此时就会对项目所有文件默认进行 Eslint 检查了，我们 dist 目录下的打包后文件就会报错。但是我们只需要检查 src 下面的文件，不需要检查 dist 下面的文件。

所以需要配置 Eslint 忽略文件：

- **旧版配置**：在项目根目录新建 `.eslintignore` 文件
  ```
  # 忽略dist目录下所有文件
  dist
  ```

- **新版配置 (Flat Config)**：直接在 `eslint.config.js` 中添加 `ignores` 属性
  ```js
  export default [
    {
      ignores: ["dist/**"] // 忽略 dist 目录下所有文件
    },
    // ... 其他配置
  ];
  ```

## Babel

JavaScript 编译器。

主要用于将 ES6 语法编写的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中

### 1. 配置文件

配置文件由很多种写法：

- `babel.config.*`：新建文件，位于项目根目录
  - `babel.config.js`（有动态逻辑就使用这个）
  - `babel.config.json`（官方推荐使用）
- `.babelrc.*`：新建文件，位于项目根目录
  - `.babelrc`
  - `.babelrc.js`
  - `.babelrc.json`
- `package.json` 中 `babel`：不需要创建文件，在原有文件基础上写

Babel 会查找和自动读取它们，所以以上配置文件只需要存在一个即可

### 2. 具体配置

注意注意注意：JSON 格式**不支持注释**，这会导致 Babel 解析配置文件时报错或**直接忽略整个配置**

我们以 `babel.config.json` 配置文件为例：

```js
{
  // 预设
  “presets”: [],
};
```

**presets 预设**

简单理解：就是一组 Babel 插件, 扩展 Babel 功能

- `@babel/preset-env`: 一个智能预设，允许您使用最新的 JavaScript。
- `@babel/preset-react`：一个用来编译 React jsx 语法的预设
- `@babel/preset-typescript`：一个用来编译 TypeScript 语法的预设

### 3. 在 Webpack 中使用

**1、下载包**

```:no-line-numbers
npm i babel-loader @babel/core @babel/preset-env -D
```

**2、定义 Babel 配置文件**

**`babel.config.json`**

```js
{
  “presets”: ["@babel/preset-env",],
};
```

**3、修改 js 文件代码**

**`main.js`**

```js
import count from "./js/count";
import sum from "./js/sum";
// 引入资源，Webpack才会对其打包
import "./css/iconfont.css";
import "./css/index.css";
import "./less/index.less";
import "./sass/index.sass";
import "./sass/index.scss";
import "./styl/index.styl";

const result1 = count(2, 1);
console.log(result1);
const result2 = sum(1, 2, 3, 4);
console.log(result2);
```

**4、配置**

**`webpack.config.js`**

```js{55-59}
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true, // 自动将上次打包目录资源清空
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.s[ac]ss$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
      {
        test: /\.styl$/,
        use: ["style-loader", "css-loader", "stylus-loader"],
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "src"),
    }),
  ],
  mode: "development",
};
```

**<font color='cornflowerblue'>5、运行指令</font>**

```:no-line-numbers
npx webpack
```

打开打包后的 `dist/static/js/main.js` 文件查看，会发现箭头函数等 ES6 语法已经转换了

### 4. babel-loader @babel/core @babel/preset-env 解析

这三个包是前端开发中处理 JavaScript 新语法的“黄金组合”，它们配合工作，让我们可以放心地使用最新的 JS 语法，而不用担心浏览器兼容性问题。它们的具体含义和分工如下：

**1. babel-loader（搬运工/桥梁）**
- **含义**：它是 Webpack 和 Babel 之间的一座桥梁。
- **作用**：Webpack 默认只能看懂基础的 JS，遇到高级的新语法它可能就不认识了。`babel-loader` 告诉 Webpack：“遇到 JS 文件时，先交给我，我把它送到 Babel 那里翻译一下，翻译好了再还给你去打包。”
- **大白话**：它就是一个快递员，负责把 Webpack 里的 JS 代码打包盒，送到 Babel 工厂去加工。

**2. @babel/core（核心大脑）**
- **含义**：Babel 的核心库，也就是刚才说的“Babel 工厂”的总部。
- **作用**：它包含了 Babel 编译代码的核心 API。它知道如何解析（Parse）JS 代码变成抽象语法树（AST），如何遍历（Traverse）这棵树，以及如何根据规则把树重新生成（Generate）为新的 JS 代码。
- **注意**：`@babel/core` 本身不包含具体的翻译规则。它只是一个引擎，提供能力，但不知道具体要把什么语法转换成什么样（比如它不知道要把箭头函数转成普通函数，这需要别人告诉它）。

**3. @babel/preset-env（规则字典/翻译官）**
- **含义**：这是一个预设（preset），也就是一组插件（翻译规则）的集合包。
- **作用**：前面说了核心大脑不知道具体的翻译规则，`@babel/preset-env` 就是这本厚厚的“翻译字典”。它包含了将最新 JavaScript 语法（如 ES6 的箭头函数、const/let、类等）转换成旧版本浏览器（如 IE、老版 Chrome）能听懂的 ES5 语法的所有必要规则（也就是各种 Babel plugins）。
- **强大之处**：它不仅包含规则，还很“智能”。你可以配置你想要支持的浏览器环境（比如 "目标是 Chrome 80 以上"），它会自动判断哪些语法需要转换，哪些不需要，从而避免过度编译，减小打包后的文件体积。

**总结：它们是如何协同工作的？**

当你写了一段包含 ES6 箭头函数的代码，并运行 Webpack 打包时：
1. Webpack 找到了你的 JS 文件，发现配置了 `babel-loader`。
2. `babel-loader` 接手代码，把它丢给 `@babel/core`（引擎）。
3. `@babel/core` 准备开始翻译，但它问：“我该怎么翻？”
4. 这时，`@babel/preset-env`（字典）站出来说：“查我的字典！把箭头函数转成 function 表达式！”
5. 核心大脑完成翻译，把旧语法（ES5）的代码交还给 `babel-loader`。
6. `babel-loader` 把翻译好的代码还给 Webpack。
7. Webpack 开开心心地把它打包进最终的输出文件中。浏览器完美运行！

这也就是为什么我们在配置 Babel 时，这三个包通常是缺一不可的。
