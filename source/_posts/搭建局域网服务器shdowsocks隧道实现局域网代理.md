---
title: 搭建局域网服务器shdowsocks隧道实现局域网代理
date: 2017-02-16 13:38:48
tags:
  - shadowsocks
  - net
---
![](http://cdn-public.imxuezi.com/netssocks.png)

技术公司经常需要通过科学上网去查询英文资料或高速上网，在公司每人配置一个账号又显得浪费。通过搭建局域网服务器，
在局域网服务器上搭建 shadowsocks 隧道代理，让每台电脑通过这台局域网服务器上网，就可以只用一个账号实现公司员工集体翻墙搞事情。

## 条件
* 一个能用的 shadowsocks 服务账号
* 一台内网服务器，我们采用的是 centOS 7


## 安装 shadowsocks 服务
pip是 python 的包管理工具。在本文中将使用 python 版本的 shadowsocks，此版本的 shadowsocks 已发布到 pip 上，因此我们需要通过 pip 命令来安装。

## 安装 pip
在控制台执行以下命令安装 pip：
```bash
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py
```

## 安装配置 shadowsocks
在控制台执行以下命令安装 shadowsocks：
```bash
pip install --upgrade pip
pip install shadowsocks
```

安装完成后，需要创建配置文件/etc/shadowsocks.json，内容如下：
```
{
    "server":"1.1.1.1",             #ss服务器IP
    "server_port":1035,             #ss服务器端口
    "local_address": "127.0.0.1",   #本地ip
    "local_port":1080,              #本地端口
    "password":"password",          #连接ss密码
    "timeout":300,                  #等待超时
    "method":"aes-256-cfb",         #加密方式,可选aes-128-cfb, aes-192-cfb, aes-256-cfb, bf-cfb, cast5-cfb, des-cfb, rc4-md5, chacha20, salsa20, rc4, table
    "fast_open": false,             # true 或 false。如果你的服务器 Linux 内核在3.7+，可以开启 fast_open 以降低延迟。开启方法： echo 3 > /proc/sys/net/ipv4/tcp_fastopen 开启之后，将 fast_open 的配置设置为 true 即可
    "workers": 1                    # 工作线程数
}
```

## 配置自启动
新建启动脚本文件/etc/systemd/system/shadowsocks.service，内容如下：
```
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
```

执行以下命令启动 shadowsocks 服务：
```bash
systemctl enable shadowsocks
systemctl start shadowsocks
```

检查 shadowsocks 服务是否已成功启动
```
systemctl status shadowsocks -l
```

## Convert Shadowsocks into an HTTP proxy
我们要将 socks 服务转为 http 实现 http(s) 代理

### 安装 polipo
执行以下命令安装 polipo 服务：
```bash
wget https://www.irif.fr/~jch/software/files/polipo/polipo-1.1.1.tar.gz
tar -xf  polipo-1.1.1.tar.gz
cd polipo-1.1.1
make all
make install
```

### 配置 polipo 服务
创建配置文件夹
```bash
mkdir   /etc/polipo
```
从源码目录拷贝配置文件：
```bash
cp config.sample  /etc/polipo/config
```
* 可根据需要配置选项

### 后台启动服务
```bash
nohup polipo socksParentProxy=localhost:1080 &
```

## 测试代理
测试挂载的代理是否可以使用
```bash
export https_proxy=http://127.0.0.1:8123
curl https://www.facebook.com
```

若能拉取页面资源，则代理成功

## 局域网使用
局域网内的电脑通过设置网络代理
我的内网服务器内网 ip 是 192.168.1.18

win 10 直接在设置中带开网络设置，添加手动代理，

* 地址 192.168.1.18
* 端口 8123
打开浏览器使用谷歌搜索马上翻墙搞事情
