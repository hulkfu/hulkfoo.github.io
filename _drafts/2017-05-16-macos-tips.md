---
layout: post
title: MacOS TIPS
permalink: macostips
---

# 自动启动
在 ~/Library/LaunchAgents 里创建 plist 文件说明就行了。

比如下面是用 homebrew 安装的 memcached 的自动运行文件:

```coffee
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>homebrew.mxcl.memcaseched</string>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/opt/memcached/bin/memcached</string>
    <string>-l</string>
    <string>localhost</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>WorkingDirectory</key>
  <string>/usr/local</string>
</dict>
</plist>
```

dict 的字段很直观的。

还有其他目录的：

```bash

Type	Location	Run on behalf of
User Agents	~/Library/LaunchAgents	Currently logged in user
Global Agents	/Library/LaunchAgents	Currently logged in user
Global Daemons	/Library/LaunchDaemons	root or the user specified with the key UserName
System Agents	/System/Library/LaunchAgents	Currently logged in user
System Daemons	/System/Library/LaunchDaemons	root or the user specified with the key UserName

```


# 参考
- http://www.launchd.info/
