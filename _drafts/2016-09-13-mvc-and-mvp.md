---
layout: post
title: MVC和MVP模式
---

MVP算是MVC的演变。MVC中，View可以从Model中读取数据而不用通过Controller。其中P是Presenter，
对Controller的代替。在MVP中，View只和Presenter通信。

回想默认的Rails就是MVC模式，一些大的逻辑比如增删改查都是View通过Router到Controller，
然后执行相应的操作的。但是有些小的数据，比如所有消息的个数，就直接在View写的代码：

```rb
<%= Messages.count %>
```

确实View有时承载了太多的逻辑业务，所以会需要helper的帮忙。在MVP中，View就只是负责显示。

上面如果是MVP，那么这个count就需要首先在Presenter里从Model里取出来了。

MVP的兴起应该是随着手机App而来的，在传统的Web里，我其实觉得除了嵌入的代码，那些HTML就是View了。
通过CSS来保证各屏幕的适应。

之前写Android也发现，Activity里的除了View，还有监控等。而这些在Web就是一个链接而已，可以很方便的
丢到Controller。

MVP和MVC其实是在考虑相同的信息量放到哪里处理方便的结果。其实在代码时，也经常考虑这个参数从哪个方法
传入比较方便和合理。


# 参考

* http://baike.baidu.com/view/3456444.htm
* http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html
