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
不使用“门”，这样就不会进入新的作用域了。如下面代码，使用**define_method**方法来定义方法，这样var与test方法同在class A的作用域里，test方法就能访问var了。这种法术就叫做**扁平作用域**。

```ruby
class A
  var = "out"
  define_method :test do
    puts var
  end
end

A.new.test  #=> out
```
对应三个“门”，有三个扁平化的方法。

* class: Class.new()
* module: Module.new()
* def: Module#define_method()

这样就可以随心选择是否共享作用域了。

## 常量作用域
常量作用域不同于变量作用域，它有自己的规则。
> 常量像文件系统一样组织成树形结构。其中模块（还有类）像目录，而常量则像文件。

仅仅用来充当常量容器的模块，被称为**命名空间**。在同一命名空间里的常量可以直接引用，否则需要使用常量路径，即*常量::常量*的形式。在常量前仅有::表示根路径，从而可以使用绝对路径。

将上面出错的例子里的var改为Var，即首字母大写，代表常量，那么便可以在“门”内访问了。虽说是常量，也可以进行修改，当然这不是一个好办法。

```ruby
class A
  Var = "out"
  def test
    puts Var
  end
end

A.new.test  #=> out
```
这么做是有道理的，很常见的情况，在外面定义一个类，然后在方法内调用

# Block、Lambda和Proc

## Block
只有在调用一个方法时才可以定义一个块，它会被传递给这个方法，然后在这个方法中可以用**yield**关键字回调这个块。可以通过Kernel#block_given?()方法来判定是否包含块。

```ruby
def m
  return yield if block_given?
  'no block'
end

puts m              #=> no block
puts m { 'block' }  #=> block
```
其实扁平作用域，就是利用块的威力——闭包，即
> 当定义一个块时，它会获取当时环境中的绑定，并且把它传给一个方法时，它会带着这些绑定一起进入该方法。

## Proc对象
块不是对象，想要存储块供以后使用或当参数传递，就需要一个对象，它就是Proc对象，**转化成对象的块**。通过Proc.new方法来创建一个Proc对象，以后就可以用Proc#call()方法来执行这个由块转换而来的对象。

```ruby
dec = Proc.new { |x| x - 1 }
puts dec.call 3  #=> 2
```
先定义，后执行，这就是**延时执行**技术。

## &操作符
在C语言里，&操作符是取地址里内容操作。Ruby里的&操作符也借鉴了这个思想，取出Proc对象里的块。

在Ruby里规定一个参数：

* 在参数列表最后一个。
* 以&符号开头。

那么这个参数就代表传给这个方法的块。

```ruby
def math(a, b)
  yield a, b
end

def teach(what, a, b, &op)
  puts "#{a} #{what} #{b} equal #{math(a, b, &op)}"
end

teach("add", 1, 1) { |x, y| x + y }  #=> 1 add 1 equal 2
teach("mul", 2, 2) { |x, y| x * y }  #=> 2 mul 2 equal 4
```

去掉这个参数的&符号，就得到了一个封装了块的Proc对象。

```ruby
def get_proc(&b)
  b
end

proc = get_proc { |name| puts "hello, #{name}" }
puts proc.class  #=> Proc
proc.call "world"  #=> hello, world
```

## proc与lambda比较
除了Proc.new，Ruby还提供了Proc:lambda()和proc()两个内核方法来创建转换成对象的块。
。。。

# Eigenclass
一个看不见的类。。。

学习时顺便建了一个学习代码库：[ruby-study][2]，想法是让代码说话，知识点都写到注释里。下一步继续整理，并能够生成注释文档。

待续。。。

[1]: http://www.amazon.cn/Ruby%E5%85%83%E7%BC%96%E7%A8%8B-Paolo-Perrotta/dp/B0073APSCK/ref=sr_1_1?ie=UTF8&qid=1392710077&sr=8-1&keywords=ruby%E5%85%83%E7%BC%96%E7%A8%8B
    "《Ruby元编程》"
[2]: https://github.com/astonfu/ruby-study
    "ruby study"
