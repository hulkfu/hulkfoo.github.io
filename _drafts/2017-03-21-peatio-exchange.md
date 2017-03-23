---
layout: post
title: 貔貅交易系统代码分析
permalink: peatio
---

[Peatio](https://github.com/peatio/peatio) 是云币用的后台交易系统，而且文档完善，按照　README 就能部署。

# 跑起来
参考[Ubuntu 指南](https://github.com/peatio/peatio/blob/master/doc/setup-local-ubuntu.md)，在 Ubuntu 上的安装开发环境依赖。

需要注意由于代码有两年没有更新了，亲测不能在 Ruby 2.4.0 和 2.3.0 上使用，还是安装指定的 2.2.1 可以使用。

顺便把数据库从 MySQL 无缝换成了 PostgreSQL。

配置好后就可以起来了。

然后跑测试：

```bash
bundle exec rake db:setup RAILS_ENV=test
bundle exec rspec
```


# 主要外部程序

## [RabbitMQ](https://www.rabbitmq.com/)
Messaging enables software applications to connect and scale. Applications can connect to each other, as components of a larger application, or to user devices and data. Messaging is asynchronous, decoupling applications by separating sending and receiving data.

You may be thinking of data delivery, non-blocking operations or push notifications. Or you want to use publish / subscribe, asynchronous processing, or work queues. All these are patterns, and they form part of messaging.

RabbitMQ is a messaging broker - an intermediary for messaging. It gives your applications a common platform to send and receive messages, and your messages a safe place to live until received.

## [bitcoind](https://en.bitcoin.it/wiki/Bitcoind)
bitcoind is a program that implements the Bitcoin protocol for remote procedure call (RPC) use.

# 主要 Gem

[enumerize](https://github.com/brainspec/enumerize) —— 枚举属性。

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

# Assets


## [flightjs](https://flightjs.github.io/)

Flight 是一个轻量级的，基于组件的 JavaScript 框架，它将行为与 DOM 的节点一一映射。

官方的一个例子：

```js
/* Component definition */

var Inbox = flight.component(inbox);

function inbox() {
  this.doSomething = function() { /* ... */ }
  this.doSomethingElse = function() { /* ... */ }

  // after initializing the component
  this.after('initialize', function() {
    this.on('click', this.doSomething);
    this.on('mouseover', this.doSomethingElse);
  });
}

/* Attach the component to a DOM node */

Inbox.attachTo('#inbox');
```

可见它是基于事件驱动的。

用来在 market 里显示动态的信息，比如买卖，帐号资产等。

- component_data
- component_ui
- component_mixin

# 数据结果

只看数据结果，其实就能分析出程序的大概来，即使不知具体的代码。

# Model

## ActiveYamlBase
使用了 ActiveHash 来存储需要 config 文件，而且还可以像 ActiveRecord。

```ruby
class ActiveYamlBase < ActiveYaml::Base
  field :sort_order, default: 9999

  if Rails.env == 'test'
    set_root_path "#{Rails.root}/spec/fixtures"
  else
    set_root_path "#{Rails.root}/config"
  end

  private

  def <=>(other)
    self.sort_order <=> other.sort_order
  end
end
```

Bank, Currency, DepositChannel, Market, MemberTag 和 WithdrawChannel 都是它的子类。可以在 config 里找到对应的 yml 文件。


## Deposit

# 问题
