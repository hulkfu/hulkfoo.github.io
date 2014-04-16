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
通过混淆，会让代码变得难读，类名、变量等都变成了字母表。

* 去掉project.propert文件中的注释：proguard.config=${sdk.dir}/tools/proguard/proguard-android.txt:proguard-project.txt。
* 在上面路径中的proguard-project.txt文件里设置。

具体可参考[官方文档](http://developer.android.com/tools/help/proguard.html)。

## JNI
反编译这个就需要更进一步的工具和能力，这样就提高了门槛。

使用步骤：

* 在Java代码中用**native**关键字声明这个原生方法。
* 用javah生成相应的头文件。
* 原生代码和Android.mk文件。在Eclipse环境下可以轻松通过：右击工程->Android Tools->Add Native Support... 来创建代码框架。
* 在上面生成的jni目录里引用头文件，编写C代码。
* 在声明native方法的Java类中使用System.loadLibrary("libname")载入库，一般在static块中载入，然后就可以使用用native方法。

Java调用C/C++通过上面步骤很容易就实现了，可如果需要在C/C++里面处理Java层的东西，就需要：

* 通过包名、类名找到类。
* 通过方法签名在类中找到方法ID。
* 调用。

下面是一个简单调用Java类中静态方法的C++代码：

```
int JNICALL Java_com_example_name_Test_init
  (JNIEnv *env, jclass klass)
{
  jclass App = env->FindClass("com/example/name/App");
  jmethodID getContext_method_id = env->GetStaticMethodID(App, "getContext", "()Landroid/content/Context;");
  jobject context = env->CallStaticObjectMethod(App, getContext_method_id);

  return 1;
}
```

上面的代码在Java中就是执行App.getContext()，但在C++中，它不知道Java中的信息。因此要通过env变量也就是虚拟机环境找到类，类的方法id，然后再调用，很符合逻辑。

说明一下这里的方法签名和smali中的是一样的。

# 总结
apk是要把自己的安装包发布出去，不像web别人浏览到的只是网页。所有发布前要谨慎，尽可能保护自己的代码。

想保护代码，就要知道怎么逆向代码，要去逆向分析代码就得会写代码。

# 参考
* [C调用Java](http://www.cnblogs.com/luxiaofeng54/archive/2011/08/17/2142000.html)
* [Java调用C](http://www.cnblogs.com/luxiaofeng54/archive/2011/08/15/2139934.html)

[1]: /file/apktool1.5.2.tar.bz2 "apktool"
[2]: /file/apktool-install-linux-r05-ibot.tar.bz2 "apktool script"
[3]: /file/dex2jar-0.0.9.15.zip "dex2jar"
[4]: /file/jd-gui-0.3.5.linux.i686.tar.gz "jd-gui"
