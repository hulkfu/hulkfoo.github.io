---
layout: post
title: Debian 包管理
permalink: dpkg
---



- 编译包： dpkg-buildpackage
- 下载包源码：

修改代码后需要去 debian/changelog 修改版本号等信息。

aptitude install 会安装要安装库相关依赖包。

apt-get build-dep package 会自动安装这个虚要编译包的环境。
# 参考
- https://wiki.debian.org/Packaging/Intro?action=show&redirect=IntroDebianPackaging
