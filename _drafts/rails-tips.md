---
layout: post
title: Rails提示
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
