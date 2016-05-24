---
layout: post
title: Ruby元编程
permalink: meta
---

又看了一遍[《Ruby元编程》][1]，这次终有所悟。

> 根本没有什么元编程，只有编程而已。


为什么在Ruby中元编程是常态呢？因为变量作用域。因为Ruby中变量是有在相同的层里才能访问，而不想Java那样里面的可以访问外面的。

下面总结一些知识和法术。

# 动态方法
使用动态方法，可以调用没有在代码中明确定义的方法。实现上有两种方式：

* 主动：在调用前就定义了方法，一般在初始化的时候。
* 被动：在调用时才去定义方法，通过覆写method_missing()方法来实现。

主动的方式会对respond_to?()有反应，被动的还需要另实现。但被动的更灵活，因为很多时候并不是有数据源的。

下面是会用到的法术。

## 动态调用方法
对一个实例调用send方法，第一个参数是要调用的方法名，后面的是参数。

通过Object#send()，调用的方法便成为了参数，这样就可以动态决定调用什么方法了。

```ruby
class A
  def echo(str)
    puts str
  end
end

a = A.new
a.echo "hi"         #=> hi
a.send :echo, "hi"  #=> hi
```

##动态创建方法
使用Module#define_method()方法定义，下文还会讲它的扁平作用域。

```ruby
define_method :echo do |arg|
  puts arg
end

echo "hi"  #=> hi
```

## method_missing()
当调用一个不存在的方法时，会触发method_missing()方法。可以通过覆写它来调用实际上并不存在的方法。

在覆写时，要注意何时触发super方法，也就是报错。

```ruby
class B
  def method_missing(method, *args)
    puts "called #{method}(#{args.join(',')})"
    puts "with a block" if block_given?
  end
end

B.new.test(1, 2) { } #=> called test(1,2)
                     #=> with a block
```

# 作用域
变量作用域跟Java、C++等区别很大，没有“内部作用域”的概念，因此无法在内部作用域中看到外部作用域。

在Ruby中，存在三个作用域门：

* class
* module
* def

一旦进入新的门，那么之前的变量就被隔离。

也就是说：

* class里def外的就是class作用域，在里面定义的@变量就是类的变量。
* 而用def则打开了实例方法的门，它定义的方法就是实例方法，在def里定义的@变量就是实例的变量。
* 如果需要定义类方法，就需要打开类，方法名前加self或class<<self。因为def已经是实例方法了。
其实类的变量或方法是定义在类的隐性父类eigenclass里的，类是其但实例。

看下面代码：

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
这么做是有道理的，很常见的情况：在外面定义一个类，然后在方法内调用。

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

用lambda()创建的Proc称为lambda，其它方式创建的叫proc。他们的区别主要是：

* return关键字

在lambda中，return仅是从这个lambda中返回，而proc中会从定义proc的作用域中返回。

```ruby
def double(callable)
  callable.call * 2
end

l = lambda { return 10 }
puts double(l)  # => 20

p = proc { return 10 }
puts double(p)  # 此时会报 unexpected return (LocalJumpError)的错误
```

* 参数检验

lambda的适应能力比proc差。如果调用lambda的参数量不对，会失败，抛出一个ArgumentError错误;而proc会忽略多余的参数，不足则补为nil。

# 类定义
类似当前对象**self**，也存在当前类的概念。当定义一个方法时，该方法将成为当前类的一个方法。每当用**class**打开一个类时，这个类就成为了当前类。

对应的打开现有类的扁平化方法Module#class_eval()，它能在一个已存在类的上下文中执行一个块：

```ruby
def add_hello_to_class(klass)

  klass.class_eval do
    # 实例方法
    def hello
      puts "hello"
    end

    # 类方法
    def self.hi
      puts "hi"
    end
  end
end

add_hello_to_class String
"s".hello  #=> hello
String.hi  #=> hi
```

这里顺便提一下Object#instance_eval()方法，顾名思义，它是打开对象，所有它只会修改当前对象self，然后来执行代码。

class_eval()只有类能够调用，但instance_eval()类和对象都能调用，然后前者里生成了类方法，后者里生成了单件方法。

## 类实例变量
Ruby解释器假定所有的实例变量都属于当前对象self。所以要牢记：

> 类也是对象，而且需要自己在程序中追踪**self**。

```ruby
class C
  @var = 2
  def self.read
    puts @var
  end

  def write(var)
    @var = var
  end

  def read
    puts @var
  end
end

c = C.new
c.write 3
c.read  #=> 3
C.read  #=> 2
```

上面的例子中，定义了两个同名@var的变量，一个在C充当self的时刻，所以是类的实例变量；另一个在对象充当self的时刻，所以是对象的实例变量。它们互不影响。

## 类变量
上面的类的实例变量，不能被继承或被其实例访问，而类变量可以，一个变量前加上 **@@** 就是了。

把上面的例子中的@var都换为@@var，那么结果就是：

```ruby
c.read  #=> 3
C.read  #=> 3
```

