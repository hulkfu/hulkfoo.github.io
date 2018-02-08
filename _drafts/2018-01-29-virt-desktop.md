---
layout: post
title: 虚拟桌面云
permalink: virt
---

虚拟桌面云就是在服务器上运行虚拟机，但是把桌面投到远程用户那里，并且可以将用户本地的硬件，如 USB、显卡等重定向到服务器的虚拟机里。

用户和远程虚拟机之间的通信协议是SPICE。

后台虚拟机的管理参考[之前写的 libvirt](/libvirt).

在桌面云里，本地硬件的重定向是关键的技术。实现了重定向，本地机器就可以成为完全的“傀儡”，向云端提供有力支持。

# [SPICE 协议](https://www.spice-space.org/)

# 使用

安装：

```bash
sudo apt install virt-viewer
```

## virt-viewer

virt-viewer --connect qemu+ssh://user@yourserver/system winVM

参数如下：

- --connect: is obviously the parameter telling the command to connect!
- qemu+ssh: telling virt-viewer that this host is reachable via SSH and QEMU is used for virtualization (since it supports Xen as well)
- system: is a static word, just type it as it is!
- winVM: The name you gave to the Virtual Machine

## remote-viewer
remote-viewer 其实是和 virt-viewer 在一个软件包里，只不过它有图形界面。

除了使用 qemu+ssh 的方式，也可以用 spice 协议。

但默认 spice 是只有本地使用的，如果外网访问，需要配置 Spice Server 的地址为 All interfaces. 用 virt-manager 可以在虚拟机 hardware details 的 Display Spice 选项卡里方便配置。

相比 spice 的速度比 ssh 的快，应该是 spice 默认是不加密的。

在命令行后加 --spice-debug，方便调试。

# USB 重定向
USB 重定向应该是 SPICE 协议最大的用处了，可以把本地的 U 盘重定向到远程的虚拟机里。

## USB3.0 重定向
而默认虚拟机是不支持 USB3.0 的，需要设置一下，可以参考[SPICE usbredir 库的官方文档](https://www.spice-space.org/usbredir.html)。

简单的就是在 virt-manager 里虚拟机的 Controller USB 设置里，Model 选择 USB 3 即可。


# [FreeRDP](https://github.com/FreeRDP/FreeRDP/)
其实对 Windows 系统，还有一种方案 —— 远程桌面。开启云里虚拟机的远程桌面，然后在本地登录。

它也支持重定向：

```bash
# 把本地的 /home/user/Share/ 目录重定向到远程，名为 linux
xfreerdp /u:username /p:password /v:192.168.0.3 /drive:linux,/home/user/Share/
```

参考[Command Line Interface](https://github.com/FreeRDP/FreeRDP/wiki/CommandLineInterface):

```bash
+clipboard : redirect clipboard
/drive:<sharename>,<path> : Redirect <path> as share <sharename>
/smartcard:<device> : Redirect smartcard <device>
/printer:<device>,<driver> : Redirect a specific printer. Redirecting all printers does not currently work.
/serial:<device> : redirect serial port <device>
/parallel:<device> : redirect parallel port <device>
/microphone:sys:alsa : Redirect audio input
/sound:sys:alsa : Redirect audio output
/multimedia:sys:alsa : Multimedia redirection
/usb:id,<usbid> : Redirect usb device <usbid>. Usbid is a string like dev:054c:0268
```


# 参考
- https://www.spice-space.org/spice-user-manual.html
- https://www.linux-kvm.org/page/SPICE
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/virtualization_administration_guide/
