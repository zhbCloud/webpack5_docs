# 提升打包构建速度

## HotModuleReplacement

### 为什么

开发时我们修改了其中一个模块代码，Webpack 默认会将所有模块全部重新打包编译，速度很慢。

所以我们需要做到修改某个模块代码，就只有这个模块代码需要重新打包编译，其他模块不变，这样打包速度就能很快。

### 是什么

HotModuleReplacement（HMR/热模块替换）：在程序运行中，替换、添加或删除模块，而无需重新加载整个页面。重新加载整个页面也意味着所有模块重新打包了。
热更新不会导致整个页面刷新加载

![image-20260228173212268](https://picgo-2024.oss-cn-beijing.aliyuncs.com/img/image-20260228173212268.png)

### 怎么用

1. 基本配置

```js
module.exports = {
  // 其他省略
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 开启HMR功能（只能用于开发环境，生产环境不需要了）
  },
};
```

此时 css 样式经过 style-loader 处理，已经具备 HMR 功能了。
但是 js 还不行。

2. JS 配置

```js{17-28}
// main.js
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

// 判断是否支持HMR功能
if (module.hot) {
  module.hot.accept("./js/count.js", function (count) {
    const result1 = count(2, 1);
    console.log(result1);
  });

  module.hot.accept("./js/sum.js", function (sum) {
    const result2 = sum(1, 2, 3, 4);
    console.log(result2);
  });
}
```

上面这样写会很麻烦，所以实际开发我们会使用其他 loader 来解决。

比如：[vue-loader](https://github.com/vuejs/vue-loader), [react-hot-loader](https://github.com/gaearon/react-hot-loader)。

## OneOf

### 为什么

打包时每个文件都会经过所有 loader 处理，虽然因为 `test` 正则原因实际没有处理上，但是都要过一遍。比较慢。

### 是什么

简单来说，它的作用是：**让文件只匹配 `oneOf` 数组中的某一个 Loader，一旦匹配成功，就不再继续向下遍历剩余的 Loader 规则。**

**如果没有 `oneOf`：**

Webpack 在处理每一个模块（文件）时，都会把 `rules` 数组里的所有规则**从头到尾检查一遍**。

- 比如你有一个 `index.css` 文件。
- Webpack 会先检查它是否匹配 CSS 规则（匹配成功，处理）。
- 然后它**还会继续检查**它是否匹配图片规则、字体规则、JS 规则……
- 虽然最终因为后缀名不符不会执行其他 Loader，但这种“遍历检查”的操作在大型项目中会浪费不少构建时间。

**有了 `oneOf` 以后：**

- 当 `index.css` 匹配到第一个 CSS 规则后，Webpack 会直接跳出 `oneOf` 队列。
- 不再去识别后续那些无关的规则，从而**提升打包速度**。

**注意事项 (重要)**

在使用 `oneOf` 时，有以下几点需要注意：

1. **不能有重复匹配**：如果一个文件需要被多个 Loader 规则处理（例如：既需要 `eslint-loader` 检查，又需要 `babel-loader` 转换），那么不能把它们都放在同一个 `oneOf` 里
   - **解决方案**：通常会将 `eslint-loader(废弃)`（或者你现在用的 `ESLintWebpackPlugin(最新)`）放在 `oneOf` 之外，或者使用 `enforce: 'pre'` 确保它先执行。
2. **收益**：在小型项目（如练习 demo）中，`oneOf` 的提速效果不明显；但在拥有成百上千个文件的商业项目中，它是必做的性能优化手段之一。

### 怎么用

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: undefined, // 开发模式没有输出，不需要指定输出目录
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    // clean: true, // 开发模式没有输出，不需要清空输出结果
  },
  module: {
    rules: [
      {
        oneOf: [
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
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
  ],
  // 开发服务器
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 开启HMR功能
  },
  mode: "development",
  devtool: "cheap-module-source-map",
};
```

生产模式也是如此配置。

## Include/Exclude

### 为什么

开发时我们需要使用第三方的库或插件，所有文件都下载到 node_modules 中了。而这些文件是不需要编译可以直接使用的。

所以我们在对 js 文件处理时，要排除 node_modules 下面的文件。

### 是什么

- include

包含，只处理 xxx 文件

- exclude

排除，除了 xxx 文件以外其他文件都处理

### 怎么用

```js{60-61,72}
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: undefined, // 开发模式没有输出，不需要指定输出目录
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    // clean: true, // 开发模式没有输出，不需要清空输出结果
  },
  module: {
    rules: [
      {
        oneOf: [
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
            // exclude: /node_modules/, // 排除node_modules代码不编译
            include: path.resolve(__dirname, "../src"), // 也可以用包含
            loader: "babel-loader",
          },
        ],
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      // exclude: 'node_modules', // 可以省略，因为默认就是排除 node_modules
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
  ],
  // 开发服务器
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 开启HMR功能
  },
  mode: "development",
  devtool: "cheap-module-source-map",
};
```

生产模式也是如此配置。

## Cache

### 为什么

每次打包时 js 文件都要经过 Eslint 检查 和 Babel 编译，速度比较慢。

我们可以缓存之前的 Eslint 检查 和 Babel 编译结果，这样第二次打包时速度就会更快了。

查阅其[eslint-webpack-plugin官方文档](https://github.com/webpack-contrib/eslint-webpack-plugin#cache)

查阅 [babel-loader 官方文档](https://github.com/babel/babel-loader#options)：

### 是什么

对 Eslint 检查 和 Babel 编译结果进行缓存。

### 怎么用

```js{63-66,76-77}
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: undefined, // 开发模式没有输出，不需要指定输出目录
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    // clean: true, // 开发模式没有输出，不需要清空输出结果
  },
  module: {
    rules: [
      {
        oneOf: [
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
            // exclude: /node_modules/, // 排除node_modules代码不编译
            include: path.resolve(__dirname, "../src"), // 也可以用包含
            loader: "babel-loader",
            options: {
              cacheDirectory: true, // 开启babel编译缓存
              cacheCompression: false, // 缓存文件不要压缩
            },
          },
        ],
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      // exclude: 'node_modules', // 可以省略，因为默认就是排除 node_modules
      // cache: true, //  可以省略，因为默认就是开启缓存
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
  ],
  // 开发服务器
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 开启HMR功能
  },
  mode: "development",
  devtool: "cheap-module-source-map",
};
```

## Thead

### 为什么

当项目越来越庞大时，打包速度越来越慢，甚至于需要一个下午才能打包出来代码。这个速度是比较慢的。

我们想要继续提升打包速度，其实就是要提升 js 的打包速度，因为其他文件都比较少。

而对 js 文件处理主要就是 eslint 、babel、Terser(构建后的代码压缩混淆) 三个工具，所以我们要提升它们的运行速度。

我们目前都是使用一个进程eslint 、babel、Terser 一个个执行处理。

我们可以开启多进程同时处理 js 文件，这样速度就比之前的单进程打包更快了。

**<font color='cornflowerblue'>ESLint、Babel、Terser 三者对比</font>**

这三个工具分别负责 JavaScript 工程化的**三个不同阶段**：

🔍 **ESLint —— 代码检查（Lint 阶段）**

- **作用**：静态分析代码，找出语法错误、风格问题、潜在 bug
- **时机**：**构建之前**，对源码进行扫描
- **例子**：发现你用了 `var`、变量未使用、缺少分号等问题
- **特点**：不会修改代码逻辑，只是"挑毛病"

**🔄 Babel —— 代码转译（Transform 阶段）**

- **作用**：将高版本 ES 语法（ES6+）转换为低版本浏览器能理解的 ES5 语法
- **时机**：**构建中**，对源码进行转换
- **例子**：将箭头函数() => {} 转换为普通 `function() {}`，将 `async/await` 转换为 Promise 链
- **特点**：改变代码**语法形式**，但保持逻辑一致

**⚡ Terser —— 代码压缩（Minify 阶段）**

- **作用**：将代码进行**压缩混淆**，减小体积，提升加载速度

- **时机**：**构建后**，对产物进行压缩

- **例子**：

  ```js
  // 压缩前
  function add(a, b) {
    return a + b;
  }

  // 压缩后
  function add(n, d) {
    return n + d;
  }
  // 甚至更极致：
  const add = (n, d) => n + d;
  ```

- **具体做什么:**
  - 删除注释、空白符、换行
  - 缩短变量名（`longVariableName` → `a`）
  - 删除 Dead Code（永远不会执行的代码）
  - Tree Shaking 辅助

- **特点**：只面向**生产环境**，开发环境不用

### 是什么

多进程打包：开启电脑的多个进程同时干一件事，速度更快。

**需要注意：请仅在特别耗时的操作中使用，因为每个进程启动就有大约为 600ms 左右开销。**

### 怎么用

我们启动进程的数量就是我们 CPU 的核数。

1. 如何获取 CPU 的核数，因为每个电脑都不一样。

```js
// nodejs核心模块，直接使用
const os = require("os");
// cpu核数
const threads = os.cpus().length;
```

2. 下载包

查阅 [thread-loader文档](https://www.webpackjs.com/loaders/thread-loader/)：

使用时，需将此 loader 放置在其他 loader 之前。放置在此 loader 之后的 loader 会在一个独立的 worker 池中运行。

```
npm i thread-loader -D
```

3. 使用

```js{1,8,10-11,89-94,114,144-155}
const os = require("os");
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
const { sentryWebpackPlugin } = require("@sentry/webpack-plugin");
const TerserPlugin = require("terser-webpack-plugin");

