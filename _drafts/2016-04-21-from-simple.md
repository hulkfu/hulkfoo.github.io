---
layout: post
title: 从最简单开始
---

从最简单开始，一步步搭建，才能知道每一步为何需要这样。

之前用Rails，一下子所有的东西都来了。包括了建网站需要的所有，大而全。但对新手，却让他不知道从哪里开始，
不明白每一个组件的用途是什么，这样设计有什么意义在。有中乎论坛在囫囵吞枣的感觉。

而当我用Python想做一个不用登录就能发邮件的应用时，从低向上开发和思考，我找到了MIME,Flask,Capistrano
等等，这些都是我之前知道熟悉的，可直到需要时，我才真正理解它们用途和用法。

在Rails中一下生成出来的MVC代码，在Flask中都要自己去写。正是这样，我才体会到POST请求与显示的表单是
分离的。表单只不过是个View，填写完后向请求服务地址发送POST而已。也就是说在view里用js验证请求数据是不安全的，
因为可以直接绕过来对action地址发请求。

简单的好处是可以看到核心的东西，而没有那么多外层的迷雾。

Rails的Assets就是一个成熟的方案，参考[大公司的前端在干什么](https://www.zhihu.com/question/20790576).




# 参考
* https://nokyotsu.com/qscripts/2015/05/deploy-a-python-web-application-on-dreamhost-with-pyenv.html
