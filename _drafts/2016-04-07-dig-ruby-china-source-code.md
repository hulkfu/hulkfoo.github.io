---
layout: post
title: 研究Ruby-China的代码
---

# 配置文件

### [settingslogic](https://github.com/settingslogic/settingslogic)
用来导入yaml配置文件，如confing/config.yml文件，如：

```ruby
  config.action_controller.asset_host = Setting.upload_url
```

### [rails-settings-cached](https://github.com/huacnlee/rails-settings-cached)
是存动态设置的，能够在运行时修改保存到数据库中，而已有缓存功能。

# 安全

### [rack-attack](https://github.com/kickstarter/rack-attack)
阻止rack攻击，


# 需要修改的地方
因为是想做出一个大众论坛

1. 文本编辑器
2. 头像上传
3. 登录：微信，qq，微博，豆瓣
