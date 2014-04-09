---
layout: post
title: 在Ruby on Rails上工作
---

对我来说，Rails如今已不仅仅是做网站的平台，甚至成为了工作的首选。因为它提供给了一个快速开发的工作环境。

对于十分简单的任务，脚本就能搞定，可对于复杂些的任务，如要xml解析、数据库存储、分析然后生成报表，就上Rails吧。它能够更好的帮助你管理工程，如：进行包管理、数据库处理、测试环境等。

**rails console**十分好用，可以方便的测试自己的想法，与数据进行交互式沟通。

正如DHH所说，Rails不是给新手用的。可是坚持使用，就变成高手了，就是这么相悖。初学者觉得复杂，因为这里面都是最佳实践，有时候为了以后会绕一下。

# Model
有了它，利用Active Record的，只要设计好数据库的表结构，各种便利方法就随之而来。

用**rails generate model**可以方便生成类、数据迁移。

```ruby
$ rails generate model
Usage:
  rails generate model NAME [field[:type][:index] field[:type][:index]] [options]
```

# Rake
Ruby Make，用来处理依赖关系。在Rails中如数据库迁移、测试等都是Rake任务。

通过依赖**:environment**就可以引用自己程序里Rails的东西了。这里的:environment也是一个任务，它会准备task运行的环境。

## 参数
向任务传递参数。

### 环境变量ENV

```ruby
task :say do
  puts ENV['what']
end

rake say what="Hello World!" #=> Hello World
```

注意这里的参数的**=**两边不能有空格。

### 方法参数

```ruby
task :add, [:one, :two] do |t, args|
  result = args[:one].to_i + args[:two].to_i
  puts "#{args[:one]} #{t} #{args[:two]} equal #{result}"
end

rake add[1,2]  #=> 1 add 2 equal 3
```

上面参数要用放括号包起来，而且参数间用逗号隔开，不能有空格。

可以看出，t, args其实是对前面的映射。t是对task name，args是对name以后的，以hash存储。


[1]: http://archives.ryandaigle.com/articles/2007/6/22/using-command-line-parameters-w-rake-and-capistrano
[2]: http://viget.com/extend/protip-passing-parameters-to-your-rake-tasks
