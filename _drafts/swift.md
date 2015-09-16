---
layout: post
title: Swift编程语言
---

一种新的语言出来总是让人兴奋，想看看有什么新的功能。毕竟语言也是有科技含量的，有功能上的高级和低级语言。

既然不喜欢Java，索性也别开发Android了，来看看新的swift吧，哈哈～～～不能不说，我喜欢动态语言，除非必要，真的不想去写静态语言。

是的，作为一个开发者，我想有一个好的开发生态环境，不用担心程序被破解，不用考虑适配各种屏幕和硬件，只想开发出心中的那个应用，分享给大家同时得到应有的收获。

# extension
很惊奇的发现，extension关键字可以将类打开。


Swift更像Java，需要编译，但是代码是运行在虚拟机上的。


# 协议
Java中的接口，只定义行为，实习多继承。


# 调用Objective-C

* 创建一个.h的头文件，包含所有要开放给Swift的函数声明。
* 在build settings里设置。在项目—TARGETS，找到Swift Compiler - Code Generation这一项，这里有一项，Objective-C Bridging Header，在其值的地方，填入头文件信息即可。

# 使用[cocoapods](https://cocoapods.org/)
cocoapods是用Ruby写的，看使用方法就是bundle的路子。

用到的pod是用OC写的，就用上面的方法桥接一下。

```
pod install
```

后会提醒去项目工程里，在build setting里，在ohter flag里多加一项**$(inherited)**，因为它覆盖了Pods里的。


# 感想
很开心，Swift要开源了。我仿佛看到了它的辉煌，所以一定要学习。

苹果是尊重开发者的，因为它的创始人就是俩个geek。

通过App的开发，我能把自己的创意分享给大家，很是开心！

有了开发cocos2d-x游戏的经验，何惧用代码写界面？这是个一劳永逸的事情。