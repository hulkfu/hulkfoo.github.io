---
layout: post
title: 貔貅交易系统代码分析
permalink: peatio
---

[Peatio](https://github.com/peatio/peatio) 是云币用的后台交易系统，而且文档完善，按照　README 就能部署。

# 运行开发环境
参考[Ubuntu 指南](https://github.com/peatio/peatio/blob/master/doc/setup-local-ubuntu.md)，在 Ubuntu 上的运行开发环境。

需要注意由于代码有两年没有更新了，不能在 Ruby 2.4.0 上跑，需要更换 Ruby 版本。

把数据库从 MySQL 无缝换成了 PostgreSQL。

其它都能正常安装，然后跑起来，可是到进入进入市场，即 /markets/btccny，Server 就挂掉。


# 主要外部程序



# 主要 Gem

[gon](https://github.com/gazay/gon) — get your Rails variables in your js.


# 数据结果


只看数据结果，其实就能分析出程序的大概来，即使不知具体的代码。

# Model



# 问题
