---
title: Webpack Chunk
tags: 性能优化
---

# Webpack Chunk

### 起源

| 名称 | 版本 |
| :--- | :--- |
| CommonsChunkPlugin | webpack 3 |
| optimization.splitChunks | webpack 4+概念 |

### Chunk  vs Bundle

Chunk是过程中的代码块，Bundle是结果的代码块。

### 产生Chunk的三种途径

#### 1. entry

```javascript
{
    entry: './src/js/main.js',
}
{
    entry: ['./src/js/main.js','./src/js/other.js'],
}
```

这种情况也只会产生一个Chunk。Webpack会将数组里的源代码，最终都打包到一个Bundle里，原因就是只生成了一个Chunk。

#### 2. 异步

```javascript
const myModel = r => require.ensure([], () => r(require('./myVue.vue')), 'myModel')
```

chunkFilename字段就派上用场了，为异步加载的Chunk命名。

#### 3. 代码分隔

**hash**

* 基于build
* 所有chunk文件使用相同的hash。
* 项目中任一文件内容发生变化都会影响所有chunk文件hash

**chunkhash**

* 基于 webpack 的 `entry point`
* 任意文件改变只会影响其所属的chunk，不会影响其它chunk。

**contenthash**

* 基于文件内容产生hash
* 影响范围只限与本文件，多用于 CSS文件。当我们JS文件引入CSS时，由于利用了 css extract-text-webpack-plugin 

**总结： hash、chunkhash、contenthash 实际上是对我们重复打包编译之后，对外部用户群体的一种优化手段。**

初始化最简易的项目，记录每一个chunk名称和chunk size

![image-20211227203350894](https://tva1.sinaimg.cn/large/008i3skNly1gxso47dcuxj31lm0bq41g.jpg)

当 test.js 中引入了我们所谓的第三方插件库

![image-20211227203400531](https://tva1.sinaimg.cn/large/008i3skNly1gxso4cw6s0j31l40hsq54.jpg)

如图可知，require 引入的模块，是不会被 Webpack Tree Shaking 掉的。

```javascript
const path = require('path');
const webpack = require('webpack');
module.exports = {
  mode: "production",
  entry: {
      index: path.resolve(__dirname, 'src') + '/index.js',
      test: path.resolve(__dirname, 'src') + '/test.js',
      vendor: ["react", "react-dom"],
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: '[name].js'
  },
};
```

![image-20211227203412445](https://tva1.sinaimg.cn/large/008i3skNly1gxso4kg63hj31li0kwwlz.jpg)



