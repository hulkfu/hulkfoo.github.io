---
layout: post
title: Rails里的ActiveRecord
---

ActiveRecord是  提出的。

# ActiveModel
ActiveModel是从ActiveRecord分离出来的，主要负责模型相关，而后者主要负责数据库等其它工作。

# 一些参数

class_name: 对应的类名
foreign_key: 关联使用的外键，默认是 #{class_name}_id。
primary_key: 指定属性名，比如select where primary_key = id
