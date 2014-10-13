---
layout: post
title: Rails 拾遗
---

controller与view中的实例变量，及以@开头的变量是相通的。

# Rails runner

runner可以用来在Rails的环境下运行不需要交互的代码。它算是简单的Rake任务。

```ruby
$ bin/rails runner "Model.long_running_method"
```

通过 -e 能够选择运行的环境。

```ruby
$ bin/rails runner -e staging "Model.long_running_method"
```

# view render

## 为什么在view里面使用render传递变量时需要在locals里？

因为这样就可以一层一层的嵌套，而不必担心名字冲突。

当然，在render partial里，默认有一个与其名字相同的变量（去掉下划线），可以通过key为:object的hash传递。

## render与render partial的区别？

可到[rails/actionview/lib/action_view/helpers/rendering_helper.rb](https://github.com/rails/rails/blob/master/actionview/lib/action_view/helpers/rendering_helper.rb)看render的定义。

首先都是render方法，只是后者传了key为**:partial**的hash。partial只不过是render方法的参数options hash里的一个item。

下面是activeview代码里的注释原话：

```
# rails/rails/actionview/lib/action_view/renderer/partial_renderer.rb
  # If you're not going to be using any of the options like collections or layouts, you can also use the short-hand
  # defaults of render to render partials.
```

# rails form

主要理解参数的命名规则，从而得到了array和hash。

* 上层相同则为一个hash里的，根据后面的来分辨：有内容则又为一个hash，无则是数组。