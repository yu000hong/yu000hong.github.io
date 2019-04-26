---
layout: post
title:  Shell字符串处理
date:   2019-02-01 16:53:00 +0800
tags: [Shell]
---

官方文档：[Advanced Bash-Scripting Guide: Manipulating Strings](https://www.tldp.org/LDP/abs/html/string-manipulation.html)

官方文档：[Bash Reference Manual](https://www.gnu.org/software/bash/manual/html_node/index.html)


语法 | 说明
-- | --
`${#string}` | 计算字符串长度
`${string:position}` | 截取子字符串
`${string:position:length}` | 截取子字符串
`${string#substring}` | 从前往后删除最短匹配的子字符串
`${string##substring}` | 从前往后删除最长匹配的子字符串
`${string%substring}` | 从后往钱删除最短匹配的子字符串
`${string%%substring}` | 从后往钱删除最长匹配的子字符串
`${string/substring/replacement}` | 替换第一个匹配的子字符串
`${string//substring/replacement}` | 替换所有匹配的子字符串
`${string/#substring/replacement}` | 如果子字符串匹配开头部分(startsWith)，则替换
`${string/%substring/replacement}` | 如果子字符串匹配末尾部分(endsWith)，则替换

### 字符串长度

