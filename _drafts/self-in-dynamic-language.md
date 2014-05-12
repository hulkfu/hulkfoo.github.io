---
layout: post
title: 动态语言中的self
---

Ruby隐性的在程序中自己寻找self对象，通过def、class、module这三个门的定义来切换不同的程序运行环境————self。

而Python需要显性的指明self。Python的风格就是明确，一种方法。