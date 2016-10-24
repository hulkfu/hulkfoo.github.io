---
layout: post
title: Assets Pipeline
---

assets pipeline，翻译过来就是“资源管道”，处理app/assets里的文件。

在Production环境里，被sprockets处理后放入public/assets里，变成了静态文件。

其主要功能：


- 使用高级语言编写静态资源，如sass写css，coffeescrit写js。
- 将所以js合成一个压缩文件，css也合成一个压缩文件

# 参考
* http://guides.rubyonrails.org/asset_pipeline.html
