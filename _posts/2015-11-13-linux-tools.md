---
layout: post
title: Linux下好用的工具
---

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

# 同步

## rsync
rsync - a fast, versatile, remote (and local) file-copying tool


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


# [lftp](http://lftp.yar.ru/)
比ftp命令好用，封装了 rm 等常用命令。

# 参考
- http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html
- http://www.cnblogs.com/peida/archive/2013/01/05/2846152.html
