---
layout: post
title: Ruby元编程
---

又看了一遍[《Ruby元编程》][1]，这次终有所悟。

> 根本没有什么元编程，只有编程而已。

下面总结一些知识和法术。

# 作用域
变量作用域跟Java、C++等区别很大，没有“内部作用域”的概念，因此无法在内部作用域中看到外部作用域。

在Ruby中，存在三个作用域门：

* class
* module
* def

一旦进入新的门，那么之前的变量就被隔离。看下面代码：

```ruby
class A
  var = "out"
  def test
    puts var
  end
end

A.new.test
```
上面代码将报*undefined local variable or method `var'*的错误，说明在方法定义内部不能访问外部的变量。因为var在class A的作用域里，而test方法中var在def test的作用域里。

而这在Java中简直不能容忍，主要原因是Ruby是动态语言，变量不需要定义就可以使用，而如果没有这种隔离，变量就很容易乱套。在Java中，外面定义一个变量，如果在内部没有定义同名的就是访问的外部的，如果定义了同名的就覆盖。而在Ruby中，没有变量定义这一环节。

那么如果想要使用外部变量呢？

## 扁平作用域

不使用门，这样就不会进入新的作用域了。如下面代码，使用**define_method**方法来定义方法，这样var与test方法同在class A的作用域里，test方法就能访问var了。这种法术就叫做**扁平作用域**。

```ruby
class A
  var = "out"
  define_method :test do
    puts var
  end
end

A.new.test
```
对应三个门，有三个扁平化的方法。

* class: Class.new()
* module: Module.new()
* def: Module#define_method()

这样就可以随心选择是否共享作用域了。

# Block、Lambda和Proc

# Eigenclass



学习时顺便建了一个学习代码库：[ruby-study][2]，想法是让代码说话，知识点都写到注释里。下一步继续整理，并能够生成注释文档。

待续。。。

[1]: http://www.amazon.cn/Ruby%E5%85%83%E7%BC%96%E7%A8%8B-Paolo-Perrotta/dp/B0073APSCK/ref=sr_1_1?ie=UTF8&qid=1392710077&sr=8-1&keywords=ruby%E5%85%83%E7%BC%96%E7%A8%8B
    "《Ruby元编程》"
[2]: https://github.com/astonfu/ruby-study
    "ruby study"
