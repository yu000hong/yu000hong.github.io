---
layout: post
title:  理解Unsafe
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



## 参考资料

[聊聊高并发（十八）理解AtomicXXX.lazySet方法](https://blog.csdn.net/ITer_ZC/article/details/40744485)