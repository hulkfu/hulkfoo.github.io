---
layout: post
title: Rails View Helper
---

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
