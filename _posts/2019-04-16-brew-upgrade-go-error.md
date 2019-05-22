---
layout: post
title: 国内GO资源安装失败的解决办法
date: 2019-04-16 16:13:00 +0800
tags: [GO, VPN]
---

由于各种原因，国内使用`brew upgrade go`进行go版本升级或者使用`go get`安装golang官方包都可能会失败，即使使用代理都不行，碰到这样的问题真的很恼火。
通过Google，目前发现了三种解决办法。

<!-- more -->

### 方法一

其实 golang 在 github 上建立了一个镜像库，如 **https://github.com/golang/net** 即是 **https://golang.org/x/net** 的镜像库。

获取 `golang.org/x/net` 包，其实只需要以下步骤：

{% highlight shell %}
mkdir -p $GOPATH/src/golang.org/x
cd $GOPATH/src/golang.org/x
git clone https://github.com/golang/net.git
{% endhighlight %}

其它 `golang.org/x` 下的包获取皆可使用该方法。

### 方法二

使用`gopm`代替go进行依赖包的下载。

[gpmgo/gopm](https://github.com/gpmgo/gopm 
https://gopm.io)


> Go Package Manager (gopm) is a package manager and build tool for Go.

**安装gopm**

```
go get -u github.com/gpmgo/gopm
```

**使用gopm**

```
gopm get -g golang.org/x/net
```

### 方法三

配置`git proxy`，这种方式是本人更推荐的方式，因为前面两种方式无法解决`brew upgrade go`失败问题。

**brew upgrade go**报错：

```
yu000hong$ brew upgrade go
==> Upgrading 1 outdated package:
go 1.10.3 -> 1.12
==> Upgrading go
==> Downloading https://dl.google.com/go/go1.12.src.tar.gz
Already downloaded: /Users/yuhong/Library/Caches/Homebrew/downloads/0f64991f71d1a7e34370af429189d0a6757948abf4f26cd18f4bd450f59b34dc--go1.12.src.tar.gz
==> Downloading https://storage.googleapis.com/golang/go1.7.darwin-amd64.tar.gz
Already downloaded: /Users/yuhong/Library/Caches/Homebrew/downloads/ad0901a23a51bac69b65f20bbc8e3fe998bc87a3be91d0859ef27bd1fe537709--go1.7.darwin-amd64.tar.gz
==> ./make.bash --no-clean
==> /usr/local/Cellar/go/1.12/bin/go install -race std
==> Cloning https://go.googlesource.com/tools.git
Cloning into '/Users/yuhong/Library/Caches/Homebrew/go--gotools--git'...
fatal: unable to access 'https://go.googlesource.com/tools.git/': Failed to connect to go.googlesource.com port 443: Operation timed out
Error: An exception occurred within a child process:
  DownloadError: Failed to download resource "go--gotools"
Failure while executing; `git clone --branch release-branch.go1.12 https://go.googlesource.com/tools.git /Users/yuhong/Library/Caches/Homebrew/go--gotools--git` exited with 128. Here's the output:
Cloning into '/Users/yuhong/Library/Caches/Homebrew/go--gotools--git'...
fatal: unable to access 'https://go.googlesource.com/tools.git/': Failed to connect to go.googlesource.com port 443: Operation timed out
```

**配置git proxy**

```
git config http.proxy socks5://127.0.0.1:1086
git config https.proxy socks5://127.0.0.1:1086
```

`127.0.0.1:1086` 这个端口号是Shadowsocks在Mac下的默认端口，可以Shadowsocks偏好设置的高级Tab下设置，如图：

![](/static/image/201904/shadowsocks-setting.png)

但是，这样设置完之后，还是报同样的错误，使用`brew config`看下Homebrew的配置：

```
yu000hong$ brew config
HOMEBREW_VERSION: 2.1.1
ORIGIN: https://github.com/Homebrew/brew
HEAD: b4f73e61649fcfc5aaa779c311e2514619ce01e7
Last commit: 3 days ago
Core tap ORIGIN: https://mirrors.ustc.edu.cn/homebrew-core.git
Core tap HEAD: 37f4aa7c45e87ec774bb9e0de55c7c57bf4aa51a
Core tap last commit: 5 weeks ago
HOMEBREW_PREFIX: /usr/local
HOMEBREW_LOGS: /Users/yuhong/Library/Logs/Homebrew
CPU: quad-core 64-bit broadwell
Homebrew Ruby: 2.3.7 => /usr/local/Homebrew/Library/Homebrew/vendor/portable-ruby/2.3.7/bin/ruby
Clang: 8.0 build 800
Git: 2.20.1 => /usr/local/opt/git/bin/git
Curl: 7.43.0 => /usr/bin/curl
Java: 10.0.2, 1.8.0_181, 1.8.0_71, 1.7.0_79
macOS: 10.11.6-x86_64
CLT: 8.2.0.0.1.1480973914
Xcode: 8.0
```

从输出结果可以看出，Homebrew使用的是自带的git，因此我们同样需要设置其对应的代理：

```
/usr/local/opt/git/bin/git config --system http.proxy socks5://127.0.0.1:1086
/usr/local/opt/git/bin/git config --system https.proxy socks5://127.0.0.1:1086
```

再次运行`brew upgrade go`，没有报错，更新升级成功！

**配置http_proxy&https_proxy**

通过配置git proxy之后，上面的问题解决了，但是使用`go get google.golang.org/grpc`还是会报网络超时异常。这种情况下，只能设置`http_proxy`和`https_proxy`两个环境变量来解决了。

```bash
$ export http_proxy=socks5://127.0.0.1:1086
$ export https_proxy=socks5://127.0.0.1:1086
```

添加上面两句到`.bash_profile`，然后**source**一下，就可以开心地使用**go get**了～


