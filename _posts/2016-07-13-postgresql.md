---
layout: post
title: Postgresql
permalink: postgresql
---

# 经常的使用

```sh
sudo su - postgres
psql

# 第一次修改postgres的密码
\password postgres

# 为应用创建角色和密码
create role myapp with createdb login password 'password';

# 回复数据库
psql exampledb < exampledb.sql

\q
```

# 控制台命令

```sh
\h：查看SQL命令的解释，比如\h select。
\?：查看psql命令列表。
\l：列出所有数据库。
\c [database_name]：连接其他数据库。
\d：列出当前数据库的所有表格。
\d [table_name]：列出某一张表格的结构。
\du：列出所有用户。
\e：打开文本编辑器。
\conninfo：列出当前数据库和连接的信息。
```

# 权限

md5: 密码方式
peer

# 问题
gem install pg error:

```
sudo apt-get install libpq-dev
```

# 参考

* http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html
* https://www.digitalocean.com/community/tutorials/how-to-setup-ruby-on-rails-with-postgres
