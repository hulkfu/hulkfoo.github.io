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

# 基础类

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

## Currencible

```ruby
module Currencible
  extend ActiveSupport::Concern

  included do
    extend Enumerize
    enumerize :currency, in: Currency.enumerize, scope: true
    belongs_to_active_hash :currency_obj, class_name: 'Currency', foreign_key: 'currency_value'
    delegate :key_text, to: :currency_obj, prefix: true
  end
end

```

# 验证系统

# Token

```ruby
create_table "tokens", force: true do |t|
  t.string   "token"
  t.datetime "expire_at"
  t.integer  "member_id"
  t.boolean  "is_used",    default: false
  t.string   "type"
  t.datetime "created_at"
  t.datetime "updated_at"
end
```

没有过期也没有用过就可用：

```ruby
scope :available, -> { where("expire_at > ? and is_used = ?", DateTime.now, false) }
```


生成一个 30 分钟后过期的 token：

```ruby
def generate_token
  self.token = SecureRandom.hex(16)
  self.expire_at = 30.minutes.from_now
end
```

# 用户系统

## identity
用户的注册信息，与 Devise 相接，用户的具体信息在与其 member 里。

```ruby
create_table "identities", force: true do |t|
  t.string   "email"
  t.string   "password_digest"
  t.boolean  "is_active"
  t.integer  "retry_count"
  t.boolean  "is_locked"
  t.datetime "locked_at"
  t.datetime "last_verify_at"
  t.datetime "created_at"
  t.datetime "updated_at"
end
```

## Member
会员的信息。

```ruby
create_table "members", force: true do |t|
  t.string   "sn"
  t.string   "display_name"
  t.string   "email"
  t.integer  "identity_id"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.integer  "state"
  t.boolean  "activated"
  t.integer  "country_code"
  t.string   "phone_number"
  t.boolean  "disabled",     default: false
  t.boolean  "api_disabled", default: false
  t.string   "nickname"
end
```


```ruby
has_many :orders
has_many :accounts
has_many :payment_addresses, through: :accounts
has_many :withdraws
has_many :fund_sources
has_many :deposits
has_many :api_tokens
has_many :two_factors
has_many :tickets, foreign_key: 'author_id'
has_many :comments, foreign_key: 'author_id'
has_many :signup_histories

has_one :id_document

has_many :authentications, dependent: :destroy
```

## Account
资金帐号。属于 member， 可以绑定银行卡、BTC 地址等。

```ruby
create_table "accounts", force: true do |t|
  t.integer  "member_id"
  t.integer  "currency"
  t.decimal  "balance",                         precision: 32, scale: 16
  t.decimal  "locked",                          precision: 32, scale: 16
  t.datetime "created_at"
  t.datetime "updated_at"
  t.decimal  "in",                              precision: 32, scale: 16
  t.decimal  "out",                             precision: 32, scale: 16
  t.integer  "default_withdraw_fund_source_id"
end
```

```ruby
belongs_to :member
has_many :payment_addresses
has_many :versions, class_name: "::AccountVersion"
has_many :partial_trees
```

## RunningAccount

```ruby
CATEGORY = {
  withdraw_fee:         0,
  trading_fee:          1,
  register_reward:      2,
  referral_code_reward: 3,
  deposit_reward:       4
}
```

```ruby
create_table "running_accounts", force: true do |t|
  t.integer  "category"
  t.decimal  "income",      precision: 32, scale: 16, default: 0.0, null: false
  t.decimal  "expenses",    precision: 32, scale: 16, default: 0.0, null: false
  t.integer  "currency"
  t.integer  "member_id"
  t.integer  "source_id"
  t.string   "source_type"
  t.string   "note"
  t.datetime "created_at"
  t.datetime "updated_at"
end
```

## IdDocument
身份认证。

