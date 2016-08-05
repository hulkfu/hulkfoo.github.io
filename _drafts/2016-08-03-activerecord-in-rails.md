---
layout: post
title: Rails里的ActiveRecord
---

ActiveRecord是  提出的。


# 一些参数

class_name: 对应的类名
foreign_key: 关联使用的外键，默认是 #{class_name}_id。
primary_key: 指定属性名，比如select where primary_key = id

## has_many VS. has_one
它们实现的区别在于
