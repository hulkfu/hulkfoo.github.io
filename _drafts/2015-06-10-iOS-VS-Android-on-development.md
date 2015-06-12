---
layout: post
title: iOS开发 vs. Android开发
---

因为有Android的开发经验，所以总是会拿来比较，这样学习起来会更快速。

它们都是MVC框架的，所以从这三个方面比较。

# Controller： UIViewController vs. Activity
它们都是MVC框架中的controller级别。


iOS中，通过navigationController来统一管理界面显示，在逻辑上，它是一个栈，push即压入栈顶显示其，pop则将其弹出，显示上一个ViewController。

Android中，也维护有界面栈，但是通过startActivity来显示一个新的，然后可以设置这个栈的属性，如SingleInstance、Stand等。

相比，iOS的栈只像Andorid的SingleInstance。

segues

## 传递参数
正规的iOS用代理，Android用Intent。

可都可以直接在新创建的Controller实例中赋值给public变量。

# View： StoryBoard、xib vs. XML

相比iOS的更适合初学者，拖拽，虽然Android也能拖拽，但一般都是直接写XML，在iOS中虽然最后也是生成了XML，但是不能直接编写，还是在代码中写比较靠谱。


# Model

# 工具
XCode比Android的开发工具好用，它能够自动识别图片文件，而Android这些都显示的展现出来了。


# 参考
* http://www.raywenderlich.com/81880/storyboards-tutorial-swift-part-2