```ruby
create_table "id_documents", force: true do |t|
  t.integer  "id_document_type"
  t.string   "name"
  t.string   "id_document_number"
  t.integer  "member_id"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.date     "birth_date"
  t.text     "address"
  t.string   "city"
  t.string   "country"
  t.string   "zipcode"
  t.integer  "id_bill_type"
  t.string   "aasm_state"
end
```


# 资金系统

## Deposit
充值，即保证金。云币是 100% 保证金的。

状态机的状态：

```ruby
STATES = [:submitting, :cancelled, :submitted, :rejected, :accepted, :checked, :warning]

```

```ruby
create_table "deposits", force: true do |t|
  t.integer  "account_id"
  t.integer  "member_id"
  t.integer  "currency"
  t.decimal  "amount",                 precision: 32, scale: 16
  t.decimal  "fee",                    precision: 32, scale: 16
  t.string   "fund_uid"
  t.string   "fund_extra"
  t.string   "txid"
  t.integer  "state"
  t.string   "aasm_state"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.datetime "done_at"
  t.string   "confirmations"
  t.string   "type"
  t.integer  "payment_transaction_id"
  t.integer  "txout"
end
```

## DepositChannel
不同渠道充值的配置，是一个 ActiveYamlBase。

数据在 config/deposit_channels.yml 文件：

```ruby
- id: 200
  key: satoshi
  currency: btc
  sort_order: 1
  min_confirm: 1
  max_confirm: 3
- id: 400
  key: bank
  currency: cny
  sort_order: 2
  bank_accounts:
    -
      bank: 'Your Bank Name'
      branch: 'Your Bank Branch'
      holder: 'Your Account Holder'
      account: 'Your Account Number'

```

## Withdraw
提币。

```ruby
create_table "withdraws", force: true do |t|
  t.string   "sn"
  t.integer  "account_id"
  t.integer  "member_id"
  t.integer  "currency"
  t.decimal  "amount",     precision: 32, scale: 16
  t.decimal  "fee",        precision: 32, scale: 16
  t.string   "fund_uid"
  t.string   "fund_extra"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.datetime "done_at"
  t.string   "txid"
  t.string   "aasm_state"
  t.decimal  "sum",        precision: 32, scale: 16, default: 0.0, null: false
  t.string   "type"
end
```

## WithdrawChannel
不同渠道提现的配置，比如最少提多少，费率等等。

配置文件：

```ruby
- id: 200
  key: satoshi
  currency: btc
  fixed: 8
  fee: 0.0005
  inuse: true
  type: WithdrawChannelSatoshi
- id: 400
  key: bank
  currency: cny
  fixed: 2
  fee_max: 0
  min: 100
  max: 50000
  fee: 0.003
  proportion: true
  inuse: true
  type: WithdrawChannelBank

```

## FundSource
资金来源。

比如当已经用来 ICO 时，就会被冻结。

```ruby
create_table "fund_sources", force: true do |t|
  t.integer  "member_id"
  t.integer  "currency"
  t.string   "extra"
  t.string   "uid"
  t.boolean  "is_locked",  default: false
  t.datetime "created_at"
  t.datetime "updated_at"
  t.datetime "deleted_at"
end
```


## PaymentAddress

```ruby
create_table "payment_addresses", force: true do |t|
  t.integer  "account_id"
  t.string   "address"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.integer  "currency"
end
```

## PaymentTransaction

```ruby
create_table "payment_transactions", force: true do |t|
  t.string   "txid"
  t.decimal  "amount",                   precision: 32, scale: 16
  t.integer  "confirmations"
  t.string   "address"
  t.integer  "state"
  t.string   "aasm_state"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.datetime "receive_at"
  t.datetime "dont_at"
  t.integer  "currency"
  t.string   "type",          limit: 60
  t.integer  "txout"
end
```

## Proof

## PartialTree






# 交易系统

## Ticket
股票代码，即交易的币种，是 BTC 还是 ETH 等等。

