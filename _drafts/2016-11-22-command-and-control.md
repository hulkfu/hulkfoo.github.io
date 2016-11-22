---
layout: post
title: C&C
---
Command and Control，很有用，思路也很清晰。服务器发送命令，客户端执行。

关键是服务器和客户端间的信道如何建立。

最容易想到的就是client打开一个监听端口，监听server的连接请求，建立个TCP连接。
但是这样会被防火墙封掉，不实用。

所有有了反向连接，client去主动上线，连接server。Server可以是打开端口的监听，
也可以是一个可以得到命令的服务，比如邮箱、Twitter等。
