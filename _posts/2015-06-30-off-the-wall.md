---
layout: post
title: Off the Wall
---

对，我喜欢Vans！无拘无束，见沟飞，见墙翻。

没有VPS又不想花钱买VPN的话用goAgent，有VPS的话可以架VPN，也可以使用隧道接口然后在使用时反向代理。

# [goAgent](https://github.com/goagent/goagent)

虽然免费，但不稳定，总要升级。

# [ShadowSocks](https://github.com/shadowsocks/shadowsocks)


## Server搭建

Debian/Ubuntu

```
apt-get install python-pip
pip install shadowsocks
```

### 打开服务

```
ssserver -p 443 -k password -m aes-256-cfb
```

### 在后台运行：

```
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start
```

### 停止服务：

```
sudo ssserver -d stop
```

### 查看Log：

```
sudo less /var/log/shadowsocks.log
```

## Client连接

```
sslocal -s IP -p PORT -b 127.0.0.1 -l 1080 -k PASSWORD -t 600 -m aes-256-cfb

```

然后在Chrome中就可以用SwitchOmega来使用了。

# SSH

下面建立了一条绑定本地**7070**端口的静默隧道，-p表示server shh的端口。

```
ssh -qTfnN -D 7070 user@server.address -p 443
```

相比ShadowSocks，SSH也是使用socket5来建立隧道，更简单，但是不如SS方便手机使用，而且比SS稍慢。

参考：

* http://www.williamlong.info/archives/4121.html
* http://vpsnews.cc/shadowsocks-ssh%E5%8C%BA%E5%88%AB/

# VPN
VPN用着很方便，还能发放帐号呢。

### 1. 安装PPTPD

```
apt-get install pptpd
```

### 2. 编辑VPN的接口IP地址

打开**/etc/pptpd.conf**，找到：

```
#localip 192.168.0.1
#remoteip 192.168.0.234-238,192.168.0.245
```

去掉"#"。

### 3. 设置DNS地址
打开**/etc/ppp/pptpd-options**，找到：

```
#ms-dns 10.0.0.1
#ms-dns 10.0.0.2
```

变为google的DNS地址：

```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

当然也可以是其它公开可用的，如OpenDNS: 208.67.222.222 & 208.67.220.220。
(Tips: You can also copy the above codes and paste it under the original ones.)

### 4. 添加VPN帐号
打开**/etc/ppp/chap-secrets**，在文件最后加入帐号，格式如下，用TAP街隔开：

username   pptpd   password   *

比如：

freenuts   pptpd   123456   *

### 5. 设置转发IPv4
打开**/etc/sysctl.conf**，找到：

```
#net.ipv4.ip_forward=1
```

去掉签名注释"#"。

### 6. 更新转发生效
输入下面命令：

```
sysctl -p
```

如果正确的话会返回：

net.ipv4.ip_forward = 1

### 6. 设置防火墙允许路由

```
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
```

### 7. 重启PPTPD

```
/etc/init.d/pptpd restart
```

然后搜索如何使用PPTP（Point-to-Point Tunneling Protocol） VPN，简单设置即可使用了。

上帝用七天创造了世界，我们用七步走进了更广阔的世界，阿门！

参考：http://freenuts.com/how-to-set-up-a-vpn-in-a-vps/

# 关于DNS

我目前用的是DigitalOcean的最低版本（5美元/月），这里有推荐链接，使用可以多得10美元：[https://www.digitalocean.com/?refcode=3d496b50e388](https://www.digitalocean.com/?refcode=3d496b50e388)

