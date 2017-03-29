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

在 lib/daemons/ 目录下以 _ctl 结尾的文件都是控制后台的控制文件，对应的 daemon_name.rb 为其执行文件，比如 k 线的分别是 k_ctl 和 k.rb，这都是用 daemons-rails 生成的：

```bash
rails generate daemon <name>
```

可以在 config 目录下为每个 daemon 建立配置文件，格式是 name-daemon.yml，否则就是默认的 daemons.yml 文件。可以在里面指明执行文件，否则就是：

```ruby
@options[:script] ||= File.join(root_path, daemons_dir, "#{app_name}.rb")
```

从 lib/deamons/deamons 文件可以看出整体控制就是调用就是遍历所以 ctl 文件，然后把第一个参数赋值过去：

```ruby
#!/usr/bin/env ruby
results = []
Dir[File.dirname(__FILE__) + "/*_ctl"].each {|f| results << `ruby #{f} #{ARGV.first}`}
results.delete_if { |result| result.nil? || result.empty? }
puts results.join unless results.empty?
```

所以可以有：

```bash
# start all daemons
bundle exec rake daemons:start

# or start daemon one by one
bundle exec rake daemon:matching:start
...

# Daemon trade_executor can be run concurrently, e.g. below
# line will start four trade executors, each with its own logfile.
# Default to 1.
TRADE_EXECUTOR=4 rake daemon:trade_executor:start

# You can do the same when you start all daemons:
TRADE_EXECUTOR=4 rake daemon:start
```



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


[eco](https://github.com/sstephenson/eco) —— Embedded CoffeeScript templates.在 app/assets/javascripts/templates 中使用，配合 JST，提前编译 js 模板文件。

[jwt](https://github.com/jwt/ruby-jwt) —— A pure ruby implementation of the [RFC 7519](https://tools.ietf.org/html/rfc7519) OAuth JSON Web Token (JWT) standard. JWT是一种用于双方之间传递安全信息的简洁的、URL安全的表述性声明规范。因为数字签名的存在，这些信息是可信的，JWT可以使用HMAC算法或者是RSA的公私秘钥对进行签名。

# 数据结构

只看数据结构，其实就能分析出程序的大概来，即使不知具体的代码。

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

# Assets

## [highcharts](http://www.highcharts.com/)

用户 order 栈图

用其 [highstock](http://www.highcharts.com/stock/demo) 模块来画 K 线图等技术指标。

在 app/assets/javascripts/highcharts/technical_indicators.js 有对币价及时指标的计算函数，比如 MACD 啦。

## [flightjs](https://flightjs.github.io/)

Flight 是一个轻量级的，基于组件的事件驱动型 JavaScript 框架。

Flight 强制严格的组件分离。等你创建一个元件时，你并不能从外部控制它。这样设计，一个元件不能被其它元件影响，也不是全局树上的属性。元件间不能相互直接操作，而是通过广播它们的动作时间来通知订阅的元件，然后这个订阅的元件执行被动的动作。

事件是开发的。当一个元件触发一个时间时，它并不知道它的请求会会谁处理。这样就实现了功能上的去耦合，让工程师独立的看待每一个元件，而不用考虑应用增长的复杂性。

通过构造符合 DOM 节点事件协议的事件，我们可以得到下面好处：

- 免费的事件传播
- 一个元件可以订阅 document 层面的事件，也可以定义指定 DOM 节点的事件
- 自定义的事件和 DOM 节点的事件是一样被处理的


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

为了逻辑方便，一般把元件分别放到 data、ui 和 mixin 里。

## market
我们看到的市场 show 正是用 flight 完成的，在 market 里显示动态显示各种信息，比如 k线图、买单卖单、帐号资产等等，被分别放到了下面三个目录里：

- component_data
- component_ui
- component_mixin

# 问题
