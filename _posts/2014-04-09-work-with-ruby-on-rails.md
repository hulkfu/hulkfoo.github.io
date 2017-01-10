---
layout: post
title: 在Ruby on Rails上工作
---

对我来说，Rails如今已不仅仅是做网站的平台，甚至成为了工作的首选。因为它提供给了一个快速开发的工作环境。

对于十分简单的任务，脚本就能搞定，可对于复杂些的任务，如要xml解析、数据库存储、分析然后生成报表，就上Rails吧。它能够更好的帮助你管理工程，如：进行包管理、数据库处理、测试环境等。

**rails console** 十分好用，可以方便的测试自己的想法，与数据进行交互式沟通。

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

记住：在 task 前用 desc 定义说明，否则在 Rails 的 "rake -T" 不显示。

## 参数
向任务传递参数。

### 环境变量ENV

```ruby
desc "Say what."
task :say do
  puts ENV['what']
end

rake say what="Hello World!" #=> Hello World
```

注意这里的参数的 **=** 两边不能有空格。

### 方法参数

```ruby
desc "Add two thing."
task :add, [:one, :two] do |t, args|
  result = args[:one].to_i + args[:two].to_i
  puts "#{args[:one]} #{t} #{args[:two]} equal #{result}"
end

rake add[1,2]  #=> 1 add 2 equal 3
```

上面参数要用放括号包起来，而且参数间用逗号隔开，不能有空格。

可以看出，t, args其实是对前面的映射。t是对task name，args是对name以后的，以hash存储。

# Thor
上面的Rake处理参数确实有些麻烦，于是有了[thor](http://whatisthor.com/)。

一个一般的例子 thor.rb：

```ruby
require 'thor'

class MyCLI < Thor
  desc "hello NAME", "say hello to NAME"
  def hello(name)
    puts "Hello #{name}"
  end
end

MyCLI.start(ARGV)
```

然后执行：

```
ruby thor.rb hello Aston  #=> Hello Aston
```

直接方法就是与命令行沟通的方法。

Rails里的例子，参考[RailsCast](http://railscasts.com/episodes/242-thor)：

在lib/tasks目录下新建setup.thor，确实Rails3以后已经内置支持thor了。

```ruby
class Setup < Thor
  desc "config [NAME]", "copy configuration files"
  method_options :force => :boolean
  def config(name = "*")
    Dir["config/examples/#{name}"].each do |source|
      destination = "config/#{File.basename(source)}"
      FileUtils.rm(destination) if options[:force] && File.exist?(destination)
      if File.exist?(destination)
        puts "Skipping #{destination} because it already exists"
      else
        puts "Generating #{destination}"
        FileUtils.cp(source, destination)
      end
    end
  end

  desc "populate", "generate records"
  method_options :count => 10
  def populate
    # 载入Rails的执行环境
    require File.expand_path('config/environment.rb')
    options[:count].times do |num|
      puts "Generating article #{num}"
      Article.create!(:name => "Article #{num}")
    end
  end
end

```

然后在命令行里用thor来执行，如：

```
thor setup:config --force
```

# 测试
测试真的是必不可少的，能省去自己不少时间。这是时间不是写代码的时间，而是功能出错后找bug及再处理的时间。

能写出测试，说明对代码的意图和结果有明确的认识。测试通过了，心中就有底了。

## 反面教材
用Nokogiri来提取xml文件中的数据，以为在子节点的node调用xpath('//node')即表示在这个子节点找匹配的node，参考[xpath语法][3]，发现要在当前节点，就需要用xpath('.//node')。就是少了那个点，让本来是O(n)的算法复杂度变成了O(n*n)。

这个如果写了测试程序，哪怕检测一下结果个数，肯定能知道自己写错代码了。


[1]: http://archives.ryandaigle.com/articles/2007/6/22/using-command-line-parameters-w-rake-and-capistrano
[2]: http://viget.com/extend/protip-passing-parameters-to-your-rake-tasks
[3]: http://www.w3school.com.cn/xpath/xpath_syntax.asp
