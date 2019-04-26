---
layout: post
title:  Memcached源码研究
date:   2019-02-01 09:44:00 +0800
tags: [Redis]
---

### 编译源码

**How to compile memcached from github sources?**

GitHub has no configure file. Only configure.am. No idea, how to handle it.

---

Memcached use [Autotools](https://en.wikipedia.org/wiki/GNU_Build_System) to build source, as you got `configure.ac` and `Makefile.am`, follow these steps to build the source:

{% highlight shell %}
./autogen.sh
./configure
make
{% endhighlight %}

The binary is generated now, find the location by running `find * -name memcached`.