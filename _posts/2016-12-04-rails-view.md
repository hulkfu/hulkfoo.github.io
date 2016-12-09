---
layout: post
title: Rails View
---
View 负责 Model 如何被展示出来。

# View Helpers
在 erb 文件里帮助生成 html。

## Form Helpers

Rails 里有两种方法创建 form：

- form_tag
- form_for

form_for后的参数是一个Rails Modle的实例或其类，这样在block里，使用text_field时会自动生成
相关属性。

form_tag后的参数是一个字符串，一般是在router里定义好的路径，所以不会自动生存，使用text_field_tag
来生成具体的表单输入元素。

form_tag 就是自己指定 form 的 action url，而 form_for 是根据 model 实例自动指定 action url。

有文件需要上传时，需要给 form 加：

```html
enctype="multipart/form-data"

# 或

multipart: true
```

如果需要输入一个 Array，那么它们的name 的最后一个都是 []，这样 Rails 就会知道：

```rb
# 选取多张图片，上传，在 Controller 可以通过 params[:gallery][:image_array] 得到这个数组
<%= file_field_tag "gallery[image_array][]", type: :file, multiple: true %>
```

### 在Rails Form里使用嵌套属性

在模型里，用accepts_nested_attributes_for声明要嵌入的其它模型，比如has_many的。

在controller里params里permit相应的属性，注意，需要包含id，否则只会插入而不会更新。

用build在new里动态生成属性。

## 自定义 helper

Rails中，同名实例变量数据在controller、view和helper 间访问到的数据是同一个，
因为 controller 里的实例变量会被复制到 view里，helper是include的文件。

在Rails 4之前，helpers只被包含在和它同名的controller和view里，比如：

BooksHelper只在BooksController和/view/books/* 里能够使用。

但是从Rails 4开始，每一个controller都包含所有的helper，如果想像从前那样，可以配置：

```ruby
config.action_controller.include_all_helpers = false
```

# JavaScript

JavaScript 在 Rails 里有两种：

- 一般的
  - 在 assets/javascripts 目录里定义
  - 在浏览器里生成

- Unobtrusive
  - 在 views 里定义，其实它就相当于 view，只不过 render 的是可执行的 js 代码
  - 在 Server 里生成
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

## escape_javascript helper
eacape，漏过，这里就是让 js 代码漏过来，按照下面这个 hash：

```
JS_ESCAPE_MAP	=	{ '\\' => '\\\\', '</' => '<\/', "\r\n" => '\n', "\n" => '\n', "\r" => '\n', '"' => '\\"', "'" => "\\'" }
```

就是把字符串里出现的 key，替换成 value，比如 '</'，变成 '<\/'，就是加上 \ 来转义，这样才能被
js 读进来，否则就是一个 html 标签啊。

escape_javascript，简写 j, 像这样用：

```js
# view/items/create_item.js.erb
item = "<%=j render "item" %>"
$("#items").append(item)
```

把 render 的 html 代码，赋值给 js 里的变量，外面还得加 “”，因为它就只是 render 而已。

有了 生成的 html 代码，就可以 apprend 或 after 等操作了。

## Ajax
虽然 Unobtrusive 写异步很方便，但它有一个局限：请求是浏览器用户的点击触发的。
那么如果一个文本编辑器想实现拖拽上传呢，那就需要被动监听编辑器的事件了，然后在
事件的回调函数里执行 post，上传文件。

即如何触发到 Controller？在事件回调里用 Ajax 发送 post 请求。

下面是发送 Ajax 请求：

```js
form = new FormData;
form.append("key", key);
form.append("Content-Type", file.type);
form.append("file", file);
xhr = new XMLHttpRequest;
// 定义方式
xhr.open("POST", host, true);
// 放入 CSRF
xhr.setRequestHeader('X-CSRF-Token', $('meta[name="csrf-token"]').attr('content'));

// 进度条回调
xhr.upload.onprogress = function(event) {
  var progress;
  progress = event.loaded / event.total * 100;
  return attachment.setUploadProgress(progress);
};

// 执行成功，Server 返回了 JSON 格式的数据
xhr.onload = function() {
  var href, url;
  if (xhr.status === 200) {
    // 如果定义了xhr.responseType = 'json'，这里就可以直接用了
    url = href = JSON.parse(xhr.response).url;
    console.log(url)
    return attachment.setAttributes({
      url: url,
      href: href
    });
  }
};

// 开始执行请求
xhr.send(form);
```

Ajax 的 JQuery 的方式：

```js
$.ajax({
  method: "POST",
  url: url_path,
  beforeSend: function(xhr) {xhr.setRequestHeader('X-CSRF-Token', $('meta[name="csrf-token"]').attr('content'))},
  data: { name: "Jack" }
});
```


虽然还没有看 Unobtrusive 的源码，我觉得它的实现就是 handle 了过滤后的 Server 发来的 js，然后执行。

# 参考

- http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html
- http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html
- https://www.youtube.com/watch?v=a61yKxi3pL0
- http://mixandgo.com/blog/the-beginner-s-guide-to-rails-helpers
- http://railscasts.com/episodes/196-nested-model-form-part-1
- http://guides.rubyonrails.org/working_with_javascript_in_rails.html
- https://github.com/rails/rails/issues/1370#issuecomment-42947111
- http://stackoverflow.com/questions/14250678/rails-file-field-tag-not-uploading-file
- http://copytime.org/posts/200
