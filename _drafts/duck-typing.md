---
layout: post
title: Duck Typing
---


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

然后“&”会调用对象的to_proc方法，返回了带有一个参数，并且这个参数会调用当前的symbol方法的块，而map的参数也是一个块。

这样就合理了。
