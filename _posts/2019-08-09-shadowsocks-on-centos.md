---
layout: post
title: CentOS 上安装 Shadowsocks 客户端
date:   2019-08-09 21:00:00 +0800
tags: [Shadowsocks]
---

开发中使用到 Facebook Accountkit，需要翻墙才能调用相关接口。在网上搜索了一下，在 CentOS 服务器上安装 Shadowsocks 客户端基本上都是使用`pip`进行安装，大致过程如下：

{% highlight shell %}
[yu000hong@ecs]$ pip install shadowsocks
[yu000hong@ecs]$ vim /etc/shadowsocks.json
[yu000hong@ecs]$ cat /etc/shadowsocks.json
{
  "server": "your-shadowsocks-server",
  "server_port": 10080,
  "method": "chacha20-ietf-poly1305",
  "password": "54sJGD8sDUSa",
  "local_address": "127.0.0.1",
  "local_port": 1080
}
[yu000hong@ecs]$ nohup sslocal -c /etc/shadowsocks.json &
{% endhighlight %}

通过上面安装配置运行后，报错了：

```
ERROR method chacha20-ietf-poly1305 not supported
```

搜了一大圈，这个问题最终都没有解决掉！

## go-shadowsocks2

最后到[shadowsocks github网站](https://github.com/shadowsocks)找到了shadowsocks的golang版本：[go-shadowsocks2](https://github.com/shadowsocks/go-shadowsocks2)(Next-generation Shadowsocks in Go)

### 安装方法

1. 使用`go get`安装

```go get -u -v github.com/shadowsocks/go-shadowsocks2```

`go get`安装存在一个问题，就是很多golang包在国内是无法下载下来的。

2. 使用`go mod` & `goproxy.io`代理

```
export GO111MODULE=on
export GOPROXY=https://goproxy.io

git clone https://github.com/shadowsocks/go-shadowsocks2.git
cd go-shadowsocks2
go mod init github.com/shadowsocks/go-shadowsocks2
go build
```

### 配置使用

```
nohup shadowsocks2 -c 'ss://AEAD_CHACHA20_POLY1305:54sJGD8sDUSa@your-shadowsocks-server：10080' -verbose -socks 127.0.0.1:1080 &
```

这样我们就启动了shadowsocks2客户端！


## curl使用shadowsocks代理

curl 通过`-x`参数来设置参数，如下方式就可以使用我们刚配置好的shadowsocks了~

```
curl -x socks5h://localhost:1080 'https://www.google.com/'
```

## Java使用shadowsocks代理

如果使用`HttpURLConnection`来访问的话，那么可以设置SOCKS类型的`Proxy`，代码如下：

{% highlight java %}
URL url = new URL("https://www.google.com/");
Proxy proxy = new Proxy(Proxy.Type.SOCKS, new InetSocketAddress("localhost", 1080));
HttpURLConnection connection = (HttpURLConnection) url.openConnection(proxy);
connection.setRequestMethod("GET");
connection.connect();
InputStream input = connection.getInputStream();
List<String> lines = IOUtils.readLines(input, StandardCharsets.UTF_8);
{% endhighlight %}


