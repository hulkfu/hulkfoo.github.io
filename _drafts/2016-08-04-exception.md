---
layout: post
title: 异常处理
---


尽可能少的在可能出现问题的地方，把那几行代码用异常处理包起来，比如插入数据库。


# 事务 —— Transaction
Rails中的事务回滚，会被里面的任意异常触发，除了ActiveRecord::Rollback

* https://ruby-china.org/topics/25427
