---
layout: post
title: Linux下好用的工具
---
Linux 下各种好用的工具，可以让日常流程自动化、也可以实现你想要的效果。它们是 Linux 的宝贵资产，是代代程序员传承下来的。

仅仅三十年间计算机软件就有了如此的积累，甚至第一代程序员还在编程，已经形成了如此庞大的基础，而且还会一直持续下去。如果说人类是靠书本将知识传递了下来，那计算机自出生就是一体的，一直多是它一个，在一步一步被人类养大。所以，怎能不看好计算机呢？

# 基本命令
## find

## xargs
xargs - build and execute command lines from standard input.

它的作用就是批量处理多行的输入。如：

```bash
# 查出当前目录的文件，对每个文件执行 grep "to_find" 命令
find ./ -type f | xargs grep "to_find"
```


# 文本处理

更多参考[这里](http://linuxtools-rst.readthedocs.io/zh_CN/latest/base/03_text_processing.html)。

## grep 文本搜索

```bash
#在多级目录中对文本递归搜索，并显示行号(程序员搜代码的最爱）:
grep "class" . -R -n
```

# 定时执行任务

## crontab
```bash
crontab -e 编辑 # -l 列出现有的， -r 删除
```

### 格式

分 时 日 月 星期 要运行的命令

- 第1列分钟0～59
- 第2列小时0～23（0表示子夜）
- 第3列日1～31
- 第4列月1～12
- 第5列星期0～7（0和7表示星期天）
- 第6列要运行的命令

### 例子

```bash
实例1：每1分钟执行一次myCommand
* * * * * myCommand

实例2：每小时的第3和第15分钟执行
3,15 * * * * myCommand

实例3：在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * myCommand

实例4：每隔两天的上午8点到11点的第3和第15分钟执行
3,15 8-11 */2  *  * myCommand

实例5：每周一上午8点到11点的第3和第15分钟执行
3,15 8-11 * * 1 myCommand

实例6：每晚的21:30重启smb
30 21 * * * /etc/init.d/smb restart

实例7：每月1、10、22日的4 : 45重启smb
45 4 1,10,22 * * /etc/init.d/smb restart

实例8：每周六、周日的1 : 10重启smb
10 1 * * 6,0 /etc/init.d/smb restart

实例9：每天18 : 00至23 : 00之间每隔30分钟重启smb
0,30 18-23 * * * /etc/init.d/smb restart

实例10：每星期六的晚上11 : 00 pm重启smb
0 23 * * 6 /etc/init.d/smb restart

实例11：每一小时重启smb
* */1 * * * /etc/init.d/smb restart

实例12：晚上11点到早上7点之间，每隔一小时重启smb
0 23-7 * * * /etc/init.d/smb restart

# 在系统启动后执行脚本
@reboot /path/to/script
```


## at
```bash
# at 5pm+3 days
$ at 2:05 tomorrow

at>echo "hi"
at> <Ctrl+D>
```

at -c 14  显示 14 号任务内容

atq 显示待执行的任务
atrm 删除任务

## rc
系统重启后执行的任务。执行的任务都需要在  /etc/init.d/ 目录里。

用 update-rc.d 进行控制。

```bash
#创建要开机自动执行的脚本：/home/test/blog/startBlog.sh，并给予可执行权限：
chmod +x /home/test/blog/startBlog.sh

# 在/etc/init.d目录下创建链接文件到前面的脚本
ln -s /home/test/blog/startBlog.sh /etc/init.d/startBlog。

# 进入/etc/init.d目录，用 update-rc.d 命令将连接文件 startBlog 添加到启动脚本中去：
# 其中的99表示启动顺序，取值范围是0-99。序号越大的越晚执行。
update-rc.d startBlog defaults 99

# 移除启动的脚本，-f选项表示强制执行
# nstead of removing the links, disable them. The difference is that removal does not make dpkg aware of your preference to prevent the startup scripts from being executed because dpkg runs  update-rc.d too for removing and adding the links.
update-rc.d -f startBlog disable
```

而 /etc/init.d/ 里的文件是有格式的，即 Subsystems，像 Nginx、PostgreSQL那样。


# [byzanz](https://github.com/GNOME/byzanz)

能录制屏幕，并输出gif格式（根据输出文件名的后缀来输出文件类型）。

这是其使用方法：

```
Usage:
  byzanz-record [OPTION...] record your current desktop session

-d, --duration=SECS      Duration of animation (default: 10 seconds)
-e, --exec=COMMAND       Command to execute and time
--delay=SECS             Delay before start (default: 1 second)
-c, --cursor             Record mouse cursor
-a, --audio              Record audio
-x, --x=PIXEL            X coordinate of rectangle to record
-y, --y=PIXEL            Y coordinate of rectangle to record
-w, --width=PIXEL        Width of recording rectangle
-h, --height=PIXEL       Height of recording rectangle
-v, --verbose            Be verbose
--display=DISPLAY        X display to use
```

默认需要指定区域，这里有一个脚本可以选择要录制的应用窗口。

byzanz-record-window文件：

```bash
#!/bin/bash

# Delay before starting
DELAY=10

# Sound notification to let one know when recording is about to start (and ends)
beep() {
    paplay /usr/share/sounds/KDE-Im-Irc-Event.ogg &
}

# Duration and output file
if [ $# -gt 0 ]; then
    D="--duration=$@"
else
    echo Default recording duration 10s to /tmp/recorded.gif
    D="--duration=10 /tmp/recorded.gif"
fi
XWININFO=$(xwininfo)
read X < <(awk -F: '/Absolute upper-left X/{print $2}' <<< "$XWININFO")
read Y < <(awk -F: '/Absolute upper-left Y/{print $2}' <<< "$XWININFO")
read W < <(awk -F: '/Width/{print $2}' <<< "$XWININFO")
read H < <(awk -F: '/Height/{print $2}' <<< "$XWININFO")

echo Delaying $DELAY seconds. After that, byzanz will start
for (( i=$DELAY; i>0; --i )) ; do
    echo $i
    sleep 1
done

beep
byzanz-record --verbose --delay=0 --x=$X --y=$Y --width=$W --height=$H $D
beep
```

从代码中可以看出，参数第一个跟的是duration，默认10秒，其它的就是byzanz的参数形式了，最后跟
输入的文件路径。

所以可以这样使用：

```bash
byzanz-record-window 30 -c output.gif
```

录30秒，输出为output.gif文件。

参考：

http://askubuntu.com/questions/107726/how-to-create-animated-gif-images-of-a-screencast

# 计算机状态

## sysctl - configure kernel parameters at runtime

永久配置在 /etc/sysctl.conf 文件里。

比如设置 swap 的使用率，参考[这里](https://askubuntu.com/questions/1357/how-to-empty-swap-if-there-is-free-ram)。

查看：

```bash
cat /proc/sys/vm/swappiness
```


临时的：

```bash
sudo swapoff -a

# To set the new value to 0:
sudo sysctl vm.swappiness=0

# To turn swap back on:
sudo swapon -a

```

永久的：

```bash
sudoedit /etc/sysctl.conf
Add this line vm.swappiness = 0
sudo shutdown -r now # restart system
```



## top

## htop - interactive process viewer

## du

## df

# 网络

## netcat — arbitrary TCP and UDP connections and listens

## nmap

## netstat

```bash
# 查看使用 80 端口的程序
sudo netstat --all --program | grep '80'
```
## fuser - identify processes using files or sockets

## iftop - display bandwidth usage on an interface by host

## vnstat - a console-based network traffic monitor
它会在后台运行一个 vnstatd 的后台进程，收集流量信息。

## nethogs - Net top tool grouping bandwidth per process
在不打断现有网络协议的情况下，统计每个进程的网络使用带宽。

## [hogwatch](https://github.com/akshayKMR/hogwatch)


# 文件传输

## [rsync] —— 同步

命令格式：

```bash
# 使用起来很像 ssh
rsync [option] 源路径 目标路径

其中 [option]：
  -a:使用archive模式，等于-rlptgoD，即保持原有的文件权限
  -z:表示传输时压缩数据
  -v:详细模式输出
  -e:使用远程shell程序（可以使用rsh或ssh）
  --progress: 显示备份过程
  --delete:精确保存副本，源主机删除的文件，目标主机也会同步删除
  --include=PATTERN:不排除符合PATTERN的文件或目录
  --exclude=PATTERN:排除所有符合PATTERN的文件或目录
  --password-file:指定用于rsync服务器的用户验证密码
  -s, --protect-args: 如果文件名里有空白，不会把它们分开当成多个文件，同时如 ~ 也不会被翻译
```

### 备份本地数据
比 cp 好用，开始几分钟显示比较慢，但后面就很快。

最大的优点是可以中断，当然可以接着 cp 的中断用 rsync。

```bash
rsync -av --progress  ./test.c  /backup
```

### rsync 和 ssh 差异远程同步命令

例子：

```bash
rsync -ave ssh  user@192.168.1.109:/home/user/test/ /home/my/test_new/　　

# 将本地内容同步到远程目录
rsync -ave ssh /home/my/test_new/  user@192.168.1.109:/home/user/test/

# 将远程上的test目录内容同步到本地的New_Test目录，并删除本地上源路径中不存在的文件或目录。
# 千万要注意--delete参数，在使用此参数的时候，建议用绝对路径指定本地目录，防止清空当前目录
rsync -avz --delete user@192.168.1.109:/home/user/test/ /home/my/New_Test/
```

默认不用 -e ssh，rsync 就会自动用 ssh 的。

## [lftp](http://lftp.yar.ru/)
比ftp命令好用，封装了 rm 等常用命令。

# 远程控制
[Remmina](https://github.com/FreeRDP/Remmina)

# 参考
- http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html
- http://www.cnblogs.com/peida/archive/2013/01/05/2846152.html
- http://www.cnblogs.com/daxian2012/archive/2012/08/01/2618375.html
- http://www.tldp.org/HOWTO/HighQuality-Apps-HOWTO/boot.html
