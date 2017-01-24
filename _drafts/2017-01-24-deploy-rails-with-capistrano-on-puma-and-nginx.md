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


###


# 参考
- http://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line
- https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-rails-app-with-puma-and-nginx-on-ubuntu-14-04
