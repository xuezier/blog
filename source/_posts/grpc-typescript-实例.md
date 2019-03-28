---
title: grpc typescript 实例
date: 2018-06-26 15:47:55
tags:
  - typescript
  - grpc
---
![](http://cdn-public.imxuezi.com/grpc.png)
## 前言
gRPC  是一个高性能、开源和通用的 RPC 框架，面向移动和 HTTP/2 设计。目前提供 C、Java 和 Go 语言版本，分别是：grpc, grpc-java, grpc-go. 其中 C 版本支持 C, C++, Node.js, Python, Ruby, Objective-C, PHP 和 C# 支持.

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。

正如其他 RPC 系统，gRPC 基于如下思想：定义一个服务， 指定其可以被远程调用的方法及其参数和返回类型。gRPC 默认使用 protocol buffers 作为接口定义语言，来描述服务接口和有效载荷消息结构。

## 包装
最近转向 typescript 做 node 开发， 基于 express 的 mvc 服务框架涉及到与其他语言服务进程通信，于是就把之前用 javascript 写的 grpc 包装重新用 typescript 写了一遍。

grpc-ts 分为两个包，一个服务端用的，一个客户端用的, 点击可跳转到 GitHub:
* [grpc-server-ts](https://github.com/xuezier/grpc-server-ts)
* [grpc-client-ts](https://github.com/xuezier/grpc-client-ts)

## 使用
这里是配合基于 express 的 typescript-mvc 框架 *mvc-ts* 使用的。
* [mvc-ts](https://github.com/xuezier/ts-mvc)

### 定义统一的 proto 文件
hello.proto
```protobuf
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.grpc.service.test";
option objc_class_prefix ="RTG";

package hello;

service Hello {
  rpc say(stream Empty) returns (stream Word) {};
}

message Empty {

}

message Word {
  string word = 1;
}
```

### 服务端使用
定义service
helloService.ts
```ts
import { Route, Service } from "grpc-server-ts";

@Service('path_to_hello.proto')
export class HelloService {
  @Route
  public async say() {
    return 'hello world';
  }
}
```
开启服务
```ts
import { RpcRegistry, Settings } from 'grpc-server-ts';

import 'helloService.ts';

@Settings(settings)
class RPC extends RpcRegistry { }
RPC.start();
```
#### settings 参数
```js
{
  port: "3333",                     // listen port
  host: "localhost",                // listen host
  ca: "runtime/rpc/ca.crt",         // ca file path
  cert: "runtime/rpc/server.crt",   // cert file path
  key: "runtime/rpc/server.key"     // key file path
}
```
服务端的 grpc 就启动在 localhost:3333

### 客户端使用
定义client, Hello.ts
```ts
import { Route, Client } from 'grpc-client-ts';

@Client(__dirname + '/protobuf/Hello.proto')
export class Hello {
  @Route
  public async say(data, result) { }
}
```
创建一个controller，HelloController.ts
```ts
import * as Express from 'express';

import { Inject, RestController, Get, Res } from "mvc-ts";
import { Hello } from 'Hello.ts';

@RestController('/example')
export class HelloController {
  @Inject()
  private helloRpc: Hello;

  @Get('/hello')
  public async indexAction(@Res() res: Express.Response) {
    let result = await this.helloRpc.say({});
    res.json(result);
  }
}
```

开启服务, index.ts
```ts
import { ApplicationLoader, ApplicationSettings, Inject, ConfigContainer } from 'mvc-ts';

import 'HelloController.ts';

import 'Hello.ts';

@ApplicationSettings({ rootDir: `${__dirname}/../` })
class Application extends ApplicationLoader { }
const Application = new CpApplication();
Application.start(5566);

@Settings(settings)
class ClientRpc extends RpcClientRegistry { }
ClientRpc.start();
```
## settings 参数
```js
{
  port: "3333",                            // server side grpc port
  host: "localhost",                       // server side grpc host
  ca: "runtime/rpc/ca.crt",                // ca file path
  client_cert: "runtime/rpc/server.crt",   // client_cert file path
  client_key: "runtime/rpc/server.key"     // client_key file path
}
```

运行代码
```
ts-node index.ts
```
ts-node 我是用的 4.x 的版本

打开浏览器访问 *http://localhost:5566/example/hello* 获得结果
```json
{
  "word": "hello world"
}
```
![](http://cdn-public.imxuezi.com/grpc-result.png)