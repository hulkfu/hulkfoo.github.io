---
layout: post
title: 逆向APK文件
---

#二次打包
[android-apktool](https://code.google.com/p/android-apktool/)能将apk里的资源文件解压出来，并将代码反编译成dalvik的机器语言。

它包括：

* [apktool.jar 核心功能文件][1]
* [aapt和apktool 脚本(Linux)][2]

下载解压放到同一个目录里，一遍是~/bin/目录，就可以使用了。

它有decode和build功能，即解包和打包。如下使用：

```bash
apktool d [OPTS] file.apk [<dir>] # 将file.apk文件解包到dir里。

apktool b [OPTS] [<app_apth>] [<out_file>] # 将app的文件夹打包成到out_file。如果app_path空，则默认当前目录；如果out_file空，则<app_path>/dist/<name_of_original.apk>。
```

#反编译代码
[dex2jar](http://code.google.com/p/dex2jar/)将Android apk里的dex文件转换成jar文件。

生成jar文件后，就可以用Java Decompiler来反编译成java源码了，这里用[JD](http://jd.benow.ca/)。


[1]: /file/apktool1.5.2.tar.bz2 "apktool"
[2]: /file/apktool-install-linux-r05-ibot.tar.bz2 "apktool script"
