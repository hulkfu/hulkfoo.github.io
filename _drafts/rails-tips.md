---
layout: post
title: Rails 拾遗
---

# 感想
Rails确实很方便，一句scaffold命令就可以生成一个博客应用。
可只有当你不去自动生成，model，controller和view都自己写一遍时，才能体会到它更深的灵活，没有模式，view可以任你合并，不用new非要单独成页；controller里可以实现
更复杂的逻辑；model可以各种scope，before，after。

是的，一个更随心的Rails。

在Rails中，一个功能可以还controller层实现，也可以在model层实现时，就尽量要在model层完成。一是保证controller的精简，而是保证model功能的完善。


#变量传递

controller与view中的实例变量即@开头的变量是相通的。

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

* 通过name来命名。
* 上层相同则为一个hash里的，根据后面的来分辨：有内容则又为一个hash，无则是数组。


# validate
验证输入数据的有效性很重要，它应该在model层完成，然后辅助View层和Database层。

主要有两个方法：validates和validate。前者是一般的验证，第一个参数是属性，然后是要验证的方法；后者是专为只有验证方法而设定。

还有validates_with和validates_each，前者使用自定义的验证类；后者直接在块里定义验证。


* http://guides.rubyonrails.org/active_record_validations.html
