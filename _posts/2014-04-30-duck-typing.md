---
layout: post
title: Duck Typing
---

> If it walks like a duck and quacks like a duck, it must be a duck.

# respond_to?
通过respond_to?方法来判断是否有要执行的方法，然后用send(method_name)去执行，而不是用kind_of?
来判断是什么类型。


# 从&:to_s说起

```ruby
[1, 2, 3].map &:to_s  #=> ["1", "2", "3"]
```

初看这段代码会有些让人摸不着头脑，我也一开始以为“&”又是一个什么定义的符号呢，结果它就是引用块时用的那个“&”。

而后面的“:to_s”是一个symbol。

其实上面的代码相当于：

```ruby
[1, 2, 3].map do |x|
  x.to_s
end
```

只是向来喜欢简单的rubyist把它简化到了尽可能没有冗余的地步。

首先在Ruby1.9后，Symbol有to_proc方法:

```ruby
class Symbol
  def to_proc
    proc { |x| x.send self }
  end
end
```

然后“&”会调用对象的to_proc方法，把生成的proc转换成块，然后就
返回了带有一个参数，并且这个参数会调用当前的symbol方法的块，而map的参数也是一个块。

这样就合理了。

这里的“to_proc”就是一个Duck Typing，任何拥有它的对象都可以被“&”调用，而不管它是什么类别。

这是世界大同的节奏啊！不管猫还是狗甚至是老鼠，只要能抓老鼠就好！
