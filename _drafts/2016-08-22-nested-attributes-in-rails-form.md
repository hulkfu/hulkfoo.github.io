---
layout: post
title: 在Rails Form里使用嵌套属性
---

在模型里，用accepts_nested_attributes_for声明要嵌入的其它模型，比如has_many的。

在controller里params里permit相应的属性，注意，需要包含id，否则只会插入而不会更新。

用build在new里动态生成属性。


# 参考
* http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html
* https://www.youtube.com/watch?v=a61yKxi3pL0
* http://railscasts.com/episodes/196-nested-model-form-part-1
