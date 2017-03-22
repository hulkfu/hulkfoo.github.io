---
layout: post
title: 貔貅交易系统代码分析
permalink: peatio
---

[Peatio](https://github.com/peatio/peatio) 是云币用的后台交易系统，而且文档完善，按照　README 就能部署。

# 运行开发环境
参考[Ubuntu 指南](https://github.com/peatio/peatio/blob/master/doc/setup-local-ubuntu.md)，在 Ubuntu 上的运行开发环境。

需要注意由于代码有两年没有更新了，不能在 Ruby 2.4.0 上跑，需要更换 Ruby 版本。

把数据库从 MySQL 无缝换成了 PostgreSQL。

其它都能正常安装，然后跑起来，可是到进入进入市场，即 /markets/btccny，Server 就挂掉。


# 主要外部程序

## [RabbitMQ](https://www.rabbitmq.com/)
Messaging enables software applications to connect and scale. Applications can connect to each other, as components of a larger application, or to user devices and data. Messaging is asynchronous, decoupling applications by separating sending and receiving data.

You may be thinking of data delivery, non-blocking operations or push notifications. Or you want to use publish / subscribe, asynchronous processing, or work queues. All these are patterns, and they form part of messaging.

RabbitMQ is a messaging broker - an intermediary for messaging. It gives your applications a common platform to send and receive messages, and your messages a safe place to live until received.

## [bitcoind](https://en.bitcoin.it/wiki/Bitcoind)
bitcoind is a program that implements the Bitcoin protocol for remote procedure call (RPC) use.

# 主要 Gem

[gon](https://github.com/gazay/gon) —— get your Rails variables in your js.

[daemons-rails](https://github.com/mirasrael/daemons-rails) —— 管理后台进程。

[rotp](https://github.com/mdp/rotp) —— The Ruby One Time Password Library.

[aasm](https://github.com/aasm/aasm) —— 状态机。

[bunny](https://github.com/ruby-amqp/bunny) ——  a popular, easy to use, mature Ruby client for RabbitMQ.

[amqp](https://github.com/ruby-amqp/amqp) —— the asynchronous Ruby RabbitMQ client.


[eventmachine](https://github.com/eventmachine/eventmachine) —— EventMachine is an event-driven I/O and lightweight concurrency library for Ruby.

[em-websocket](https://github.com/igrigorik/em-websocket) —— EventMachine based, async, Ruby WebSocket server.

[datagrid](https://github.com/bogdan/datagrid) —— Gem to create tables grids with sortable columns and filters.

[act-as-taggable-on](https://github.com/mbleigh/acts-as-taggable-on)

[paranoia](https://github.com/rubysherpas/paranoia) —— 假删除数据。

[active_hash](https://github.com/zilkey/active_hash) —— A readonly ActiveRecord-esque base class that lets you use a hash, a Yaml file or a custom file as the datasource.


[liability-proof](https://github.com/peatio/liability-proof) —— A ruby implementation of Greg Maxwell's Merkle approach to prove Bitcoin liabilities.


[phonelib](https://github.com/daddyz/phonelib) —— Ruby gem for phone validation and formatting using google libphonenumber library data.

[unread](https://github.com/ledermann/unread) —— Handle unread records and mark them as read with Ruby on Rails.


# 数据结果

只看数据结果，其实就能分析出程序的大概来，即使不知具体的代码。

# Model



# 问题
