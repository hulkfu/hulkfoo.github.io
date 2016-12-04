---
layout: post
title: Rails View
---

# Form

Rails 里有两种方法创建 form：

- form_tag
- form_for

form_tag 就是自己指定 form 的 action url，而 form_for 是根据 model 实例自动指定 action url。

有文件需要上传时，需要给 form 加：

```html
enctype="multipart/form-data"

# 或

multipart: true
```

## 在Rails Form里使用嵌套属性

在模型里，用accepts_nested_attributes_for声明要嵌入的其它模型，比如has_many的。

在controller里params里permit相应的属性，注意，需要包含id，否则只会插入而不会更新。

用build在new里动态生成属性。

# JavaScript

JavaScript 在 Rails 里有两种：

- 一般的
  - 在 assets/javascripts 目录里定义

- Unobtrusive
  - 在 views 里定义，其实它就相当于 view，只不过 render 的是可执行的 js 代码
  - 所以在它里能执行 render 方法，也能访问 Controller 里定义的 @变量
  - <% %> 里的 Ruby 代码，和外面的 JS 代码是不共享变量的

以上两种 JS，获取变量的方式也不同。一般的是定义在 html 标签 attr 里属性，比如 data-type 来
获取 type 这个变量。而 Unobtrusive 是通过 Controller 传来的 @实例变量。一个是 JS 代码，
一个是 Ruby 代码，两个世界也是不通的。但是 Ruby 是嵌入到 JS 里，所以可以把 Ruby 里值赋给 JS，
反过来就不行了。

在一般 JS 里，是不能 render views 里的 view 的，因为一般 JS 在 assets 里，里面存的是静态的
文件，发布是会被编译成一个大文件的，是不经常变的。而 render 的 view 一般都是根据变量而不停变的。

那么如何在一般 JS 里，也能 render view 呢？一般这些 view 是固定的，只是为了分离。

我觉得，可以直接把这些 view 写到或读到 Assets 里。但觉得没有那么大的意义，Rails 禁用了 render，
就是想你按照它提供的默认模式，把带数据的 JS 代码写在 View 里，作为 View 的延伸，需要是用 Unobtrusive 的
方法在 Server 里生成，然后返回。


# 参考

- http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html
- http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html
- https://www.youtube.com/watch?v=a61yKxi3pL0
- http://railscasts.com/episodes/196-nested-model-form-part-1
- http://guides.rubyonrails.org/working_with_javascript_in_rails.html
- https://github.com/rails/rails/issues/1370#issuecomment-42947111
- http://stackoverflow.com/questions/14250678/rails-file-field-tag-not-uploading-file
