---
layout: post
title: iOS WebDriverAgent
permalink: wda
---

由于微信跳一跳的原因找到了它。

[WebDriverAgent](https://github.com/facebook/WebDriverAgent) 是 Facebook 开源的 iOS [WebDriver server](https://w3c.github.io/webdriver/webdriver-spec.html) 的实现，可以用来远程控制 iOS 设备。

It allows you to launch & kill applications, tap & scroll views or confirm view presence on a screen. This makes it a perfect tool for application end-to-end testing or general purpose device automation. It works by linking XCTest.framework and calling Apple's API to execute commands directly on a device. WebDriverAgent is developed and used at Facebook for end-to-end testing and is successfully adopted by Appium.

# 安装

首先安装 [Carthage](https://github.com/Carthage/Carthage)，用于管理依赖包：

```bash
brew install carthage
```

之后下载代码，运行初始化脚本：

```bash
git clone https://github.com/facebook/WebDriverAgent
./Scripts/bootstrap.sh
```

执行完成后，用 xCode 打开 WebDriverAgent.xcodeproj 项目。

之后需要设置证书，分别为 WebDriverAgentLib 和 WebDriverAgentRunner 设置。

设置目标设备：Menu > Product > Destination > xx的 iPhone

设置 Scheme: Menu > Scheme > WebDriverAgentRunner

这样就可以执行测试了！ Product > Test

此时在 Console 会显示一个 http 地址，一般是 http://your-iphone-ip:8100/，在这个地址后加上 status 就可以查看状态了。如果此时报错没有地址，那么把手机上装的这个应用卸载了试试。

而我们在 Mac 上测试有时是访问的 localhost:8100，因此需要将 iPhone 的端口转发过来：

```bash
brew install libimobiledevice
iproxy 8100 8100
```

通常来说为了持续集成，能够全部自动化比较好一些。

```bash
# 解锁keychain，以便可以正常的签名应用，
PASSWORD="replace-with-your-password"
security unlock-keychain -p $PASSWORD ~/Library/Keychains/login.keychain

# 获取设备的UDID
UDID=$(idevice_id -l | head -n1)

# 运行测试
xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination "id=$UDID" test
```

# 使用
使用 Python 控制，则需要安装相应的库 [facebook-wda](https://github.com/openatx/facebook-wda)：

```bash
pip install --pre facebook-wda
```

```python
import random
import wda

c = wda.Client()
s = c.session()

# 截屏
c.screenshot('1.png')

# 按屏幕
press_time = 1000
s.tap_hold(random.uniform(100,200), random.uniform(400,500), press_time)
```


# 参考
- https://github.com/wangshub/wechat_jump_game
- https://testerhome.com/topics/7220
- https://testerhome.com/topics/4904
- https://www.jianshu.com/p/ff973a5910ae
