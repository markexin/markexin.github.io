## RPC框架  - Thrift



### 一.  Thrift 安装

1. 安装thrift

```bash
# 主要还是针对 linux 和 macOS系统
brew install thrift
```

2. 编写一个 IDL 文件

````thrift
struct TAccountRpcDTO {
    1: i64 uid,
    2: string name,
}

service TAccountService {
    // 创建一个账号
    void createAccount(1: TAccountRpcDTO tAccountRpcDTO);

    // 获取某个账号
    TAccountRpcDTO getAccount(1: i64 uid);
}
````

3. 创建server端

```javascript
const thrift = require('thrift');

// 引入生成的struct和service方法
const TAccountService = require('./gen-nodejs/TAccountService')

// 模拟数据库，本地创建一个对象，用作存储account
const accounts = {};

// 开启rpc server
const server = thrift.createServer(TAccountService, {
	createAccount(account, result) {
		console.log('server createAccount:', account.uid);

		accounts[account.uid] = account;
		result(null);
	},

	getAccount(uid, result) {
		console.log('server getAccount:', uid);

		result(null, accounts[uid]);
	},
});

// 监听3030端口
server.listen(3030);
```

4. 创建client

```java
const thrift = require('thrift');

// 引入生成的struct和service方法
const TAccountService = require('./gen-nodejs/TAccountService')
const TAccountTypes = require('./gen-nodejs/hello_types')

// 建立连接和初始化client
const connection = thrift.createConnection('127.0.0.1', 3030, {
    connect_timeout: 100,
    max_attempts: 2 
});

const client = thrift.createClient(TAccountService, connection);

// 定义一个新的accountDto对象，用作测试
const account = new TAccountTypes.TAccountRpcDTO({
	uid: 1,
	name: 'Hello World',
});

// 监听error事件，node的NodeJS.EventEmitter类的on方法
connection.on('error', err => {
	console.error(err);
});

// 调用server端的createAccount方法
client.createAccount(account, err => {
	if (err) {
		console.error(err);
		return;
	}

	console.log('client createAccount:', account.uid);

	// 获取刚刚”新建“的account
	client.getAccount(account.uid, (err, resp) => {
		if (err) {
			console.error(err);
			return;
		}

		console.log(`client getAccount: uid=${resp.uid}, name=${resp.name}`);
		connection.end();
	});
});
```

5. 启动服务

```bash
# 启动服务
node server.js
# 启动客户端
node client.js
```

6. 打印结果

![image-20211212220740992](/Users/maqun/Documents/fontend-note/docs/media/image-20211212220740992.png)

![image-20211212220800393](/Users/maqun/Documents/fontend-note/docs/media/image-20211212220800393.png)



### 二. IDL 编写

1. 基本类型

| 类型   | 含义    |
| ------ | ------- |
| bool   | Boolean |
|        |         |
| i16    | Short   |
| I32    |         |
| I64    |         |
| double | double  |
| String | String  |
| binary | Byte[]  |

2. struct 结构体

```java
struct User {
  1: required string name, //改字段必须填写
  2: optional i32 age = 0; //默认值
  3: bool gender //默认字段类型为optional
}
```

- struct不能继承，但是可以嵌套，不能嵌套自己。
- 其成员都是有明确类型
- 成员是被正整数编号过的，其中的编号使不能重复的，这个是为了在传输过程中编码使用。
- 成员分割符可以是逗号（,）或是分号（;），而且可以混用
- 字段会有optional和required之分和protobuf一样，但是如果不指定则为无类型–可以不填充该值，但是在序列化传输的时候也会序列化进去，optional是不填充则部序列化，required是必须填充也必须序列化。
- 每个字段可以设置默认值
- 同一文件可以定义多个struct，也可以定义在不同的文件，进行include引入。



3. Container 容器

   ```java
   struct Test {
     1: map<string, User> usermap,
     2: set<i32> intset,
     3: list<double> doublelist
   }
   ```

   - list: 元素类型为t的有序表，容许元素重复。对应c++的vector，java的ArrayList或javascript的数组
   - set: 元素类型为t的无序表，不容许元素重复。对应c++中的set，java中的HashSet,python中的set，php中没有set，则转换为list类型了
   - map<t, t>: 键类型为t，值类型为t的kv对，键不容许重复。对用c++中的map, Java的HashMap, PHP 对应 array, Python/Ruby 的dictionary

4. Enum 枚举

   ```java
   enum HttpStatus {
     OK = 200,
     NOTFOUND=404
   }
   ```

   - 编译器默认从0开始赋值
   - 可以赋予某个常量某个整数
   - 允许常量是十六进制整数
   - 末尾没有分号
   - 给常量赋缺省值时，使用常量的全称

5. Service 服务定义类型

   ```java
   service HelloService {
       i32 sayInt(1:i32 param)
       string sayString(1:string param)
       bool sayBoolean(1:bool param)
       void sayVoid()
   }
   ```

6. Namespace (名字空间)

```java
namespace java com.example.test
// 转义后
package com.example.test
```



### 3 . gRPC 与 thrift 框架对比

![image-20211212212828511](/Users/maqun/Documents/fontend-note/docs/media/image-20211212212828511.png)

