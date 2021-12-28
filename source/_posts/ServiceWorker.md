---
title: Service Worker
tags: 浏览器
---

##  Service Worker



### 1. 概念区分

PWA / Service Worker / WorkBox 虽然相互之间有着千丝万缕的联系，但是，他们工作的位面不一样。

![image-20211221112823453](https://tva1.sinaimg.cn/large/008i3skNly1gxlamuaofxj31240fmgmp.jpg)



#### 1. 1 Service Worker 

- Service Worker 工作在浏览器的worker进程中，API设计是纯异步的。所以同步的API在不能在其上面调用和使用。
- Service worker运行在worker上下文，因此它不能访问DOM。
- Service workers只能由HTTPS承载。（开发环境认可： localhost or 127.0.0.1）
- 使用前，需要确认兼容性。（https://caniuse.com/?search=Service%20Worker）



#### 1.2 PWA （Progressive Web App）

- 用户端存储：Cache、IndexedDB，但是使用前要注意不同浏览器厂商对应的存储大小是有限的。溢出的话容易造成I/O失败。

- 关注 ： https://web.dev/pwa-checklist/#exemplary 详细列举了 PWA 有哪些优势。

- 请搞清楚如下几个 Manifest

  | `html`标签的`manifest`属性: [离线缓存](https://link.segmentfault.com/?enc=30FUPA%2B0OF6S8jcR%2Btiy6w%3D%3D.HQfRYNa7bCuePN0A7niiqhgM5Puuf2cpEO2DOXmUdx%2FGMnuhEc2LPlRxvzCBpKmCqUoecRqV9Djwdtd5VlSsJt4E%2BPM%2BV68ZFZgexgB0NbI%3D)（目前已被废弃） |
  | ------------------------------------------------------------ |
  | [PWA](https://link.segmentfault.com/?enc=3uYS%2BFS2GnqcfbLMGXI4jg%3D%3D.cMqJMTrtQpDJhGjKLO68V2f2C6jpt9ducPhvcYmscQYNzA8h9A3rCsbYm%2Fio9UTuBfUZ7YEKEINj2xE1H%2FGrLw%3D%3D): 将Web应用程序安装到设备的主屏幕 |
  | webpack中[webpack-manifest-plugin](https://link.segmentfault.com/?enc=l3XiYvyvmeI3JdAAqzzvrQ%3D%3D.ktTdDfXIuLCHOKl%2BvDPi3HXlUrXH6l3xGk3y264UcPN0ekiR4P98WtlW0t5lJ7msjmZn%2FeTAbFkIItR0D1ckfQ%3D%3D)插件打包出来的`manifest.json`文件，用来生成一份资源清单，为后端渲染服务 |
  | webpack中[DLL](https://link.segmentfault.com/?enc=nlZhAFjfVIrvKisyS%2FszGw%3D%3D.qMADE2SbyN1uREswaprW7SqyrlrjoMTHAc1WCZ%2FxSK0N%2FqkOOtFKTraF21YFUjvI)打包时,输出的`manifest.json`文件，用来分析已经打包过的文件，优化打包速度和大小 |
  | webpack中[manifest](https://link.segmentfault.com/?enc=mA6r%2FdGrhlWqzosrYJc5eQ%3D%3D.89YiF%2BuDET6nLBPORl39UYZmPdk2eFLyTMgNVeWjOWd4m8fmUR2s%2FYJ1oyMZnkYP)运行时代码 |



下面是 PWA 中 manifest 的编写示例：

```json
{ 
  "name" : "Minimal PWA" , 
  "short_name" : "PWA Demo" , 
  "display" : "standalone" , 
  "start_url" : "/" , 
  "theme_color" : "#313131" , 
  "background_color" : "#313131" , 
  "icons" : [ 
    {
      "src": "images/touch/homescreen48.png",
      "sizes": "48x48",
      "type": "image/png"
    }
   ] 
}
```

#### 1.3 workbox

Workbox 是一组库，可以为 Progressive Web App 提供生产就绪的 Service Worker。

![image-20211222202841855](https://tva1.sinaimg.cn/large/008i3skNly1gxmvvbunp3j31l40kmq4o.jpg)



### 2. 构建 service worker



#### 2.1 页面创建定义 sw



说明： 由于我们构建的应用可能是多页面的，每一个工程都可能会存在service worker，为防止 service worker 爆炸，我们在定义之初最好定义作用范围。

```javascript
if ('serviceWorker' in navigator) {
       navigator.serviceWorker.register('/sw.js', { 
         scope: '/' 
       }).then(function(swReg) {
				// 此处编写加载sw之后的逻辑
       }).catch(function(error) {
       // registration failed
       console.log('Registration failed with ' + error);
     });
}
```



#### 2.2 定义规则规则

```javascript
// 注册成功后要立即缓存的资源列表
workbox.precaching.precache([
  'https://avatars.githubusercontent.com/iceprosurface'
]);

// html的缓存策略
workbox.routing.registerRoute(
  '/(.*?)\/$',
  new workbox.strategies.NetworkFirst({
    cacheName: 'index',
    plugins: [
      new workbox.expiration.Plugin({
        maxAgeSeconds: 7 * 24 * 60 * 60,
      }),
    ],
  }),
)

// html的缓存策略
workbox.routing.registerRoute(
  new RegExp('.*.html'),
  new workbox.strategies.NetworkFirst({
    cacheName: 'html-main',
    plugins: [
      new workbox.expiration.Plugin({
        maxEntries: 20,
        maxAgeSeconds: 7 * 24 * 60 * 60,
      }),
    ],
  }),
)

workbox.routing.registerRoute(
  new RegExp('.*.(js|css)'),
  new workbox.strategies.NetworkFirst({
    cacheName: 'icepro-resource',
    plugins: [
      new workbox.expiration.Plugin({
        maxEntries: 20,
        maxAgeSeconds: 7 * 24 * 60 * 60,
      }),
    ],
  }),
)

workbox.routing.registerRoute(
  new RegExp('https://icepro.oss-cn-shanghai.aliyuncs.com/'),
  new workbox.strategies.NetworkFirst({
    cacheName: 'image-oss',
  }),
)
```



#### 2.3 效果展示

![image-20211228130607493](https://tva1.sinaimg.cn/large/008i3skNly1gxtgsnoecnj30i009ymy2.jpg)



![image-20211228130702137](https://tva1.sinaimg.cn/large/008i3skNly1gxtgtlep4rj31qe0smwjm.jpg)

