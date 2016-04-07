---
layout: post
title: 部署 Ruby China
---

把Ruby-China跑到自己的服务器上了。

服务器是Ubuntu 12.04,大致是按照README上步骤安装，只是中间出现了些插曲。

# 需求

* Ruby 2.3.0 +
* Memcached 1.4 +
* Redis 2.8 +
* PostgreSQL 9.4 +
* ImageMagick 6.5+
* Elasticsearch 2.0+

# 安装

## Ruby
使用rbenv安装。

## PostgreSQL

这里安装的是9.4版本，Ubuntu 12.04里需要添加源，参考

http://www.postgresql.org/download/linux/ubuntu/


创建 /etc/apt/sources.list.d/pgdg.list 文件,然后加入一行源：

```
deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main
```

导入源的签名，然后更新包：

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -
sudo apt-get update
```

就可以安装啦：

```
sudo apt-get install postgresql-9.4
```

### 使用

* 切换到postgres用户： sudo  su - postgres
* 进入控制台： psql
* 修改密码： \password postgres
* 创建其它用户: CREATE USER deploy WITH PASSWORD 'password'

### 问题

#### No PostgreSQL clusters exist; see “man pg_createcluster”

```
sudo pg_createcluster 9.4 main
```

#### Cannot set LC_ALL to default locale: No such file or directory

在 /etc/environment 文件最后加入:

```
LANGUAGE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
```

#### role does not exist

默认postgresql会创建一个postgres用户，具有所以权限。可我是用deploy用户部署的，
因此需要在数据库里创建这个用户，并加入权限。

```
postgres=# CREATE ROLE deploy CREATEDB;
```

然后可以用“\du”查看到相应的role信息了。

如果想添加权限，比如LOGIN：

```
postgres=# ALTER ROLE deploy LOGIN;
```


## Elasticsearch

```
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
sudo apt-get update && sudo apt-get install elasticsearch openjdk-7-jre-headless
sudo update-alternatives --config java
# 选择 java-7
```

安装完成后需要启动：

```
sudo /etc/init.d/elasticsearch start
```

## 其它

```
$ sudo apt-get install memcached redis-server imagemagick ghostscript
```

More details about install PostgreSQL

## Ruby-China

```
$ git clone https://github.com/ruby-china/ruby-china.git
$ cd ruby-china
$ ./bin/setup
Checking Package Dependencies...
--------------------------------------------------------------------------------
Redis 2.0+                                                                 [Yes]
Memcached 1.4+                                                             [Yes]
ImageMagick 6.5+                                                           [Yes]
--------------------------------------------------------------------------------

Installing dependencies
--------------------------------------------------------------------------------
The Gemfile's dependencies are satisfied
--------------------------------------------------------------------------------

Configure
--------------------------------------------------------------------------------
Your Redis host (default: 127.0.0.1:6379):
Your Elasticsearch host (default: 127.0.0.1:9200):
--------------------------------------------------------------------------------

Seed default data...                                                      [Done]

== Removing old logs and tempfiles ==

Ruby China Successfully Installed.

$ rails s
```

## 测试

```
bundle exec rake
```

只要上面配置好，测试会一遍跑通。

# 部署

参考[Rails部署](http://fuhao.im/2015/09/22/rails-deploy.html)和
[Ruby-China的WiKi](https://github.com/ruby-china/ruby-china/wiki/Ubuntu-14.04-%E4%B8%8A%E4%BD%BF%E7%94%A8-Nginx-Passenger-%E9%83%A8%E7%BD%B2-Ruby-on-Rails)。

但是按照上面会出现两个问题：没有资源文件和HTTPS访问。


## CDN配置

对于https，简单粗暴的方法，在 config/environments/production.rb配置文件里关掉https：

```
config.force_ssl = false
```

虽然已经precompile资源文件，可还是没有css等资源文件。

看源码，发现stylesheet引用的文件NOT FOUND。

因为Ruby-China使用的是upyun的cdn，会发现上面的那个文件是指向upyun的，而不是本地的。

再参考 config/deploy.rb 文件，发现：

```ruby
task :compile_assets, roles: :web do
  run "cd #{current_path}; RAILS_ENV=production bundle exec rake assets:precompile"
  run "cd #{current_path}; RAILS_ENV=production bundle exec rake assets:cdn"
end
```

所以需要执行assets:cdn任务，将资源文件传上去，之后就可以正常访问啦！

## HTTPS

如果想使用https呢，那么常规的方法在此。

### 1.生成证书

```
openssl req -new -newkey rsa:2048 -nodes -keyout myapp.key -out myapp.csr

# 生成证书
openssl x509 -req -days 3650 -in myapp.csr -signkey myapp.key -out myapp.crt
```

有钱的话可以申请正规的哦～

### 2.配置Nginx

```
server {
        listen 80 default;
        listen 443 ssl;

        server_name myapp.com;

        root /home/deploy/apps/myapp/public;

        ssl on;
        ssl_certificate /etc/ssl/myapp.crt;
        ssl_certificate_key /etc/ssl/myapp.key;

        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        ssl_prefer_server_ciphers on;

        passenger_enabled on;
        passenger_ruby /home/deploy/.rbenv/versions/2.3.0/bin/ruby;

}

```

然后重启Nginx。

###

按理说应该OK的，但错误不期而遇，看Chrome里的Log：

```
xxx was loaded over HTTPS, but requested an insecure stylesheet
```

原理css文件也要通过https传输，而我们的还是http，就被block了。

于是去config/config.yml里更改upload_url，加个s。

然后重启precompile assets，然后同步传到cdn上。

这样按道理重启Passenger就会又见css的，可是没有，因为我们用的人家Ruby-China的测试cdn，没有开通
https权限。

好吧，那我把assets改到我自己的服务器上好吧，就改上面的upload_url吧，
因为它在config/environment/production.rb里是这样被使用的：

```
config.action_controller.asset_host = Setting.upload_url
```

很简单，改成“”，那么就会默认使用本机的assets路径啦。


# 感想
从ruby-china能学到很到，代码也紧跟Rails的最新版。

要另开一篇专门研究ruby-china。

源代码下下来没用，只有研究明白了，知道是怎么写的，才算是好的。一个好的项目的代码，如同一部精彩的
文学作品，细细品来，里面有作者的精心设计的框架，有精妙的生花之笔，有漂亮的启承转合，漂亮！

# 参考
* https://github.com/ruby-china/ruby-china
* http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html
* http://www.cnblogs.com/pxpxstudy/p/3731858.html
* http://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue
