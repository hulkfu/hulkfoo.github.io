---
layout: post
title: Linux下好用的工具
---

# 定时执行任务

## crontab
```bash
crontab -e
```

## at
```bash
$ at 2:05 tomorrow

at>echo "hi"
at> Ctrl+D
```

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