// cpu核数
const threads = os.cpus().length;

// 获取处理样式的Loaders
const getStyleLoaders = (preProcessor) => {
    return [
        MiniCssExtractPlugin.loader,
        "css-loader",
        {
            loader: "postcss-loader",
            options: {
                postcssOptions: {
                    plugins: [
                        "postcss-preset-env", // 能解决大多数样式兼容性问题
                    ],
                },
            },
        },
        preProcessor,
    ].filter(Boolean);

    // .filter(Boolean) 等同于 .filter((item) => Boolean(item))
};

module.exports = {
    // 入口
    entry: "./src/main.js",
    // 输出
    output: {
        // __dirname nodejs的变量，表示当前文件的目录
        path: path.resolve(__dirname, "../dist"), // 绝对路径
        filename: "js/main.js",
        // ✨ 全局兜底：所有没单独配置 generator 的资源，都放到 media 文件夹
        assetModuleFilename: "images/[hash:10][ext][query]", // 图片的输出路径
        clean: true, // 自动将上次打包目录资源清空
    },
    // 加载器
    module: {
        rules: [
            {
                oneOf: [
                    // loader的配置(执行顺序是 从下往上 或 从右到左)
                    {
                        // 处理css的loader
                        test: /\.css$/,
                        use: getStyleLoaders(),
                    },
                    {
                        test: /\.s[ac]ss$/,
                        use: getStyleLoaders("sass-loader"),
                    },
                    {
                        // 处理图片的loader
                        test: /\.(png|jpe?g|gif|webp)$/,
                        type: "asset",
                        parser: {
                            // 解析图片的配置
                            dataUrlCondition: {
                                // 当图片小于10kb时，会自动转换为base64格式
                                maxSize: 10 * 1024, // 10kb
                            },
                        },
                        generator: {
                            // ✨ 优先级更高：覆盖兜底配置，放到 images 目录
                            filename: "images/[hash:10][ext][query]",
                        },
                    },
                    {
                        // 处理字体图标的loader
                        test: /\.(woff2?|eot|ttf|otf)$/,
                        type: "asset/resource",
                        generator: {
                            filename: "fonts/[hash:10][ext][query]",
                        },
                    },
                    {
                        test: /\.m?js$/,
                        exclude: /node_modules/, // 编译的时候可以排除node_modules中的js文件
                        use: [
                            {
                                loader: "thread-loader", // 开启多进程
                                options: {
                                    workers: threads, // 数量
                                },
                            },
                            {
                                loader: "babel-loader",
                                options: {
                                    //  presets: ['@babel/preset-env'], // 不推荐写这里 一般babel的预设都放在babel.config.xxx文件中
                                    cacheDirectory: true, // 开启babel缓存
                                    cacheCompression: false, // 关闭缓存压缩
                                },
                            },
                        ],
                    },
                ],
            },
        ],
    },
    // 插件
    plugins: [
        // 插件的配置
        new ESLintWebpackPlugin({
            context: path.resolve(__dirname, "../src"),
            threads, // 开启多进程
            // exclude: 'node_modules', // 可以省略，因为默认就是排除 node_modules
            // cache: true, //  可以省略，因为默认就是开启缓存
        }),
        // 自动引入打包后的js文件
        // 这种写法打包出来 里面只有引入一个js文件作为入口， 你public中index.html中写的 html不会打包进去
        // new HtmlWebpackPlugin()

        new HtmlWebpackPlugin({
            // 以 public/index.html 为模板创建文件
            // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
            template: path.resolve(__dirname, "../public/index.html"),
        }),
        // 本插件会将 CSS 提取到单独的文件中 使用link异步引入
        new MiniCssExtractPlugin({
            filename: "css/index.css",
        }),
        // css压缩
        // new CssMinimizerPlugin(),
        sentryWebpackPlugin({
            authToken: "YOUR_AUTH_TOKEN",
            org: "ad67cbf5e064",
            project: "javascript",
            sourcemaps: {
                assets: ["./dist/**"],
                ignore: ["node_modules", "config"],
            },
        }),
    ],
    optimization: {
        // Webpack 的构建优化控制中心，主要用于生产环境打包时的代码压缩、分割和模块合并等优化策略
        minimize: true, // // 开启压缩（生产模式默认就是 true）
        minimizer: [ //  压缩器列表 一旦自定义了 minimizer，Webpack 默认的 JS 压缩器（TerserPlugin）就会被覆盖！必须要写上TerserPlugin  否则不做压缩
            // css压缩
            new CssMinimizerPlugin(),
            //  JS 压缩（多进程）
            new TerserPlugin({
                parallel: threads, // 开启多进程
            }),
        ],
    },
    // 模式
    mode: "production",
    devtool: "hidden-source-map",
};

