---
layout: post
title:  理解AtomicXXX.lazySet方法
date:   2019-01-31 16:46:00 +0800
tags: [JAVA, JUC, 并发]
---

看过`java.util.concurrent.atomic`包里面各个AtomicXXX类实现的同学应该见过lazySet方法，比如AtomicBoolean类的lazySet方法:

{% highlight java %}
public final void lazySet(boolean newValue) {
    int v = newValue ? 1 : 0;
    unsafe.putOrderedInt(this, valueOffset, v);
}
{% endhighlight %}

从源代码可以看出lazySet是利用`Unsafe.putOrderedInt()`方法实现的，我们看看这个方法的文档：

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