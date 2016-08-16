---
layout: post
title: 我是如何在twitter里一次取消成千的following的
---

# 方案

```js
$(".unfollow-text").click()
```

对上面就是代码，

1. 进入 https://twitter.com/following，查看自己关注的人；
2. 一直往下拉，显示所以，期间可以把还想保留关注的先取消关注；
3. 进入chrome的调试模式，在console里，输入上面代码。

然后所有的“取消关注”和“关注”按钮都会被点一遍的，10秒以内吧，之前取关的就被关注了。


# 起因

自己的twitter被黑，关注了特别多的垃圾帐号，让我一下子不想上twitter了。

# 之前的方案

## api接口
有一个[Ruby Twitter Gem](https://github.com/sferik/twitter)，写代码调用取关，
可是每个key每天只能取关几十人就失效了，还要申请，麻烦，效率也不高。

## 屏幕Macro
开始发现可以一个一个点时，觉得费劲，就想写个宏或录制一个，自动下拉，然后点击。也没有找到。
突然想到查看一下取关的类，就有了上面的方案。很满意。


# 总结

虽然懂些锋利JQuery，但作为Rubyist，以为要用each来调用，没有想到直接click()就是对所有元素执行。

当然，JQuery是包装好JS，$选择的返回结果会遍历执行后面的方法，ruby也可以很轻松写出这样的类，把除了
count等集体的方法外的方法send给数组里的每一个成员。

相比使用公开的api，逆向来的更管用！
