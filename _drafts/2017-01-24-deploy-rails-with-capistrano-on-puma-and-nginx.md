---
layout: post
title: 用 Capistrano 在 Puma 和 Nginx 上部署 Rails 应用
---

这里用的是 Ubuntu 16.04.

# 从头来

### 创建 deploy 用户

```bash
sudo adduser deploy
sudo adduser deploy sudo
```

### 配置 ssh

```bash
cd ~
mkdir .ssh
touch .ssh/authorized_ke
chmod 600 .ssh/authorized_keysys
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

### 安装 Capistrano 3

```ruby
# Gemfile
gem "capistrano"
gem 'capistrano-rbenv'
gem 'capistrano3-puma'
gem 'capistrano-rails'
```

配置 Capfile 和 deploy.rb 及 /deploy 下相应的服务器。


### 配着 Puma
配合着 [capistrano3-puma](https://github.com/seuros/capistrano-puma) 很方便，比 passenger 还简单。

#### 1. 相关 gem 和 引用

```ruby
# Gemfile
gem 'capistrano3-puma' , group: :development
```

```ruby
# Capfile

require 'capistrano/puma'
# require 'capistrano/puma/workers' # if you want to control the workers (in cluster mode)
require 'capistrano/puma/jungle'  # if you need the jungle tasks
# require 'capistrano/puma/monit'   # if you need the monit tasks
require 'capistrano/puma/nginx'   # if you want to upload a nginx site template

```

#### 2. 生成 nginx 和 puma 配置模板文件

```ruby
rails g capistrano:nginx_puma:config
```

默认会在 config/deploy/templates 目录生成，不用改，默认多方便。

#### 3. 上传 nginx 和 puma 配置文件

```ruby
# nginx 如何在 /etc 目录需要 sudo，它会传到 /tmp 里，然后自己拷贝呗
cap puma:nginx_config
# 直接上传到 shared/puma.rb，所有需要将它也加到 deploy.rb 的 linked_files 里。
cap puma:config
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

#### 5. 发布

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

# 参考
- http://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line
- https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-puma-and-nginx-on-ubuntu-14-04
- https://ruby-china.org/topics/17425
