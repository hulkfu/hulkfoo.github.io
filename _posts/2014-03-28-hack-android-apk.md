---
layout: post
title: 逆向与反逆向Android APK文件
---

# 二次打包
[android-apktool](https://code.google.com/p/android-apktool/)能将apk里的资源文件解压出来，并将代码反编译成dalvik的机器语言。

它包括：

* [apktool.jar 核心功能文件][1]
* [aapt和apktool 脚本(Linux)][2]

下载解压放到同一个目录里，就可以使用了。

它有decode和build功能，即解包和打包。如下使用：

```bash
apktool d [OPTS] file.apk [<dir>] # 将file.apk文件解包到dir里。

apktool b [OPTS] [<app_apth>] [<out_file>] # 将app的文件夹打包成到out_file。如果app_path空，则默认当前目录；如果out_file空，则<app_path>/dist/<name_of_original.apk>。
```

其它：

```
sed -i "s/oldString/newString/g" `grep oldString -rl /path`
```

# 重新签名
因为二次打包会抹掉签名信息，因此需要重新签名。

## 创建签名
利用标准的java工具keytool创建key，利用jarsigner工具使用生成的key来生成证书和给程序签名。


```
 keytool -genkey -alias demo.keystore -keyalg RSA -validity 20000 -keystore demo.keystore
```

keytool工具是Java JDK自带的证书工具，参数解释如下：

* -genkey参数表示：要生成一个证书（版权、身份识别的安全证书）
* -alias参数表示：证书有别名，-alias demo.keystore表示证书别名为:demo
* -keyalg RSA表示加密类型，RSA表示需要加密，以防止别人盗取
* -validity 20000表示有效时间20000天
* -keystore demo.keystore表示要生成的证书名称为demo

## 签名
使用生成的key对apk签名，运行如下命令：

```
jarsigner -verbose -keystore demo.keystore -signedjar demo_signed.apk demo.apk demo.keystore
```

jarsigner是Java的签名工具，参数解释如下：

* -verbose参数表示：显示出签名详细信息
* -keystore表示：使用当前目录中的demo.keystore签名证书文件。
* -signedjar demor_signed.apk demo.apk demo.keystore 正式签名，三个参数中依次为签名后产生的文件demo_signed，要签名的文件demo.apk和密钥库demo.keystore.

如果有pem和pk8文件，也可以直接使用：

```
signapk.jar platform.x509.pem platform.pk8 demo.apk demo_signed.apk
```

可以用下面命令查看是否签名成功：

```
jarsigner –verify demo.apk
```

签名后需要做对齐优化处理：

```
zipalign -v 4 your_project_name-unaligned.apk your_project_name.apk
```

# 反编译代码
[dex2jar](http://code.google.com/p/dex2jar/)将Android apk里的dex文件转换成jar文件。

```bash
dex2jar.sh file.apk #就会在相同目录出来它的jar版
```

生成jar文件后，就可以用Java Decompiler来反编译成java源码了，这里用[JD-GUI](http://jd.benow.ca/)，直接打开jar文件即能看见源码。

下面软件下载：

* [dex2jar][3]
* [jd-gui linux][4]

# 反逆向
上面可知，在发布apk时，一定要做好反逆向，要不你的代码就是裸奔，其实保护一下就是多穿了件衣服，费些时间还是能看到里面的。

## 混淆

## .so库



[1]: /file/apktool1.5.2.tar.bz2 "apktool"
[2]: /file/apktool-install-linux-r05-ibot.tar.bz2 "apktool script"
[3]: /file/dex2jar-0.0.9.15.zip "dex2jar"
[4]: /file/jd-gui-0.3.5.linux.i686.tar.gz "jd-gui"