## Eigenclass
一个看不见的类，像JavaScript的原型类。通过它，可以为单个实例创建方法。

每个eigenclass只有一个实例，并且不能被继承。但是

>eigenclass的超类是超类的eigenclass。

因此，可以在子类中调用超类的类方法。方法路径是子类 -> 子类eigenclass -> 超类eigenclass。

通过下面的语法进入eigenclass

```ruby
class << an_object

end
```

## 类扩展汇入
在Ruby中，一般用include来扩展类，而include方法会将模块的方法扩展成类的实例方法，extend方法会将模块的方法扩展成类的类方法。那如何在用include时也扩展类方法呢？

答案是用钩子。勾住module的included方法。

```ruby
module M
  def self.included(base)
    base.extend(ClassMethods)
  end

  module ClassMethods
    def m1
    end
  end
```

Rails里的[ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)，不仅替我们写了上面的过程，还很好的整理了module依赖关系。

```ruby
module M
  extend ActiveSupport::Concern

  module ClassMethods
    def m1
    end
  end
```

模块混入Concern后，会自动将ClassMethods里方法扩展成类方法。

关于依赖关系，看下面代码：

```ruby
module Foo
  def self.included(base)
    base.class_eval do
      def self.method_injected_by_foo
        ...
      end
    end
  end
end

module Bar
  def self.included(base)
    base.method_injected_by_foo
  end
end

class Host
  include Foo # We need to include this dependency for Bar
  include Bar # Bar is the module that Host really needs
end
```

Foo定义了一个Host的类方法method_injected_by_foo，即Host.method_injected_by_foo，而在
includeBar时需要调用这个方法。因此需要首先inclued Foo。

而我们实际只需要提供的信息是include Bar，然后由Bar去负责其依赖的module。

```ruby
module Bar
  include Foo
  def self.included(base)
    base.method_injected_by_foo
  end
end

class Host
  include Bar
end
```

但是上面代码行不通，因为Bar include Foo，那么method_injected_by_foo便定义给了Bar，而不是我们期望的Host::method_injected_by_foo。

通过使用 ActiveSupport::Concern便可实现上面的想法：

```ruby
require 'active_support/concern'

module Foo
  extend ActiveSupport::Concern
  included do
    def self.method_injected_by_foo
      ...
    end
  end
end

module Bar
  extend ActiveSupport::Concern
  include Foo

  included do
    self.method_injected_by_foo
  end
end

class Host
  include Bar # works, Bar takes care now of its dependencies
end
```

在Concern里，included块里就成了base的块，即base.class_eval的块。

Concern的具体实现可参考[源码](https://github.com/rails/rails/blob/master/activesupport/lib/active_support/concern.rb)。

## 类继承
理解了Ruby寻找方法的算法，就能清晰分析出方法的调用的路径。

> “向右一步，再向上。”

先找定义自己的类，没有的话再找其父类，直到尽头BasicObject.

所以当父类的方法被子类覆盖(override)时，就不再调用父类的方法，除非用super显性调用。

被include进来的module，也被包装成**包含类(include class)** ，superclass()找不到它们，但是
它却相当于当前类的父类。include了一个模块，就相当于在当前类和其父类间的继承链中插入了一个类。

所以如果一个类include了几个模块，那么越往后include的被插入的越靠前，即其可以覆盖上面模块的方法，这
也符合情理。


# eval()
在eval眼里，代码只是文本。

与instance_eval()和class_eval()不同，eval()执行的是一个**代码字符串**，而不是块。而instance_eval()和class_eval()也可以执行代码字符串。

## Binding类
Binding的对象是一个完整的作用域，可以通过它来捕捉并带走当前的作用域。可以通过Kernel#binding()方法来创建。

```ruby
class C
  def m
    @val = 7
    binding
  end
end

b = C.new.m
eval "puts @val", b
```

## 安全级别


# 例子
学习时顺便建了一个学习代码库：[ruby-study][2]，想法是让代码说话，知识点都写到注释里。下一步继续整理，并能够生成注释文档。

# 使用感想
在Ruby的使用中，我越来越多的使用元编程了。不是为了酷，是因为它确实能实现更高的抽象，而这不正是编程所要去做的吗？
只需一条指令，剩下的交给程序去处理；再复杂，也只有一个接口。

> "The more you use it, the more you understand it."

在使用的时候，我也会脑补一下C或者Java要如何实现，Ruby确实简单不少，能够很快实现那想法。
但首先要意思到更高层抽象的可能性，这就关乎框架了。

元编程，让代码可以动态的定义。充分利用字符变量，可以成为方法名、代码块、变量名，因为有define_method、eval、instance_variable_get等方法。


[1]: http://www.amazon.cn/gp/product/B0073APSCK/ref=as_li_tf_tl?ie=UTF8&camp=536&creative=3200&creativeASIN=B0073APSCK&linkCode=as2&tag=phun-23
    "《Ruby元编程》"
[2]: https://github.com/astonfu/ruby-study
    "ruby study"
