---
layout: post
title:  Redis源码分析－zmalloc内存分配
date:   2019-02-26 09:45:00 +0800
tags: [Redis, 缓存, 源码分析]
---

Redis进行内存分配使用的是zmalloc，对应源文件为：`zmallo.h`，`zmalloc.c`。

> zmalloc - total amount of allocated memory aware version of malloc()

zmalloc的作用是对内存分配进行一些统计，使我们能清楚知道现在分配了多少总内存。

## 两个宏

- USE_TCMALLOC
- USE_JEMALLOC

如果定义了USE_TCMALLOC，那么zmalloc()底层使用的是tc_malloc()；

如果定义了USE_JEMALLOC，那么zmalloc()底层使用的是je_malloc()；

否则，zmalloc()使用glibc提供的malloc()。

[what're the differences between tcmalloc/jemalloc and memory pool](https://stackoverflow.com/questions/9866145/whatre-the-differences-between-tcmalloc-jemalloc-and-memory-pool)

## 内存分配

zmalloc分配的内存由两部分组成：
- 实际大小部分
- 内容数据部分

zmalloc使用底层的malloc分配完内存之后，把内存大小存储在最开始的4个字节里面，返回的指针部分是指向的内容数据部分。我们使用通过zmalloc分配到的内存与直接用malloc分配的内存是一样的，**实际大小部分**是由zmalloc去处理的。

看下源代码：

{% highlight c %}
void *zmalloc(size_t size) {
    void *ptr = malloc(size+PREFIX_SIZE);//把实际大小部分占据的空间也要加进去
    if (!ptr) zmalloc_oom_handler(size); //如果内存不足，调用OOM Handler
    
    *((size_t*)ptr) = size; //设置实际大小
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);//更新总内存使用数据
    return (char*)ptr+PREFIX_SIZE;//返回指向内容数据部分的指针
}
{% endhighlight %}

## 统计功能

zmalloc在malloc的基础上做了一些内存使用的统计功能：
* 每一块分配内存的前4个字节记录这块内存的实际大小
* 定义全局变量：static size_t `used_memory` = 0

为了线程安全的修改全局变量used_memory，引入了两个变量：
* pthread_mutex_t `used_memory_mutex`：线程互斥锁
* int `zmalloc_thread_safe`：是否开启线程安全

{% highlight c %}
#ifdef HAVE_ATOMIC
#define update_zmalloc_stat_add(__n) __sync_add_and_fetch(&used_memory, (__n))
#else
#define update_zmalloc_stat_add(__n) do { \
    pthread_mutex_lock(&used_memory_mutex); \
    used_memory += (__n); \
    pthread_mutex_unlock(&used_memory_mutex); \
} while(0)
#endif
{% endhighlight %}

这里如果编译器支持`HAVE_ATOMIC`的话，直接使用__sync_add_and_fetch()，否则，使用互斥锁更新内存使用量。

## OOM处理器

Redis提供了一个OOM处理器：**zmalloc_oom_handler**

当调用malloc()返回NULL，也即是内存不足时，输出错误日志并退出。

{% highlight c %}
static void zmalloc_default_oom(size_t size) {
    fprintf(stderr, "zmalloc: Out of memory trying to allocate %zu bytes\n",
        size);
    fflush(stderr);
    abort();
}
{% endhighlight %}