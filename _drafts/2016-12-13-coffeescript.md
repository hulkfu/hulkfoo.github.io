---
layout: post
title: CoffeeScript
permalink: coffeescrit
---
写 CoffeeScript 比起 JavaScript 幸福太多了，没有那么多层层叠叠的括号。

# 语法

下面是官方的概览例子：

```coffee
# Assignment:
number   = 42
opposite = true

# Conditions:
number = -42 if opposite

# Functions:
square = (x) -> x * x

# Arrays:
list = [1, 2, 3, 4, 5]

## Objects:
# 定义对象，里面用 hash 的方式定义，而不能用等号。
math =
  root:   Math.sqrt
  square: square
  cube:   (x) -> x * square x

# Splats:
race = (winner, runners...) ->
  print winner, runners

# Existence:
alert "I knew it!" if elvis?

# Array comprehensions:
cubes = (math.cube num for num in list)
```

## 变量
默认的变量都带 var，被封进了局部里。

@开头的变量，如 Ruby 里，是实例变量的意思。在实现上就是在前面加上 this.。
当然 js 里方法名也是变量。

```coffee
# this.a = 1
@a = 1

# this.m()
@m()
```

## 函数

```coffee
hi = (name) ->
  console.log "Hi" + name
```

翻译成 js：

```js
var hi;

hi = function(name) {
  return console.log("Hi" + name);
};
```

Ruby 里的新的 lambda 语法应该就是跟 CoffeeScript 学的，只不过参数在箭头的后面而不是前面。
应该是考虑到块的习惯 —— { |param| ... }。
松本行宏也在书里表达了对 CoffeeScript 的看好和尊敬。

那为什么 Coffee 要放前面呢，因为它支持这样用：

```coffee
hi = (name) -> console.log "Hi" + name

# 如果放到后面
hi = -> (name) console.log "Hi" + name
```

如果把参数放到箭头后面，分行的话语法是错误的，单行的话会翻译成：

```js
var hi;

hi = function() {
  return name(console.log("Hi" + name));
};
```

这是在对后面执行 name 方法，显然不是我们的本意。

Ruby 为啥可以呢？因为它的代码块是用 {} 包括的，而不是 Coffee 那样“裸露”的。

一个小的差异，其实源于基因上的差异。

### 方法的参数是匿名函数
在到处是回调的 js 里，这很常见，比如 setTimeout():

```coffee
setTimeout ->
  console.log "delay"
, 1000
```
这里的第二个参数前的逗号要于上一层同样的缩进。

按照这个姿势，再多的匿名函数参数也不怕。

翻译出来就是：

```js
setTimeout(function() {
  return console.log("delay");
}, 1000);
```

### 胖箭头 =>
胖箭头会传当前环境上一层的 this 到相应的方法里，代替事件方法里的 this（这个 this 指的相应这个事件的东西，比如按钮）。

在 JS 里，上一层意味着调用这个函数的函数。

比如点击一个按键想执行 setTimeout 方法，而这个是方法是上层 window 的，所以要把它传到 setTimeout 的this 里。

比如将上面的 setTimeout 的匿名函数换成 => , 那么：

```js
setTimeout((function(_this) {
  return function() {
    return console.log("delay");
  };
})(this), 1000);
```

又如：

```coffee
Account = (customer, cart) ->
  @customer = customer
  @cart = cart

  $('.shopping_cart').on 'click', (event) =>
    @customer.purchase @cart
```

生成：

```js
var Account;

Account = function(customer, cart) {
  this.customer = customer;
  this.cart = cart;
  return $('.shopping_cart').on('click', (function(_this) {
    return function(event) {
      return _this.customer.purchase(_this.cart);
    };
  })(this));
};
```
