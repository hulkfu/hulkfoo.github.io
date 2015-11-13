---
layout: post
title: Linux下好用的工具
---

# [byzanz](https://github.com/GNOME/byzanz)

能录制屏幕，并输出gif格式（根据输出文件名的后缀来输出文件类型）。

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

所以可以这样使用：

```bash
byzanz-record-window 30 -c output.gif
```

* http://askubuntu.com/questions/107726/how-to-create-animated-gif-images-of-a-screencast
