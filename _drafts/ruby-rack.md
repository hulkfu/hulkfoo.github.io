---
layout: post
title: Ruby Rack
---

Rack是Web服务器（如Thin、WEBrick）和上层应用（如Rails、Sinatra）之间的接口。

A Rack application is an Ruby object (not a class) that responds to call. It takes exactly one argument, the environment and returns an Array of exactly three values: The status, the headers, and the body.

Rack gem is a collection of utilities and facilitating classes, to make life easier for anyone developing Rack applications. It includes basic implementations of request, response, cookies & sessions.

# 运行

# 中间件

Rack::Builder implements a small DSL to iteratively construct Rack applications.

这时AOP(Aspect Oriented Programming，面向切面编程)的思想，按照Rack提供的接口规范，来插入自己的代码，像回调函数的filter一样。

[1]: http://m.onkey.org/ruby-on-rack-1-hello-rack
[2]: http://m.onkey.org/ruby-on-rack-2-the-builder
[3]: http://railscasts.com/episodes/151-rack-middleware
