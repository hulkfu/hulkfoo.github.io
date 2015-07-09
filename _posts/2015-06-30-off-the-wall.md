---
layout: post
title: Off The Wall
---

对，我喜欢Vans！无拘无束，见沟飞，见墙翻。

只为科学上网！

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

## Linux/MacOS Client 使用

1.首先也安装ss：pip install shadowsocks

2.然后执行下面的命令，不用装什么客户端：

```
sslocal -s IP -p PORT -b 127.0.0.1 -l 1080 -k PASSWORD -t 600 -m aes-256-cfb

```

3.在Chrome下用[SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega)设置即可。

## iOS 使用

去下载官方应用，[iTunes走起](https://itunes.apple.com/us/app/shadowsocks/id665729974?ls=1&mt=8)。

### 浏览器使用
可以通过**+**来设置自己的代理，默认会用公用的代理。

### 本地全局PAC代理

但有以下限制：

* 需要WiFi联网。
* 只能用几分钟。因为iOS的限制，它不能在后台一直运行，所以需要不时的来看看。

设置代理：

* 打开iOS的设置 -> Wi-Fi -> i 图标（在当前所连WiFi的后面） -> HTTP 代理
* 选择自动，输入http://127.0.0.1:8090/proxy.pac
* 返回

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

## UFW防火墙配置

### 允许1723默认端口
```
sudo ufw allow 1723
```

### 在**/etc/default/ufw**里将DEFAULT_FORWARD_POLICY 变成 “ACCEPT”:

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

### 在**/etc/ufw/before.rules**文件里

#### 1. 在Don't delete these required lines上面加入

```
# NAT table rules
*nat

:POSTROUTING ACCEPT [0:0]
# Allow forward traffic to eth0
-A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE

# Process the NAT table rules
COMMIT

# Don't delete these required lines, otherwise there will be errors
...
```

#### 2. 在filter部分前面加入（其实位置都无所谓，只要在require后，COMMIT前即可）

```
# First, since we trust pptpd completely, I would accept all traffic to/from my pptpd. I added this lines at the beginning of the filter section.
-A ufw-before-input -i ppp+ -j ACCEPT
-A ufw-before-output -i ppp+ -j ACCEPT

# Additionally, I must forward traffic to/from my pptpd. These lines was also added after the above lines.
-A ufw-before-forward -s 192.168.0.0/24 -j ACCEPT
-A ufw-before-forward -d 192.168.0.0/24 -j ACCEPT
```

#### 3. 在drop INVALID packets前面加上：

```
-A ufw-before-input -p 47 -j ACCEPT
-A ufw-before-output -p 47 -j ACCEPT

# drop INVALID packets (logs these in loglevel medium and higher)
...
```

#### 参考：

* http://www.cviorel.com/2009/02/09/how-to-set-up-a-vpn-server-on-ubuntu/
* http://askubuntu.com/questions/119534/easiest-way-to-setup-ubuntu-as-a-vpn-server
* http://ubuntuforums.org/showthread.php?t=1113911

# 关于VPS

我目前用的是DigitalOcean的最低版本（5美元/月），这里有推荐链接，使用可以多得10美元：[https://www.digitalocean.com/?refcode=3d496b50e388](https://www.digitalocean.com/?refcode=3d496b50e388)

