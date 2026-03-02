# 修改输出资源的名称和路径

默认情况下，`asset` 模块以 `[hash][ext][query]` 文件名发送到输出目录。

可以通过在 webpack 配置中设置`Rule.generator.filename` 与 [`output.assetModuleFilename`](https://www.webpackjs.com/configuration/output/#outputassetmodulefilename)来修改此模板字符串，并且仅适用于 `asset` 和 `asset/resource` 模块类型。

**最佳实践：**

在 output 里设置一个比如叫 media 或 assets 的通用目录作为“兜底”，然后对于图片，再在 rule 里单独覆盖它

```js
// ✨ 全局兜底：所有没单独配置 generator 的资源，都放到 media 文件夹
assetModuleFilename: 'static/media/[hash][ext][query]',

{
    test: /\.(png|jpe?g|gif|webp)$/,
    type: "asset",
    generator: {
        filename: "images/[hash:10][ext][query]" // ✨ 优先级更高：覆盖兜底配置，放到 images 目录
    }
},
{	
    test: /\.(woff2?|eot|ttf|otf)$/,
    type: "asset/resource",
    // 这里不写 generator，就会默认触发兜底，被放进 media/ 目录下
}
```



## 1. 配置

```js{7,8,9,39-47}
const path = require("path");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    // ✨ 全局兜底：所有没单独配置 generator 的资源，都放到 media 文件夹
    assetModuleFilename: 'static/images/[hash][ext][query]',
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
          // ✨ 优先级更高：覆盖兜底配置，放到 images 目录
          filename: "static/images/[hash:8][ext][query]",
        },
      },
    ],
  },
  plugins: [],
  mode: "development",
};
```

## 2. 修改 index.html

```html{17}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>webpack5</title>
  </head>
  <body>
    <h1>Hello Webpack5</h1>
    <div class="box1"></div>
    <div class="box2"></div>
    <div class="box3"></div>
    <div class="box4"></div>
    <div class="box5"></div>
    <!-- 修改 js 资源路径 -->
    <script src="../dist/static/js/main.js"></script>
  </body>
</html>
```

## 3. 运行指令

```:no-line-numbers
npx webpack
```

- 此时输出文件目录：

（注意：需要将上次打包生成的文件清空，再重新打包才有效果）

```
├── dist
    └── static
         ├── imgs
         │    └── 7003350e.png
         └── js
              └── main.js
```
