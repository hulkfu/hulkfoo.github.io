---
layout: post
title: Rails 设置
---

Rails 里的设置，有这几个级别：

- 系统
- 保密
- 一般
- 用户

# 保密
这里保密的主要是 secret_key_base、邮箱密码、OAuth 密钥等不能传到 git 里，但在运行时又需要使用的。

有两种解决方案，简单的是不存 secret.yml 而存 secret.yml.example 文件，配置新环境时根据后者建立前者。

另一种就是存 secret.yml 了，只不过里面的重要的东西用 <%= ENV["xxxx_secret"] %> 来获取，而这个环境变量是不存在 git 里的。

## ENV 变量
我用的是 [dotenv](https://github.com/bkeepers/dotenv) Gem。

在跟目录里创建个 .evn 文件，写入变量即可了。

但默认是不会覆盖已有的 ENV 的，所以就会导致更新失效，用 Dotenv.overload 文件代替 .env 即可。
