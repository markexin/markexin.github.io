---
title: Nuxt 3 概述
tags: vue
---

# Nuxt 3 概述



## 一.  Nuxt 框架



### 1. 概述

**Nuxt 当前版本介绍：** Build your next Vue.js application with confidence using Nuxt. An open source framework making web development simple and powerful.

**Nuxt 3 新版本介绍：**Build your next application with Vue 3 and experience hybrid rendering, powerful data fetching and new features. Nuxt 3 is an open source framework making web development simple and powerful.

**Nuxt** 框架，是一个集百家之所长，可以让开发者不用关心同构、注水等 **SSR** 细节，快速构建**WEB**应用的一个开源框架。**Nuxt3** 基于**vue3.0** 版本的一些新的特性，相比于nuxt之前版本，提供更快、更强、更轻量的能力。



### 2. 版本更新概述

- Webpack 5 and Vite 支持
- PostCSS 8
- rollup-plugin-esbuild 融合了 Esbuild、Rollup等优秀特性
- Vue-bundle-renderer 基于 vue ssr 官方提供的基础API 
- 基于 Composition API



### 3. Nuxt 3 工程

**仓库地址：** https://github.com/nuxt/framework

**工程结构简述：**

![image-20211227192251521](https://tva1.sinaimg.cn/large/008i3skNly1gxsm2c2ar5j31i40ts41k.jpg)



## 二. SSR 简史



### 1. web 1.0 时代



![image-20211213194138296](https://tva1.sinaimg.cn/large/008i3skNly1gxgtwi248qj315h0n9mye.jpg)



| 优势       | 劣势                                 |
| ---------- | ------------------------------------ |
| 更好的SEO  | 协同开发困难                         |
| FCP 耗时短 | 服务端对数据库频繁操作带来的性能损耗 |



### 2. SSR

> CSR: 客户端渲染(Client Side Render)

伴随着AJAX技术的不断发展，渐渐出现前后端分离、CSR 慢慢进入到了公众的视野。但是随着业务不断的繁重，打包编译后的Bundle文件随着代码量的增加，业务功能渐渐丰满，无脑引入第三方开源库，很快变成一个巨婴项目。

此时，SSR （ Server Side Rendering ）重新被提起和认识。

![image-20211213194213452](https://tva1.sinaimg.cn/large/008i3skNly1gxgtwu4q0ij315l0msmyq.jpg)



![image-20211213195240072](https://tva1.sinaimg.cn/large/008i3skNly1gxgtx3h2i6j315l0ll75r.jpg)



#### 2.1 概念说明

| 概念（行话、黑话） | 含义                                                         |
| ------------------ | :----------------------------------------------------------- |
| 注水               | 抵达客户端后，会将一部分数据、DOM结构体等注入到HTML页面中    |
| 吸水               | 抵达客户端后，当期环境运行javascript脚本，发起第二次增量渲染 |
| 脱水               | 使其在恶劣的环境同样能够以一种更简单的形态“生存”下来，比如禁用了 JavaScript 的客户端环境，脱去生命的水气（动态数据），成为风干标本一样的静态快照。 |
| 同构               | 当我们有且只有一份源代码，要实现不同环境的运行要求所采取的一种构建方式；例如：客户端运行的Bundle需要适配低版本浏览器，转成ES5代码；服务端Bundle不需要进行代码混淆压缩；服务端Bundle不需要安装css-loader等插件。 |

#### 2.2 流行框架支持

![image-20211213195907961](https://tva1.sinaimg.cn/large/008i3skNly1gxgtxf5flbj315y0iwjsr.jpg)

#### 2.3 Nuxt 3 举例

![image-20211213200008949](https://tva1.sinaimg.cn/large/008i3skNly1gxgtxkxq9aj316c0nsq8e.jpg)

## 三. Nitro 服务引擎



### 1. Nitro 简介

Nuxt 的新服务器引擎，Nuxt研发团队历时 9 个月，解锁的服务引擎。实现类似于 Next.js 中 API 路由。在生产中，它将您的应用程序和服务器构建到一个通用[`.output` ](https://v3.nuxtjs.org/docs/directory-structure/output)目录中。此**输出很轻**：从任何 Node.js 模块（polyfill 除外）中缩小并删除。您可以将此输出部署到任何支持 JavaScript 的系统上，从 Node.js、Serverless、Workers、或纯静态。Nitro 服务器的基础是 rollup 和[h3](https://github.com/unjs/h3)：为高性能和可移植性而构建的最小 http 框架，如下图，Nitro 实际上相当于下图红色框部分。

![image-20211213223119233](https://tva1.sinaimg.cn/large/008i3skNly1gxgtxt92n9j31h20u076z.jpg)



### 2. h3 概述

Nitro 核心基于h3， 仓库地址： https://github.com/unjs/h3，试想一下，当你想实现一个SSR HTTP服务时，需要做那些事？

```javascript
// 创建HTTP服务
const http = require('http')

// 创建server
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    data: 'Hello World!'
  }));
});

server.listen(8000);
```

当你有了一个服务之后，就需要根据req path去寻找对应源文件中的components。

![image-20211227192318025](https://tva1.sinaimg.cn/large/008i3skNly1gxsm2snok3j31l807ewfd.jpg)



对req path normalize的过程，发生在 **createServer** 第一个参数 Function 中。实际上，h3 这个库主要就是在做这个事情。总结一下，如下如：



![image-20211227192326296](https://tva1.sinaimg.cn/large/008i3skNly1gxsm2xu7gtj31iu0hygmw.jpg)

### 3. Nitro 核心模块

```javascript
export * from './build'
export * from './context'
export * from './server/middleware'
export * from './server/dev'
export { wpfs } from './utils/wpfs'
```

梳理模块功能如下图所示：

![image-20211227193521127](https://tva1.sinaimg.cn/large/008i3skNly1gxsmfc33ycj319c0l2wg8.jpg)

其中，中间件 Middleware 目前主要服务于开发环境下，server 端 Bundle 文件的热更新。详细代码如下：

```javascript
// 1. 初始化 worker
function initWorker (filename): Promise<NitroWorker> {
  return new Promise((resolve, reject) => {
    const worker = new Worker(filename)
    worker.once('exit', (code) => {
      if (code) {
        reject(new Error('[worker] exited with code: ' + code))
      }
    })
    worker.on('error', (err) => {
      console.error('[worker]', err)
      err.message = '[worker] ' + err.message
      reject(err)
    })
    worker.on('message', (event) => {
      if (event && event.address) {
        resolve({
          worker,
          address: event.address
        } as NitroWorker)
      }
    })
  })
}

// 2. 观察更新
function watch () {
  if (watcher) { return }
  const dReload = 
        debounce(() => 
                 reload().catch(console.warn), 
                 200, 
                 { before: true })
  watcher = chokidar.watch([
    resolve(nitroContext.output.serverDir, pattern),
    resolve(nitroContext._nuxt.buildDir, 'dist/server', pattern)
  ]).on('all', 
        event => events.includes(event) && dReload()
  )
}

// 3. 执行更新
async function reload () {
  // Create a new worker
  const newWorker = await initWorker(workerEntry)

  // Kill old worker in background
  killWorker(currentWorker).catch(err => console.error(err))

  // Replace new worker as current
  currentWorker = newWorker
}
```

从上述代码中我们可以看到，当你 **dist/server** 文件下内容发生更新，会触发 watch 监听，并执行 reload 操作。此时，用到了一个 node 原生模块 **worker_threads** 。



### 4. 聊聊 worker_threads

Node 创建多个workers有如下几种方式：

- 利用 child_process 创建子进程，子进程拥有独立的 V8 和 Libuv，父子之间可以通讯。 

- 基于 child_process 利用CPU多核提供可负载均衡集中式管理 Cluster，父子之间可以通讯，内存不共享，但是子进程间是不能通讯的。

- 利用worker_threads创建多个workers是可以实现内存共享的，因为在创建worker_threads worker 同时，底层 C++ 创建的是 **V8 Isolates**，它是一个独立的 chrome V8 运行实例，其有独立的 JS 堆和微任务队列。这就为每一个 Node.js worker 独立运行提供了保障。其缺陷就是，workers 之间没法直接访问对方的堆。由于这个原因，每个 worker 都有其自己的 [libuv](https://link.zhihu.com/?target=https%3A//github.com/libuv/libuv) event loop。

  

![image-20211227193553758](https://tva1.sinaimg.cn/large/008i3skNly1gxsmfwaha0j30y80u0tav.jpg)



#### 4.1 **初始化阶段**

1. 用户空间的脚本通过 `worker_threads` 模块创建一个 worker 实例
2. Node.js 的 parent worker 初始化脚本调用 C++ 模块，创建一个空的 C++ worker 对象。
3. 当 C++ worker 对象被创建后，它生成一个线程 ID 并分配给自己
4. 当 worker 对象被创建的时候，parent worker 会初创建一个空的始化信道（让我们称它为 `IMC`）。就是上面图二的 “Initialisation Message Channel”
5. Node.js 的 parent worker 初始化脚本创建一个公共的 JS 信道（让我们称它为 `PMC`）。该信道是给用户空间的 JS 使用的，便于他们在 parent worker 和 child worker 之间通过 `*.postMessage()` 方法来传递消息。就是上面图一和图二的红色部分
6. Node.js 的 parent worker 初始化脚本调用 C++ 模块，将初始化的 metadata 写入 IMC 来传递给 worker 的执行脚本。

#### 4.2 **运行阶段**

此时，初始化阶段完成。然后，就会调用 C++ 来启动这个 worker 线程。

1. 一个新的 v8 isolate 被创建并分配给这个 worker。这使得该 worker 可以拥有自己的运行时环境
2. libuv 被初始化。这使得该 worker 可以拥有自己的 event loop
3. Worker 初始化脚本被执行，启动 worker 的 event loop
4. Worker 初始化脚本 调用 C++ 模块从 IMC 读取初始化 metadata
5. Worker 执行脚本在 worker 中运行代码文件或代码片段。在我们的例子中就是 `worker-simple.js`



## 四. Nuxt Bridge

Bridge 是一个向前兼容层，它允许您通过简单地安装和启用 Nuxt 模块来体验许多新的 Nuxt 3 功能。

使用 Nuxt Bridge，您可以确保您的项目（几乎）为 Nuxt 3 做好准备，并拥有最佳的开发人员体验，而无需进行重大重写或冒险更改。

建议参考一下官网文档： https://v3.nuxtjs.org/getting-started/bridge



## 五. 结束语

Nuxt 3 目前仍处于Beta阶段，h3 & Nitro 仍在不断适配中，所以不建议应用于生产环境。作为学习，当我们透过本质去回忆 SSR 实现的全过程，那么 Nuxt 3 源码部分就不会生涩难懂。Nuxt 不仅仅是搬运了其他框架的优势，自己在内部也有很多比较有意思的实现。比如 mini 版本的 tapable。在Nuxt 2中定义为 Hable。它贯穿了整个 Nuxt 实例化后的生命周期。有兴趣的同学我们可以一起探索学习一下~