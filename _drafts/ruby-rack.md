---
layout: post
title: Ruby Rack
---

rack在英文里是“行李架”的意思。而它是在Ruby里，是Web服务器（如Thin、WEBrick）和上层应用（如Rails、Sinatra）之间的接口，它对底层的HTTP进行了接口定义，包括：request、response、cookies和sessions。

一个Rack应用就是一个对访问做应答的对象。它只接收环境一个参数，然后返回三个值：status、headers和body。

# 使用

```ruby
# my_rack_app.rb

require 'rack'

app = Proc.new do |env|
  ['200', {'Content-Type' => 'text/html'}, ['A barebones rack app.']]
end

Rack::Handler::WEBrick.run app
```

执行执行即可，或者使用rackup命令配合.ru配置文件执行。

```ruby
# config.ru

run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['get rack\'d']] }
```

执行： rackup config.ru


从上面的例子可以看到Rack的功能，Rails正是使用它来使用HTTP，相当于Rails的底层，而正好也是Thin、Puma等的上层。

Rails对返回的内容进行生成，借助于MVC框架、erb模板等生成Rack所需要的三个返回值。

Puma对HTTP的请求及返回进行处理。

可见Rack是夹在Rails和Puma间的。


# 中间件

Rack::Builder implements a small DSL to iteratively construct Rack applications.

这时AOP(Aspect Oriented Programming，面向切面编程)的思想，按照Rack提供的接口规范，来插入自己的代码，像回调函数的filter一样。

[1]: http://m.onkey.org/ruby-on-rack-1-hello-rack
[2]: http://m.onkey.org/ruby-on-rack-2-the-builder
[3]: http://railscasts.com/episodes/151-rack-middleware
