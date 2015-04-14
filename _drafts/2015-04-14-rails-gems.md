---
layout: post
title:  Rails 必备 Gems
---


# devise

# cancancan

有一个坑，就是经它验证权限的controller必须有一个与之对应的model，否则会报错NameError，找不到那个model。

看来它是在model层面做限制，而不是controller。