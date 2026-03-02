# Loader 的写法

在 Webpack 的 `module.rules` 里，指定用哪个 loader 处理文件有两种方式：`loader` 或 `use`。二者都支持多种写法，按「单个/多个、有无选项」选即可。以下写法均以 [Webpack 中文文档](https://webpack.docschina.org/loaders/babel-loader/) 为准。

---

## 一、写法总览

| 写法 | 适用场景 |
|------|----------|
| `loader: "babel-loader"` | 只有一个 loader 且无选项时的简写 |
| `use: "babel-loader"` | 单个 loader，无选项，字符串 |
| `use: ["style-loader", "css-loader"]` | 多个 loader，都无选项，字符串数组 |
| `use: { loader: "babel-loader", options: {} }` | 单个 loader 带选项，**一个对象**（官网常用） |
| `use: [{ loader: "babel-loader", options: {} }]` | 单个 loader 带选项，数组里一个对象 |
| `use: [{ loader, options? }, { loader, options? }, ...]` | 多个 loader，且有的要传选项，**必须用数组 + 对象** |

要点：**只有一个 loader 且要传 options 时**，可以写 `use: { loader, options }`，不必包在数组里。

---

## 二、按场景怎么选

### 1. 一个 loader、无选项

用 `loader` 或 `use` 字符串均可：

```js
{ test: /\.js$/, loader: "babel-loader" }
// 或
{ test: /\.js$/, use: "babel-loader" }
```

### 2. 一个 loader、有选项

推荐用 **对象** 形式的 `use`（官网常见写法），也可用数组包一个对象：

```js
// 推荐：直接一个对象
{
  test: /\.js$/,
  use: {
    loader: "babel-loader",
    options: {
      presets: ["@babel/preset-env"],
    },
  },
}

// 也可以：数组里一个对象
{ test: /\.js$/, use: [{ loader: "babel-loader", options: { presets: ["@babel/preset-env"] } }] }
```

### 3. 多个 loader

必须用 **数组**。都无选项时用字符串数组；有选项的写成对象放进数组：

```js
// 都无选项
use: ["style-loader", "css-loader"]

// 有的有选项：全部写成对象
use: [
  { loader: "style-loader", options: { injectType: "styleTag" } },
  { loader: "css-loader", options: { modules: true } },
]
```

---

## 三、示例：多个 loader 且都带选项

例如给 JS 先经 `thread-loader` 再经 `babel-loader`，且都要传选项：

```js
{
  test: /\.m?js$/,
  exclude: /node_modules/,
  use: [
    { loader: "thread-loader", options: { workers: 2 } },
    {
      loader: "babel-loader",
      options: {
        presets: ["@babel/preset-env"],
        cacheDirectory: true,
      },
    },
  ],
}
```

以上写法适用于所有 loader，不限于 babel-loader。
