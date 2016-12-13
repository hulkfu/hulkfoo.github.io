---
layout: post
title: 动态语言中的self
---

self是主题，是当前运行的环境。

# Ruby
Ruby隐性的在程序中自己寻找self对象，通过def、class、module这三个门的定义来切换不同的程序运行环境————self。

## 赋值

```rb
class People
  attr_accessor :name

  def wrong_replace_name(new_name)
    name = new_name
  end

  def right_replace_name(new_name)
    self.name = new_name
    # or use @name directly
    # @name = new_name
  end


  def print_name
    puts name
  end
end

p = People.new
p.name = "Jack"
p.print_name
p.wrong_replace_name "Lucy"
p.print_name
p.right_replace_name "Lucy"
p.print_name
```

错误的方法即不加self，那么name就会理解成一个临时变量，而我们想要调用的是定义的name=()方法。

# Python
而Python需要显性的指明self。Python的风格就是明确，一种方法。

# JavaScript
JavaScript中的this是调用这个函数的环境，默认是触发事件的，隐性的。


http://www.jimmycuadra.com/posts/self-in-ruby
