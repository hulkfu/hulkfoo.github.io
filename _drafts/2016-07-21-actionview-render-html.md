---
layout: post
title: ActionView如何生成HTML

# render
Rails 5里，ActionView::Render定义在 ** action_view/render/render.rb ** 里。


# local var

# partial render
partial render的功能是：

1. 自定义layout
2. 自动展开collection

如果不需要上述功能，可以直接用render。

partial没有local变量，需要的话要用locals来传递，比如:

```ruby
<%= render partial: "account", locals: { account: @buyer } %>
```

但可以传一个object变量，它的名字就是partial的名字，比如上面的代码等于：

```ruby
<%= render partial: "account", object: @buyer %>
