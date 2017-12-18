---
layout: post
title: Phantomjs
permalink: phantomjs
---

Phantomjs 是一个用 JavaScprit 编写脚本的无界面的 Webkit 浏览器。它原生快速的支持各种 Web 标准：DOM，CSS，JSON，Canvas 和 SVG。

我们知道 Node 也是一个集成了 V8 内核的 js 处理器，可是它是为服务端使用的，而 Phantomjs 是纯模仿浏览器的，可以把它当作一个没有界面的浏览器，如何执行操作需要编写 js 代码。

Phantomjs 它的主要作用是：

- 执行 headless 网站测试
- 抓屏
- 网页自动化操作
- 网络监控


# HEADLESS 测试


PhantomJS itself is not a test framework, it is only used to launch the tests via a suitable test runner.
