---
layout: post
title: 部署 Rails 应用
---

不得不说，Rails的发布是个坎，它涉及到自动部署、Nginx配置、缓存、代理等，
需要对这个Web系统概念有一个明确的了解，否则真的搞不定。

往往遇到搞不定的情况，就是自己的步子迈的太大，理解这一步的基础还没有打好。
比如还没有把rails应用手动部署成功，就想着用capistrano自动部署。

手动部署，就要对Rack、Rake、Bundler有所了解，然后还需要Linux、Nginx、数据库等的知识。

# Passenger & Nginx
按着[Passenger的教程](https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/digital_ocean/nginx/oss/trusty/install_passenger.html)，还是能够很轻松的deploy的。因为Passenger已经编译好了现成的Nginx。

## 安装软件

### 1. 安装Passenger和Nginx
```
# Install our PGP key and add HTTPS support for APT
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates

# Add our APT repository
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update

# Install Passenger + Nginx
sudo apt-get install -y nginx-extras passenger
```
### 2. 开启Nginx的Passenger模式

编辑 /etc/nginx/nginx.conf， 删除 passenger_root and passenger_ruby的注释：

```
# passenger_root /some-filename/locations.ini;
# passenger_ruby /usr/bin/passenger_free_ruby;
```

去掉”#“后：

```
Copypassenger_root /some-filename/locations.ini;
passenger_ruby /usr/bin/passenger_free_ruby;
```

然后重启Nginx：

```
sudo service nginx restart
```

### 3. 检查安装

检查Passenger是否正确安装：

```
sudo /usr/bin/passenger-config validate-install

* Checking whether this Phusion Passenger install is in PATH... ✓
* Checking whether there are no other Phusion Passenger installations... ✓
```

然后检测Nginx是否开启Passenger核心进程：

```
sudo /usr/sbin/passenger-memory-stats
Version: 5.0.8
Date   : 2015-05-28 08:46:20 +0200
...

---------- Nginx processes ----------
PID    PPID   VMSize   Private  Name
-------------------------------------
12443  4814   60.8 MB  0.2 MB   nginx: master process /usr/sbin/nginx
12538  12443  64.9 MB  5.0 MB   nginx: worker process
### Processes: 3
### Total private dirty RSS: 5.56 MB

----- Passenger processes ------
PID    VMSize    Private   Name
--------------------------------
12517  83.2 MB   0.6 MB    PassengerAgent watchdog
12520  266.0 MB  3.4 MB    PassengerAgent server
12531  149.5 MB  1.4 MB    PassengerAgent logger
...
```

### 4. 经常更新

它们的源已经在APT里了，所以经常更新哦：

```
sudo apt-get update
sudo apt-get upgrade
```

## 发布

### 1. 代码

把代码pull到合适的位置，比如/var/www/myapp/code目录下：

```
sudo mkdir -p /var/www/myapp
sudo chown myappuser: /var/www/myapp

cd /var/www/myapp
sudo -u myappuser -H git clone git://github.com/username/myapp.git code
```

### 2. 准备App的运行环境

#### a.安装相关gems

```
cd /var/www/myapp/code
bundle install --deployment --without development test
```

#### b.配置database.yml和secrets.yml

生成secret：

```
bundle exec rake secret
```
修改产品环境的secret：

```
production:
  secret_key_base: <%=ENV["SECRET_KEY_BASE"]%>
```

设置文件权限：

```
chmod 700 config db
chmod 600 config/database.yml config/secrets.yml
```

secrets必须要配置，否则在访问时会报“Incomplete response received from application”错误。

#### c.编译Rails的assets并且进行数据迁移

```
bundle exec rake assets:precompile db:migrate RAILS_ENV=production
```

### 3. 配置Nginx和Passenger

首先看一下Passenger使用的Ruby：

```
$ passenger-config about ruby-command
passenger-config was invoked through the following Ruby interpreter:
  Command: /usr/local/rvm/gems/ruby-2.2.3/wrappers/ruby
  Version: ruby 2.2.3p85 (2015-02-26 revision 49769) [x86_64-linux]
  ...
```

然后可以设置Nginx啦，编辑/etc/nginx/sites-enabled/myapp.conf：

```
server {
    listen 80;
    server_name yourserver.com;

    # Tell Nginx and Passenger where your app's 'public' directory is
    root /var/www/myapp/code/public;

    # Turn on Passenger
    passenger_enabled on;
    passenger_ruby /path-to-ruby; # 就是上一步得到的路径
}
```

重启Nginx就可以访问啦，如果有问题去/log/nginx/error.log 去排出解决。

## 更新
这里是手动的，当然可以用capistrano自动哦:)，只是明白了手动才知道自动是为什么。

首先拉取最新代码，然后：

```
bundle install --deployment --without development test
bundle exec rake assets:precompile db:migrate RAILS_ENV=production
passenger-config restart-app $(pwd)
```

# 参考
* https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/
* http://bundler.io/rationale.html
* http://stackoverflow.com/questions/29241053/incomplete-response-received-from-application-from-nginx-passenger
