# 减少代码体积

## Tree Shaking

### 为什么

开发时我们定义了一些工具函数库，或者引用第三方工具函数库或组件库。

如果没有特殊处理的话我们打包时会引入整个库，但是实际上可能我们可能只用上极小部分的功能。

这样将整个库都打包进来，体积就太大了。

### 是什么

`Tree Shaking` 是一个术语，通常用于描述移除 JavaScript 中的没有使用上的代码。

**注意：它依赖 `ES Module`。**

### 怎么用

Webpack 已经默认开启了这个功能，无需其他配置。

## Babel

### 为什么

Babel 为编译的每个文件都插入了辅助代码，使代码体积过大！

Babel 对一些公共方法使用了非常小的辅助代码，比如 `_extend`。默认情况下会被添加到每一个需要它的文件中。

```js
// 【转换前】没有 plugin-transform-runtime 时，每个文件都会冗余内联 helper
// a.js
function _classCallCheck() {
    /* 重复代码 */
}
class A {}

// b.js
function _classCallCheck() {
    /* 重复代码 */
}
class B {}
```

你可以将这些辅助代码作为一个独立模块，来避免重复引入。

### 是什么

`@babel/plugin-transform-runtime`

- **定位**：编译时的工具（安装在 `devDependencies`）。
- **职责 1**：提取runtime库中 Babel 辅助函数，防止重复注入，减小打包体积（我们这次使用的功能）
- **职责 2**：配合 `corejs` 配置，提供沙箱式的环境，防止 Polyfill 污染全局作用域（Global Scope）。

`@babel/runtime`

- **定位**：运行时的代码库（必须安装在 `dependencies`）。
- **内容**：只包含 helpers 函数实现（如 `_classCallCheck`）和 `regenerator-runtime`（用于 async/await）。
- **限制**：**不包含**像 `Promise`、`Array.prototype.includes` 等现代 API 的 Polyfill垫片。

两者必须结合使用：`@babel/plugin-transform-runtime`在编译时工作，自动将代码中内联的 Babel 辅助函数替换为从 `runtime` 库中统一引入，`@babel/runtime`提供 Babel 辅助函数（helpers）的实际运行的时代码

```js
// 【转换后】使用 plugin-transform-runtime 后，统一从模块按需引入
// a.js
import _classCallCheck from "@babel/runtime/helpers/classCallCheck";
class A {}

// b.js
import _classCallCheck from "@babel/runtime/helpers/classCallCheck"; // 完美复用
class B {}
```



### 怎么用

1. 下载包

```
npm i @babel/plugin-transform-runtime -D
npm i @babel/runtime -S
```

2. 配置

```json{5-12}
{
  "presets": [
    "@babel/preset-env"
  ],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": false // 告诉该插件：不要去处理 API Polyfill 的事情。
      }
    ]
  ]
}

```

## Image Minimizer

### 为什么

开发如果项目中引用了较多图片，那么图片体积会比较大，将来请求速度比较慢。

我们可以对图片进行压缩，减少图片体积。

**注意：如果项目中图片都是在线链接，那么就不需要了。本地项目静态图片才需要进行压缩。**

### 是什么

`image-minimizer-webpack-plugin`: 用来压缩图片的插件

### 怎么用

1. 下载包

```bash
npm install image-minimizer-webpack-plugin sharp svgo --save-dev
```

`sharp`：目前 Webpack 官方推荐的高性能图片压缩底层引擎，它速度不仅极快，而且安装友好，支持最新的 WebP 和 AVIF 格式。

`svgo`：svg压缩，纯 JS 写的，很好用。

2. 配置

我们以无损压缩配置为例：

```js{1,10-77}
const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin");

module.exports = {
  // ... 其他配置
  optimization: {
    minimizer: [
      // 其他的 css / js 压缩器...

      // 最新的图片压缩配置
      new ImageMinimizerPlugin({
        minimizer: {
          // 这里强制指定使用 sharp 引擎
          implementation: ImageMinimizerPlugin.sharpMinify,
          options: {
            encodeOptions: {
              // sharp 的压缩参数，你可以按需调整质量
              jpeg: {
                // https://sharp.pixelplumbing.com/api-output#jpeg
                quality: 80,
              },
              webp: {
                // https://sharp.pixelplumbing.com/api-output#webp
                lossless: true,
              },
              avif: {
                // https://sharp.pixelplumbing.com/api-output#avif
                lossless: true,
              },
              // png 默认使用 zlib 压缩，通过 quality 设置可以开启有损压缩（需要 libvips 支持）
              png: {
                // 设置 1-100 的数值。数值越小图片越小，但低于 60 可能会有肉眼可见色彩断层
                // 官方建议值在 75-80 左右，体积能大幅缩小且肉眼几乎无损
                quality: 80,
                // palette 参数：进一步将图片转为基于调色板的索引色格式，
                // 也能大幅度降低某些卡通或 UI 图标类型的 PNG 体积
                palette: true,
              },
              // gif不支持无损压缩
              // https://sharp.pixelplumbing.com/api-output#gif
              gif: {},
            },
          },
        },
      }),
      // 注意：sharp 目前不处理 SVG，如果你项目中有很多 SVG 并且需要压缩
      // 你需要再 new 一个专门处理 svg 的实例
      new ImageMinimizerPlugin({
            minimizer: {
                implementation: ImageMinimizerPlugin.svgoMinify,
                options: {
                    encodeOptions: {
                        // svgo 的配置项
                        multipass: true,
                        plugins: [
                            "preset-default",
                            // 确保不要移除 SVG 中的 viewBox 属性，否则会导致各种自适应问题
                            // svgo 新版本中已将其移出 preset-default，可直接在这里配置
                            {
                                name: "removeViewBox",
                                active: false,
                            },
                            {
                                name: "addAttributesToSVGElement",
                                params: {
                                    attributes: [
                                        // 强制让每一个 SVG 标签都加上合规的身份证 xmlns，彻底解决浏览器兼容兼容性渲染破图的玄学 Bug
                                        {
                                            xmlns: "http://www.w3.org/2000/svg",
                                        },
                                    ],
                                },
                            },
                        ],
                    },
                },
            },
        }),
    ],
  },
};

```
