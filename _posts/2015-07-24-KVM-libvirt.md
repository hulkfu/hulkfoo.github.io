---
layout: post
title: KVM和libvirt
---

> Ubuntu uses KVM as the back-end virtualization technology primarily for non-graphic servers and libvirt as its toolkit/API.

KVM是内核级的虚拟化实现，而libvirt是对kvm的接口封装。

# libvirt

它就像kvm的shell，主要有三部分中组成：

* 应用程序编程接口库，即API
* 守护进程：libvirtd，在ubuntu里的service叫libvirt-bin。负责监听上层的命令。
* 默认命令行管理工具：virsh

重要概念：

* 节点（Node）：一个物理机器，上面可以运行多个虚拟机。
* Hypervisor：虚拟机监控器（VMM），如KVM、Xen、VMware、Hyper-V等，是虚拟化中的一个底层软件层。
* 域（Domain）：是在Hypervisor上运行的一个客户机操作系统实例。

# 安装
当前环境：Mint15.

## 1.首先检测是否支持kvm。

```
$ kvm-ok

# ok
INFO: /dev/kvm exists
KVM acceleration can be used

```

## 2.安装包

```
$ sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
```

## 3.加入用户组

```
$ sudo adduser `id -un` libvirtd
Adding user '<username>' to group 'libvirtd' ...
```

然后需要relogin一次，使用户加入libvirtd组。

## 4.管理

### virt-manager
安装图形化管理界面virt-manager，直接就可以使用了。

### virsh

安装操作系统：

```
virt-install -r 1024 --name xp --disk path=./xp2.img,size=8 --network network:default --cdrom ~/Downloads/xpsp3.iso --os-type=windows --accelerate --graphics vnc
```

### [webvirtmgr](https://github.com/retspen/webvirtmgr)
Django写的web管理。所依赖包有：

* libvirt-python：libvirt的python接口。
* novnc：web端vnc。
* supervisor：监控服务器，如自动运行webvirtmgr、配置noVNC。
* nginx：web反向代理服务器。


按照官方说明，配置好nginx后，配置supervisor，然后是配置ssh（如果用ssh来连接libvirt服务器）。


当然，也可以手动启动服务，在webvirtmgr的根目录下启动本地服务：

```
./manage runserver
```

然后重启nginx，就会将8000端口的服务映射到80上了。

然后开启[noVNC](https://github.com/kanaka/noVNC)，这样就可以在浏览器中查看界面了。

```
$ websockify 6080 127.0.0.1:5900
$ sudo iptables -I INPUT -m state --state NEW -s YOUR_IP_ADDRESS -m tcp -p tcp --dport 6080 -j ACCEPT
```


# 吐槽
好吧，又掌握了一项新技能，虚拟机平台，听说搜狗几千台服务器就是用的libvirt。一直以为很复杂呢，因为管几千台机器呢，结果用了一个晚上就能跑起来了（因为我只知道个大概原理，然后会敲命令用就行）。

我总是把事情想得太复杂，给自己不去做的理由。可只有去做才知道深浅啊，不要被自己首先拦着。

在新东西新领域新朋友新技术面前，放开心扉去接受吧！

会用libvirt让我很激动，仿佛看到了有千万台电脑在我的控制下。

# 参考

* https://help.ubuntu.com/community/KVM/Installation
* 《KVM虚拟化技术实战与原理解析》，任永杰，单海涛著