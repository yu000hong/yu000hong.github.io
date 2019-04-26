---
layout: post
title:  理解Unsafe这三个方法的区别：putInt & putIntVolatile & putOrderedInt
date:   2019-01-31 17:36:00 +0800
tags: [JAVA, JUC, 并发]
---

先看一下javadoc文档：

**putInt**

{% highlight java %}
{% endhighlight %}

**putOrderedInt**

{% highlight java %}
/***
* Sets the value of the integer field at the specified offset in the
* supplied object to the given value.  This is an ordered or lazy
* version of <code>putIntVolatile(Object,long,int)</code>, which
* doesn't guarantee the immediate visibility of the change to other
* threads.  It is only really useful where the integer field is
* <code>volatile</code>, and is thus expected to change unexpectedly.
*
* @param obj the object containing the field to modify.
* @param offset the offset of the integer field within <code>obj</code>.
* @param value the new value of the field.
* @see #putIntVolatile(Object,long,int)
*/
public native void putOrderedInt(Object obj, long offset, int value);
{% endhighlight %}


## 参考资料

[聊聊高并发（十八）理解AtomicXXX.lazySet方法](https://blog.csdn.net/ITer_ZC/article/details/40744485)