```

**我们目前打包的内容都很少，所以因为启动进程开销原因，使用多进程打包实际上会显著的让我们打包时间变得很长。**

### `optimization` 配置详解

`optimization` 是 Webpack 的**构建优化控制中心**，主要用于生产环境打包时的代码压缩、分割和模块合并等优化策略。

**核心子配置项**

1、`minimize` — 是否启用压缩

```js
optimization: {
  minimize: true, // 生产模式默认为 true，开发模式默认为 false
}
```

控制是否启用 `minimizer` 中配置的压缩器。**生产模式下默认开启**，无需手动设置。

2、`minimizer` — 自定义压缩器

```js
const TerserPlugin = require('terser-webpack-plugin')
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')

optimization: {
  // 一旦自定义了 minimizer，Webpack 默认的 JS 压缩器（TerserPlugin）就会被覆盖！必须要写上TerserPlugin  否则不做压缩
  minimizer: [
    // 压缩 JS（生产模式默认使用 TerserPlugin，这里可以自定义配置）
    new TerserPlugin({
      parallel: threads, // 你已经引入了 threads，可以开启多进程压缩
    }),
    // 压缩 CSS（注意：你目前是把它放在 plugins 里，也可以移到这里）
    new CssMinimizerPlugin(),
  ],
}
```

**重点**：把 `CssMinimizerPlugin` 放在 `plugins` 里也能工作，但官方**推荐放在 `minimizer`** 里，语义更清晰，且只在 `minimize: true` 时才执行。

3、`splitChunks` — 代码分割（最常用）

```js
optimization: {
  splitChunks: {
    chunks: 'all', // 对所有模块（同步 + 异步）进行分割
  }
}
```

**作用**：将多个入口或动态 `import()` 共享的模块提取成独立 chunk，避免重复打包。

| `chunks` 值 | 含义                            |
| :---------- | :------------------------------ |
| `'async'`   | 只分割异步加载的模块（默认）    |
| `'initial'` | 只分割同步加载的模块            |
| `'all'`     | 同步 + 异步全部分割（**推荐**） |

**效果举例**：如果你的项目在多个页面都用了 `lodash`，设置 `chunks: 'all'` 后，`lodash` 会被单独打包成一个 chunk，所有页面共享，而不是每个页面各打一份。

4、`runtimeChunk` — 提取运行时代码

```js
optimization: {
  runtimeChunk: {
    name: (entrypoint) => `runtime~${entrypoint.name}`,
  }
}
```

**作用**：将 Webpack 的运行时（runtime）代码单独提取出来。

**为什么需要它？**

- Webpack 每次打包后，`main.js` 里会内嵌一段 runtime 代码，其中包含模块间的依赖关系映射（contenthash）。
- 如果你的业务代码没改变，但某个被引用的模块 chunk 文件名变了，runtime 也会跟着变，导致 `main.js` 的 hash 也变了，**浏览器缓存失效**。
- 提取出 `runtime` 后，`main.js` 的 hash 就能保持稳定，充分利用缓存。
