---
layout: post
title: HART协议
---

HART协议是工业上使用现有电源线使设备间通信的协议。

# SDC625
SDC625是HART电脑版上位机软件，加入协会后就可以下载源码。

* 开发环境：Visual Studio 2008
* 图表库：[Iocomp Components](http://www.iocomp.com/)

## 跑起来

1. 安装VS2008和Iocomp。
2. 双击SDC/trunk/APPS/SDC625/SDC625.sln
3. 在打开的项目里右键点击左侧的根项目，选择编译。
4. 编译成功后会在相应工程的DEBUG目录里生成可执行文件。

注意：
1. 上面的Iocomp可以到http://www.iocomp.com/下载，是图表库。
2. Visual Studio的版本必须是2008。
3. SDC625里面有多个sln文件，只有SDC625.sln可用。
