---
title: HTTP/2 知识点梳理
tags: 浏览器
---

## HTTP/2 知识点梳理



### 1. **知识点**

**HTTP/2 中，同域名下所有通信都在单个连接上完成，该连接可以承载任意数量的双向数据流。**

-  **帧：**HTTP/2 数据通信的最小单位消息：指 HTTP/2 中逻辑上的 HTTP 消息。例如请求和响应等，消息由一个或多个帧组成。

- **流：**存在于连接中的一个虚拟通道。流可以承载双向消息，每个流都有一个唯一的整数ID。

#### **多路复用**：

 代替原来的序列和阻塞机制。所有就是请求的都是通过一个 TCP连接并发完成。 HTTP 1.x 中，如果想并发多个请求，必须使用多个 TCP 链接，且浏览器为了控制资源，还会对单个域名有 6-8个的TCP链接请求限制。



#### **服务器推送**

服务端可以在发送页面HTML时主动推送其它资源，而不用等到浏览器解析到相应位置，发起请求再响应。例如服务端可以主动把JS和CSS文件推送给客户端，而不需要客户端解析HTML时再发送这些请求。

服务端可以主动推送，客户端也有权利选择是否接收。如果服务端推送的资源已经被浏览器缓存过，浏览器可以通过发送RST_STREAM帧来拒收。主动推送也遵守同源策略，服务器不会随便推送第三方资源给客户端。



![image-20211217174305422](https://tva1.sinaimg.cn/large/008i3skNly1gxgyzfzh45j31740mg75n.jpg)



```javascript
const http2 = require('http2');
const fs = require('fs');
const path = require('path')

const server = http2.createSecureServer({
  key: fs.readFileSync(path.resolve('../../Documents/keys/localhost-privkey.pem')),
  cert: fs.readFileSync(path.resolve('../../Documents/keys/localhost-cert.pem'))
});
server.on('error', (err) => console.error(err));

server.on('stream', (stream, headers) => {
  // 流是双工的
  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200
  });
  stream.pushStream({ ':path': '/index.css' }, (err, pushStream, headers) => {
    if (err) throw err
    pushStream.respond({ ':status': 200 , 'content-type': 'application/json'});
    pushStream.end(fs.readFileSync('./index.css'))
  })
  stream.end(fs.readFileSync('./index.html'));
});

server.listen(8080);
```



![image-20211217175216824](https://tva1.sinaimg.cn/large/008i3skNly1gxgz9043lzj31oi0dsju4.jpg)



### 2. Node 启动 HTTP/2



```bash
# 本地生成证书&秘钥
openssl req -x509 -newkey rsa:2048 -nodes -sha256 -subj '/CN=localhost' \
  -keyout localhost-privkey.pem -out localhost-cert.pem
```



```javascript
const http2 = require('http2');
const fs = require('fs');
const path = require('path')

const server = http2.createSecureServer({
  key: fs.readFileSync(path.resolve('../../Documents/keys/localhost-privkey.pem')),
  cert: fs.readFileSync(path.resolve('../../Documents/keys/localhost-cert.pem'))
});
server.on('error', (err) => console.error(err));

server.on('stream', (stream, headers) => {
  // 流是双工的
  stream.respond({
    'content-type': 'text/html; charset=utf-8',
    ':status': 200
  });
  stream.end('<h1>Hello World</h1>');
});

server.listen(8080);
```

![image-20211217172158431](https://tva1.sinaimg.cn/large/008i3skNly1gxgydh6hzfj31je0tk0vg.jpg)

### 3. 注意：

- HTTP/2 客户端浏览器基于 SSL/TLS 之上，所以实现 HTTP/2的 Web 浏览器都只支持加密。但是针对 RPC 通讯，并没有特殊的限制，例如：gRPC框架的实现就是基于 HTTP/2 建立在TCP层上通讯的。