```ruby
create_table "tickets", force: true do |t|
  t.string   "title"
  t.text     "content"
  t.string   "aasm_state"
  t.integer  "author_id"
  t.datetime "created_at"
  t.datetime "updated_at"
end
```

```ruby
belongs_to :author, class_name: 'Member', foreign_key: 'author_id'
```

用 AASM 记录状态：

```ruby
aasm whiny_transitions: false do
  state :open
  state :closed

  event :close do
    transitions from: :open, to: :closed
  end

  event :reopen do
    transitions from: :closed, to: :open
  end
end
```

## Market

## Order
买或卖，只是要价，还没有被撮合。

```ruby
create_table "orders", force: true do |t|
  t.integer  "bid"
  t.integer  "ask"
  t.integer  "currency"
  t.decimal  "price",                     precision: 32, scale: 16
  t.decimal  "volume",                    precision: 32, scale: 16
  t.decimal  "origin_volume",             precision: 32, scale: 16
  t.integer  "state"
  t.datetime "done_at"
  t.string   "type",           limit: 8
  t.integer  "member_id"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.string   "sn"
  t.string   "source",         null: false
  t.string   "ord_type",       limit: 10
  t.decimal  "locked",                    precision: 32, scale: 16
  t.decimal  "origin_locked",             precision: 32, scale: 16
  t.decimal  "funds_received",            precision: 32, scale: 16, default: 0.0
  t.integer  "trades_count",                                        default: 0
end
```

order 创建后触发：

```ruby
after_commit :trigger

ATTRIBUTES = %w(id at market kind price state state_text volume origin_volume)

def trigger
  return unless member

  json = Jbuilder.encode do |json|
    json.(self, *ATTRIBUTES)
  end
  member.trigger('order', json)
end
```

Member 的 trigger 方法：

```ruby
def trigger(event, data)
  AMQPQueue.enqueue(:pusher_member, {member_id: id, event: event, data: data})
end
```



### OrderAsk

### OrderBid

## Trade
交易记录。

```ruby
create_table "trades", force: true do |t|
  t.decimal  "price",         precision: 32, scale: 16
  t.decimal  "volume",        precision: 32, scale: 16
  t.integer  "ask_id"
  t.integer  "bid_id"
  t.integer  "trend"
  t.integer  "currency"
  t.datetime "created_at"
  t.datetime "updated_at"
  t.integer  "ask_member_id"
  t.integer  "bid_member_id"
  t.decimal  "funds",         precision: 32, scale: 16
end
```

## matching
撮合系统，是交易所的关键：如何把买单和买单匹配，让它们交易，并生成交易数据。

对，这里才是系统的核心，其它的一切都是为了它服务。

主要在 app/models/matching 目录下。

主要撮合代码在 executor.rb：

```ruby
def create_trade_and_strike_orders
  ActiveRecord::Base.transaction do
    @ask = OrderAsk.lock(true).find(@payload[:ask_id])
    @bid = OrderBid.lock(true).find(@payload[:bid_id])

    raise TradeExecutionError.new({ask: @ask, bid: @bid, price: @price, volume: @volume, funds: @funds}) unless valid?

    @trade = Trade.create!(ask_id: @ask.id, ask_member_id: @ask.member_id,
                           bid_id: @bid.id, bid_member_id: @bid.member_id,
                           price: @price, volume: @volume, funds: @funds,
                           currency: @market.id.to_sym, trend: trend)

    @bid.strike @trade
    @ask.strike @trade
  end

  # TODO: temporary fix, can be removed after pusher -> polling refactoring
  if @trade.ask_member_id == @trade.bid_member_id
    @ask.hold_account.reload.trigger
    @bid.hold_account.reload.trigger
  end
end
```


# API

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

# 技术方案

## 技术指标显示
用 Redis 缓存当前价格，提供数据，用 JS 代码绘制显示。

## 撮合
使用 RabbitMQ

# 问题
