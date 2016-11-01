---
layout: post
title: Rails View Helper
---

# form_for VS. form_tag

form_for后的参数是一个Rails Modle的实例或其类，这样在block里，使用text_field时会自动生成
相关属性。

form_tag后的参数是一个字符串，一般是在router里定义好的路径，所以不会自动生存，使用text_field_tag
来生成具体的表单输入元素。


# 自定义helper

Rails中，同名实例变量数据在controller、view和helper间是同一个，
因为他们都是当前controller的实例变量，helper是include的文件，view只是controller的模板文件，
实现上像是block。

在Rails 4之前，helpers只被包含在和它同名的controller和view里，比如：

BooksHelper只在BooksController和/view/books/* 里能够使用。

但是从Rails 4开始，每一个controller都包含所有的helper，如果想像从前那样，可以配置：

```ruby
config.action_controller.include_all_helpers = false
 ```


# 参考
* http://mixandgo.com/blog/the-beginner-s-guide-to-rails-helpers
