---
layout: post
title:  Redis源码分析－slowlog慢日志
date:   2019-02-26 10:45:00 +0800
tags: [Redis, 缓存, 源码分析]
---

Redis会把处理比较耗时的命令存入慢日志以便我们查找分析系统问题，系统默认的是将执行超过10ms的命令存入到慢日志列表中，最多不超过128条，如果超过了条数限制那么进行滚动删除，保留最近的128条慢日志。

## 性能影响

> The slow log is accumulated in memory, so no file is written with information about the slow command executions. This makes the slow log remarkably fast at the point that you can enable the logging of all the commands (setting the slowlog-log-slower-than config parameter to zero) with minor performance hit.

官方文档都说了，所有慢日志都记录在内存里面的，不会写文件，所以记录慢日志对Redis的性能影响是很小的，几乎可以忽略。即使将`slowlog-log-slower-than`设置为0，也即是把所有执行的命令都记录到慢日志列表里也不会产生太大的影响。

## 配置选项

* `slowlog-max-len`：最多保留多少条记录，默认值128
* `slowlog-log-slower-than`：默认值10000，单位微妙（即10毫秒）

因为所有慢日志都保存在内存里，所以**slowlog-max-len**不要设置得太大；

如果**slowlog-log-slower-than**设置为负，表示不启用慢日志功能；

## slowlog命令

[https://redis.io/commands/slowlog](https://redis.io/commands/slowlog) 

慢日志相关3个命令：
* 当前慢日志记录条数：`slowlog len`
* 获取最近N条慢日志：`slowlog get N`
* 删除所有慢日志记录：`slowlog reset`

## 全局数据

redis.h中**redisServer**结构中包含4个与慢日志有关的字段：

* unsigned long `slowlog_max_len`：服务器配置 slowlog-max-len 选项的值
* long long `slowlog_log_slower_than`：服务器配置 slowlog-log-slower-than 选项的值
* list *`slowlog`：保存了所有慢查询日志的链表
* long long `slowlog_entry_id`：下一条慢查询日志的 ID

slowlog类型是list*，是一个双端链表，链表元素的数据类型为slowlogEntry，当慢日志列表已经达到条数上限的时候，每生成一条慢日志slowlogEntry，就会在slowlog这个双端链表的前端插入这条慢日志，并同时删掉后端最后一条慢日志。

slowlog_entry_id是一个全局的ID，每条慢日志都具有一个从slowlog_entry_id获取的全局唯一ID，每生成一条慢日志slowlog_entry_id就会自动加一。

Redis每条命令执行完之后都会调用**slowlogPushEntryIfNeeded()**这个方法，看看源码：

{% highlight c %}
void slowlogPushEntryIfNeeded(robj **argv, int argc, long long duration) {
    // 慢查询功能未开启，直接返回
    if (server.slowlog_log_slower_than < 0) return; 

    // 如果执行时间超过服务器设置的上限，那么将命令添加到慢查询日志
    if (duration >= server.slowlog_log_slower_than)
        // 新日志添加到链表表头
        listAddNodeHead(server.slowlog,slowlogCreateEntry(argv,argc,duration));

    // 如果日志数量过多，那么进行删除
    while (listLength(server.slowlog) > server.slowlog_max_len)
        listDelNode(server.slowlog,listLast(server.slowlog));
}
{% endhighlight %}


## slowlogEntry

{% highlight c %}
typedef struct slowlogEntry {
    // 命令与命令参数
    robj **argv;
    // 命令与命令参数的数量
    int argc;
    // 唯一标识符
    long long id;  
    // 执行命令消耗的时间，以微秒为单位
    long long duration; 
    // 命令执行时的时间，格式为 UNIX 时间戳
    time_t time; 
} slowlogEntry;
{% endhighlight %}

生成slowlogEntry需要注意的是有可能会截断数据，分两种情况：
- 当命令与命令参数的数量超过**SLOWLOG_ENTRY_MAX_ARGC**(32)时，产生截断
- 当每个参数的长度超过**SLOWLOG_ENTRY_MAX_STRING**(128)时，产生截断
