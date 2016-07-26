---
layout: post
title: 如何写一个代理服务器
---

在研究[shadowsockets](http://githut.com/shadowsockets/shadowsockets)的代码，很规整，
注释也很清晰，然后准备仿一个Ruby版的出来，当是练手。

SOCKS不是Socket，前者是Socket Secure的简称，是一个代理协议，而socket是通信链的句柄。它们不是
一个层面上的概念。参考OSI七成模型的话，SOCKS工作在第五层会话层（Session layer），而Socket工作
在第四层传输层（Transport layer），可以说SOCKS是基于Socket的，所以叫Socket Secure，多了安全。
