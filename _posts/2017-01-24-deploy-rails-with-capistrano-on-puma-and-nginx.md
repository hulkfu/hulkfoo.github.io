---
layout: post
title: 用 Capistrano 在 Puma 和 Nginx 上部署 Rails 应用
---

# 基本

## 服务器配置

这里用的是 Ubuntu 16.04.

### 创建 deploy 用户

```bash
sudo adduser deploy
sudo adduser deploy sudo
```

### 配置 ssh

```bash
cd ~
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys

# 添加自己 PC 的公钥
vim .ssh/authorized_keysys

# 关闭 Server 的 sshd 的密码认证
sudo vim /etc/ssh/sshd_config
# 设置 PasswordAuthentication no
# 重启 ssh 服务使之生效
sudo service ssh restart
```

### 配着防火墙 ufw

秉着最小权限原则。

```bash
# 开启后默认会阻止一起外部发起的访问，允许所有内部对外部的访问
sudo ufw enable
# 开启 ssh，默认 22 端口
sudo ufw allow 22/tcp

# web 端口
sudo ufw allow 80/tcp
```

### 安装 Ruby 环境

参考 [rbenv](https://github.com/rbenv/rbenv) 安装:

- rbenv
- ruby-build
- [rbenv-vars](https://github.com/rbenv/rbenv-vars)：可以用来配着环境变量


```bash
# install rbenv
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
# optionally
cd ~/.rbenv && src/configure && make -C src
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
~/.rbenv/bin/rbenv init

# install plugins
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
git clone https://github.com/rbenv/rbenv-vars.git $(rbenv root)/plugins/rbenv-vars

# install Ruby
rbenv install 2.4.0
rbenv global 2.4.0

gem install bundler
```

### 安装 Nginx

```bash
sudo apt-get install nginx
```

### 安装 PostgreSQL

```bash
# 安装自己需要的版本
sudo apt-get install postgresql-9.5 libpq-dev

sudo su - postgres
psql

# 第一次修改postgres的密码
\password postgres

# 为应用创建角色和密码
create role myapp with createdb login password 'password';

# 创建数据库
create database myapp_production owner = myapp encoding = 'UTF-8';

\q
```

之后需要去 /etc/postgresql/9.5/main/pg_hba.conf 配置一下访问权限：

```bash
local   all             all                                peer
# 改为
local   all             all                                md5
```

这样就能密码验证了。

其他可以参考之前的[文章](/postgresql)。

### 安装其他依赖库

```bash
sudo apt-get install nodejs memcached imagemagick
```

## Rails 配置

### 相关 Gem

```ruby
# Gemfile
group :development do
  gem "capistrano", require: false
  gem 'capistrano-rbenv', require: false
  gem 'capistrano3-puma', require: false
  gem 'capistrano-rails', require: false
  gem 'capistrano-postgresql', require: false
end
```

### Capistrano 3

```bash
# 应用根目录下 初始化
cap install
```

然后配置 Capfile：

```bash
require "capistrano/rbenv"
# require "capistrano/chruby"
require "capistrano/bundler"
require "capistrano/rails/assets"
require "capistrano/rails/migrations"
# require "capistrano/passenger"
require 'capistrano/postgresql'

# ...

# config/deploy.rb
set :rbenv_type, :user # or :system, depends on your rbenv setup
set :rbenv_ruby, '2.4.0'

# in case you want to set ruby version from the file:
# set :rbenv_ruby, File.read('.ruby-version').strip

set :rbenv_prefix, "RBENV_ROOT=#{fetch(:rbenv_path)} RBENV_VERSION=#{fetch(:rbenv_ruby)} #{fetch(:rbenv_path)}/home/deploy/.rbenv/bin/rbenv exec"
set :rbenv_map_bins, %w{rake gem bundle ruby rails}
set :rbenv_roles, :all # default value
```

在 config/deploy.rb 设置应用的相关信息，包括 application，repo_url，deploy_to，linked_files，linked_dirs 等主要信息。

在 config/deploy/production.rb 下设置相应的服务器。

配置完后首次部署一下：

```bash
cap production deploy setup
```

此前需要先将 linked_files 和 linked_dirs 在 server 上创建，否则它要先找到这些文件才去部署代码。

比如 database, secrets 等。

生成 secret_key_base：

```bash
rails secret
```


### Puma
配合着 [capistrano3-puma](https://github.com/seuros/capistrano-puma) 很方便，比 passenger 还简单。

#### 1. gem 引用

```ruby
# Capfile

require 'capistrano/puma'
install_plugin Capistrano::Puma  # Default puma tasks
install_plugin Capistrano::Puma::Workers  # if you want to control the workers (in cluster mode)
install_plugin Capistrano::Puma::Jungle # if you need the jungle tasks
install_plugin Capistrano::Puma::Monit  # if you need the monit tasks
install_plugin Capistrano::Puma::Nginx  # if you want to upload a nginx site template
```

#### 2. 生成 nginx 和 puma 配置模板文件

```ruby
rails g capistrano:nginx_puma:config
```

默认会在 config/deploy/templates 目录生成，不用改，默认多方便。

#### 3. 上传 nginx 和 puma 配置文件

```ruby
# nginx 如何在 /etc 目录需要 sudo，它会传到 /tmp 里，然后自己拷贝呗
cap production puma:nginx_config

# 直接上传到 shared/puma.rb，所有需要将它也加到 deploy.rb 的 linked_files 里。
cap production puma:config
```

#### 4. 配置
其实用默认的模板就可以正常运行了。

下面的就是 puma 模板生成的数据源，可以在 deploy.rb 里配置：

```ruby
set :puma_user, fetch(:user)
set :puma_rackup, -> { File.join(current_path, 'config.ru') }
set :puma_state, "#{shared_path}/tmp/pids/puma.state"
set :puma_pid, "#{shared_path}/tmp/pids/puma.pid"
set :puma_bind, "unix://#{shared_path}/tmp/sockets/puma.sock"    #accept array for multi-bind
set :puma_default_control_app, "unix://#{shared_path}/tmp/sockets/pumactl.sock"
set :puma_conf, "#{shared_path}/puma.rb"
set :puma_access_log, "#{shared_path}/log/puma_access.log"
set :puma_error_log, "#{shared_path}/log/puma_error.log"
set :puma_role, :app
set :puma_env, fetch(:rack_env, fetch(:rails_env, 'production'))
set :puma_threads, [0, 16]
set :puma_workers, 0
set :puma_worker_timeout, nil
set :puma_init_active_record, false
set :puma_preload_app, false
set :puma_plugins, []  #accept array of plugins
set :nginx_use_ssl, false
```

## 发布

首先去 server 上重启一下 Nginx，让新的 sites-enabled 配置文件生效，就能接收 puma 的信息了：

```bash
sudo service nginx restart
```

在自己 PC 上发布吧：

```ruby
cap production deploy
```

deploy 的命令过程中，尤其是第一次部署，还是需要解决些问题的，把提示的错误解决完就好了：

- shared 文件夹里文件：database.yml, secrets.yml 等。
- "Failed to build gem native extension" 类型的 gem 的安装。

# SSL

## 生成证书
使用免费的 [let's encrypt](https://letsencrypt.org/) 服务，参考[Ruby-China 的帖子](https://ruby-china.org/topics/31983) 和 [官方的 wiki](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)。

使用 [acme.sh](https://github.com/Neilpang/acme.sh)，很方便的。

主要是参考前者，并中和后者，比如 ngingx 的 reload 要用 force-reload。

修改一下 sudoer 文件，让 sudo service nginx reload 不需要输入密码：

```bash
sudo visudo

# 打开文件以后新增:
# NOTE: ubuntu 是 acme.sh 安装所用的账号
ubuntu  ALL=(ALL) NOPASSWD: /usr/sbin/service nginx reload
```

## 修改 Nginx 启用 SSL：

```
http {
  # 新增，在我的 /etc/nginx/nginx.conf 文件里已经有了
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  # 兼容其他老浏览器的 ssl_ciphers 设置请访问 https://wiki.mozilla.org/Security/Server_Side_TLS

  server {
    listen 80 default_server;
    # 新增
    listen 443 ssl;
    ssl_certificate         /etc/nginx/ssl/www.your-app.com.key.pem;
    ssl_certificate_key     /etc/nginx/ssl/www.your-app.com.key;
    # ssl_dhparam
    ssl_dhparam             /etc/nginx/ssl/dhparam.pem;

    # 其他省略
  }

  location @yourapp.com_production {
    # 解决 redirected you too many times
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # ...
  }
}
```

检查 Nginx 配置是否正确后重启

```bash
sudo service nginx configtest
sudo service nginx restart
```

## 问题

### sudo nginx -t
查看 nginx 的配置，并指出问题。

nginx 其实会把 sites-enabled 里所有的配置文件合并到 /etc/nginx/nginx.conf 来生成总的配置文件。

### redirected you too many times
把 environment/production.rb 的 config.force_ssl = true 开启后，发现出来了上述文件。

参考[ stackoverflow 里的回答](http://stackoverflow.com/questions/14930452/too-many-redirects-error-while-trying-to-configure-rails-application-as-ssl-usin)，主要原因是force_ssl 依赖于 HTTP_X_FORWARDED_PROTO HTTP 头来判断请求是否是 HTTPS 请求，如果不设置就会一直死循环的跳转。

在 app Nginx 配置文件里加入：

```bash
proxy_set_header X-Forwarded-Proto https;
```

# 原理
Rails 启动后通过 Puma 用 socket 绑定 app 的处理逻辑接口，Nginx 作为反向代理与 Puma 的 socket 进行交互。

主要在于 Nginx 的配置文件：

```c
upstream puma_my-awesome-app_production {
  server unix:/home/deploy/apps/my-awesome-app/shared/tmp/sockets/puma.sock fail_timeout=0;
}

server {
  listen 80;
  server_name caogupiao.com;
  # 公开静态资源的目录
  root /home/deploy/apps/my-awesome-app/current/public;
  # 上面的 public 里没有找到的话就把请求传到 puma 里
  try_files $uri/index.html $uri @puma_my-awesome-app_production;

  client_max_body_size 4G;
  keepalive_timeout 10;

  error_page 500 502 504 /500.html;
  error_page 503 @503;

  location @puma_my-awesome-app_production {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Forwarded-Proto http;
    proxy_pass http://puma_my-awesome-app_production;
    # limit_req zone=one;
    access_log /home/deploy/apps/my-awesome-app/shared/log/nginx.access.log;
    error_log /home/deploy/apps/my-awesome-app/shared/log/nginx.error.log;
  }

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  location = /50x.html {
    root html;
  }

  location = /404.html {
    root html;
  }

  location @503 {
    error_page 405 = /system/maintenance.html;
    if (-f $document_root/system/maintenance.html) {
      rewrite ^(.*)$ /system/maintenance.html break;
    }
    rewrite ^(.*)$ /503.html break;
  }

  if ($request_method !~ ^(GET|HEAD|PUT|PATCH|POST|DELETE|OPTIONS)$ ){
    return 405;
  }

  if (-f $document_root/system/maintenance.html) {
    return 503;
  }
}
```

# 升级
Rails 的更新还是很快的，一年至少两个新版本，而且新的功能都很吸引人。

升级时主要参考[http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html),看看具体的新版本的变化。

执行 app:update 命令能补全新版本的文件。

```bash
rails app:update
```

之后：

- 修改 Gemfile 里 rails 到要升级到的版本
- bundle update 升级 gems


# 参考
- http://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line
- https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-puma-and-nginx-on-ubuntu-14-04
- https://ruby-china.org/topics/17425
- https://ruby-china.org/topics/31983
