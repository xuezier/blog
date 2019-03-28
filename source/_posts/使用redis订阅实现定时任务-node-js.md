---
title: 使用redis订阅实现定时任务(node.js)
date: 2016-11-29 11:37:56
tags:
  - redis
  - javascript
  - node.js
---
![](http://cdn-public.imxuezi.com/redis.png)

## 条件
安装的redis版本至少为2.8.0，查看redis版本
```bash
> redis-cli --version
redis-cli 3.2.5
> redis-server --version
Redis server v=3.2.5 sha=00000000:0 malloc=libc bits=64 build=112b103d6428f815
```
只有2.8.0以上的版本支持键事件订阅

## 开启键失效事件
```
redis事件：
K     Keyspace events, published with __keyspace@<db>__ prefix.
E     Keyevent events, published with __keyevent@<db>__ prefix.
g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
$     String commands
l     List commands
s     Set commands
h     Hash commands
z     Sorted set commands
x     Expired events (events generated every time a key expires)
e     Evicted events (events generated when a key is evicted for maxmemory)
A     Alias for g$lshzxe, so that the "AKE" string means all the events.
```

redis默认是关闭键事件订阅，可以在bash输入以下命令开启，因为我们只要订阅键失效的事件，所以只需要Ex就够了：
```bash
> redis-cli config set notify-keyspace-events Ex
OK
```

## 订阅事件
在第一个终端窗口(windows是控制台cmd)，打开redis-cli，进行以下操作
```bash
> redis-cli
> PSUBSCRIBE __keyevent@1__:expired
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "__keyevent@1__:expired"
3) (integer) 1
```

__keyevent代表订阅的是键事件，1是redis的db，expired表示订阅的是键失效的事件

如果要订阅所有数据库的键，可以改成：
```bash
> PSUBSCRIBE __keyevent@*__:expired
```

## 触发订阅
在新的一个终端或控制台中，打开redis-cli，创建一个键，并让其失效：
```bash
> redis-cli
> select 1
OK
> set mykey 'hello'
OK
> expire mykey 10
(integer) 1
```

10秒后mykey过期会触发终端1的事件订阅，终端1会输出
```bash
1) "pmessage"
2) "__keyevent@1__:expired"
3) "__keyevent@1__:expired"
4) "mykey"
```

所以我们只要制定好键的过期时间，去触发键订阅事件，利用这个特性，可以实现定时任务

## node.js代码
用到第三方模块redis(npm install redis)
```js
'use strict';
let redis = require('redis');
let child_process = requrie('child_process');
const keyClient = redis.createClient({db: 1});
const pubClient = redis.createClient();
child_process.execSync('redis-cli config set notify-keyspace-events Ex'); // 确保键事件订阅开启
pubClient.psubscribe('__keyevent@1__:expired');
keyClient.set('mykey', 'hello', ()=>{
  keyClient.pexpireat('mykey', +new Date('2016/12/23 10:00:00'));
});
let futureFun = () =>{
  console.log('hello world');
};
pubClient.on('pmessage', (channel, listen, key)=>{
  key == 'mykey' && futureFun();
});
```
futureFun会在2016/12/23 10点，mykey这个键过期的时候执行。
