# 处理图片资源

过去在 Webpack4 时，我们处理图片资源通过 `file-loader` 和 `url-loader` 进行处理

现在 Webpack5 已经将两个 Loader 功能内置到 Webpack 里了，我们只需要简单配置即可处理图片资源

在 webpack 5 之前，通常使用：

- [`raw-loader`](https://v4.webpack.js.org/loaders/raw-loader/) 将文件导入为字符串
- [`url-loader`](https://v4.webpack.js.org/loaders/url-loader/) 将文件作为 data URI 内联到 bundle 中
- [`file-loader`](https://v4.webpack.js.org/loaders/file-loader/) 将文件发送到输出目录

资源模块类型(asset module type)，通过添加 4 种新的模块类型，来替换所有这些 loader：

- `asset/resource` 

  - 作用：将资源文件原封不动地打包输出到一个单独的文件中，并导出该文件的 URL 路径。
  - 替代：Webpack 4 中的 `file-loader`。
  - 适用场景：较大的图片文件、字体文件（woff2, eot 等）、音视频文件等。

- `asset/inline` 

  - 作用：将资源文件转换为 Data URI（通常是 Base64 编码的字符串），并直接内嵌到打包后的 JS（或 CSS）文件中，不会生成独立的资源文件。

  - 替代：Webpack 4 中的` url-loader`（且不配置 limit 限制大小的情况）。

  - 适用场景：极小的图标（如 1kb、2kb 的小 png）。好处是可以减少网页加载时的 HTTP 请求数量；坏处是如果把大图片转成 Base64，会让你的 JS 文件体积急剧增大。

- `asset/source` 

  - 作用：导出资源的源代码（把文件内容作为字符串原样导出）。

  - 替代：Webpack 4 中的 `raw-loader`。

  - 适用场景：需要读取文本内容时，比如导入 .txt 文本文件、.md Markdown 文件，或者你想把 SVG 图标直接当成 HTML 字符串插入到页面中时。

- `asset` (最常用)

  - 作用：这是一个智能类型。它会在 asset/resource（输出文件）和 asset/inline（转 Base64 内嵌）之间自动做出选择。

  - 替代：Webpack 4 中配置了文件体积限制（limit）的 `url-loader`。

  - 适用场景：常规的图片处理。

  - 默认规则：如果文件大小小于 8kb，Webpack 会自动将其当作 asset/inline 处理（转 Base64）；如果文件大小大于 8kb，则将其当作 asset/resource 处理（输出为单独文件）。你可以通过配置 

    ```js
    {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
            dataUrlCondition: {
            	maxSize: 10 * 1024 // 小于10kb的图片会被base64处理
            }
        }
    },
    ```

    

## 1. 配置

```js{29-32}
const path = require("path");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
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
      },
    ],
  },
  plugins: [],
  mode: "development",
};
```

## 2. 添加图片资源

- src/images/1.jpeg
- src/images/2.png
- src/images/3.gif

## 3. 使用图片资源

- src/less/index.less

```css
.box2 {
  width: 100px;
  height: 100px;
  background-image: url("../images/1.jpeg");
  background-size: cover;
}
```

- src/sass/index.sass

```css
.box3
  width: 100px
  height: 100px
  background-image: url("../images/2.png")
  background-size: cover
```

- src/styl/index.styl

```css
.box5
  width 100px
  height 100px
  background-image url("../images/3.gif")
  background-size cover
```

## 4. 运行指令

```:no-line-numbers
npx webpack
```

打开 index.html 页面查看效果

## 5. 输出资源情况

此时如果查看 dist 目录的话，会发现多了三张图片资源

因为 Webpack 会将所有打包好的资源输出到 dist 目录下

- 为什么样式资源没有呢？

因为经过 `style-loader` 的处理，样式资源打包到 main.js 里面去了，所以没有额外输出出来

## 6. 对图片资源进行优化

将小于某个大小的图片转化成 data URI 形式（Base64 格式）

```js{32-36}
const path = require("path");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "main.js",
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
            maxSize: 10 * 1024 // 小于10kb的图片会被base64处理
          }
        }
      },
    ],
  },
  plugins: [],
  mode: "development",
};
```

- 优点：减少请求数量
- 缺点：体积变得更大

此时输出的图片文件就只有两张，有一张图片以 data URI 形式内置到 js 中了
（注意：需要将上次打包生成的文件清空，再重新打包才有效果）
