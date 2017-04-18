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

trust: 只要知道数据库用户名就不需要密码或ident就能登录。
password: 明文密码方式，一般用md5.
md5: md5密码方式。
peer: 当前Linux的用户名，比如默认的postgres用户，需要切换到postgres用户去访问设置数据库。

# auto start

```
sudo update-rc.d postgresql enable
```

# 安装

## Mac
直接下载[Postgres.app](http://postgresapp.com/)，拉到应用目录里就可以使用。

然后在.bash_profile里配置PATH:

```
export PATH=$PATH:/Applications/Postgres.app/Contents/Versions/latest/bin
```

Postgres.app默认是trust认证，所以只需要有用户名和host就可以连接。

当然也可以用brew或直接安装，只是Mac一般开发用，没有必要一直开着数据库服务。

## Linux

之前需要进行数据迁移，并卸载旧的 Postgresql，比如：

```bash
sudo apt-get purge postgresql-9.1
```

否则新旧都会运行，而且新的端口号会加一，默认是 5432,新的就成了 5433 了。

# 问题
gem install pg error:

```
sudo apt-get install libpq-dev
```

# 参考

* http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html
* https://www.digitalocean.com/community/tutorials/how-to-setup-ruby-on-rails-with-postgres
