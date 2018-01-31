---
layout: post
title: 自动化操作
permalink: automation
---

自动化操作，就是模仿人的一系列操作。

# [Watir](http://watir.github.io/)

基于[Selenium](http://www.seleniumhq.org/)的浏览器自动控制接口，浏览器行为模拟器。

如果用Chrome做代理，需要先去下载ChromeDriver。

```ruby
require 'watir-webdriver'

browser = Watir::Browser.new :chrome

browser.goto 'google.com'
browser.text_field(title: 'Search').set 'Hello World!'
browser.button(type: 'submit').click

puts browser.title
# => 'Hello World! - Google Search'
browser.quit
```

## 选择
看[Container的文档](http://www.rubydoc.info/gems/watir-webdriver/Watir/Container)，
知道它能嵌套选择。

### Text Fields

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
t = b.text_field :id => 'entry_1000000'
t.exists?
t.set 'your name'
t.value
```

### Select Lists – Combos

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
s = b.select_list :id => 'entry_1000001'
s.select 'Ruby'
s.selected_options
```

### Radios

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
r = b.radio :value => 'A gem'
r.exists?
r.set
r.set?
```

### Checkboxes

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
c = b.checkbox :value => '1.9.2'
c.exists?
c.set
c.set?
```

### Buttons

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
btn = b.button :value => 'Submit'
btn.exists?
btn.click
```

### Links

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
l = b.link :text => 'Google Forms'
l.exists?
l.click
```

### Divs & Spans

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
d = b.div :class => 'ss-form-desc ss-no-ignore-whitespace'
d.exists?
d.text
s = b.span :class => 'powered-by-text'
s.exists?
s.text
```

## 等待
等待很有用的，因为很多网页现在都是异步加载的。

```ruby
require 'watir-webdriver'
b = Watir::Browser.start 'bit.ly/watir-webdriver-demo'
b.text_field(:id => 'entry_1000000').when_present.set 'your name'
b.select_list(:id => 'entry_1000001').wait_until_present
b.select_list(:id => 'entry_1000001').select('Ruby')
b.button(:value => 'Submit').click
b.button(:value => 'Submit').wait_while_present
Watir::Wait.until { b.text.include? 'Thank you' }
```

## 高级

### Window选择

```ruby
browser.window(:title => "annoying popup").use do
  browser.button(:id => "close").click
end
```

### 截屏

```ruby
# Save screenshot to file
browser.screenshot.save 'screenshot.png'

# Represent screenshot as PNG image string
browser.screenshot.png

# Represent screenshot as Base64 encoded string
browser.screenshot.base64
```

## 经验
有些内容是动态加载的，因此需要等待，但通过判断浏览器的title更快。


# [RAutomation](https://github.com/jarmo/RAutomation)
Windows上像Watir控制浏览器那样控制Windows程序。

通过title定位到程序，然后控制它。

# [xdotool](https://github.com/jordansissel/xdotool)

```bash
sudo apt install xdotool

xdotool key abc
xdotool key alt+Tab
xdotool type ''

xdotool search --name [name of the window] key [keys to press]
xdotool mousemove x y click 1
```

# [autokey](https://github.com/autokey-py3/autokey)
这个使用 Python3 写的最新版，这里是[旧版](https://github.com/autokey/autokey)。

通过设置触发快捷键，执行对应的 Python 脚本。

安装：

```bash
pip3 install autokey
# or, if you want the latest from this repository,
pip3 install git+https://github.com/autokey-py3/autokey
# or apt
sudo add-apt-repository ppa:troxor/autokey
sudo apt update
sudo apt install autokey-gtk
```


# [PyAutoGUI](https://github.com/asweigart/pyautogui)
可以模拟键盘鼠标的输入，所有操作系统通用！

## 安装与依赖
PyAutoGUI支持Python 2.x和Python 3.x

Windows：PyAutoGUI没有任何依赖，因为它用Python的ctypes模块所以不需要pywin32

```sh
pip3 install pyautogui
```

OS X：PyAutoGUI需要PyObjC运行AppKit和Quartz模块。这个模块在PyPI上的按住顺序是pyobjc-core和pyobjc

```sh
sudo pip3 install pyobjc-core
sudo pip3 install pyobjc
sudo pip3 install pyautogui
```

Linux：PyAutoGUI需要python-xlib（Python 2）、python3-Xlib（Python 3）：

```sh
sudo pip3 install python3-xlib
# screen capture
sudo apt-get scrot
sudo apt-get install python-tk
sudo apt-get install python3-dev
sudo pip3 install pyautogui
```

## 使用

```py
import pyautogui
screenWidth, screenHeight = pyautogui.size()
currentMouseX, currentMouseY = pyautogui.position()
pyautogui.moveTo(100, 150)
pyautogui.click()
#  鼠标向下移动10像素
pyautogui.moveRel(None, 10)
pyautogui.doubleClick()
#  用缓动/渐变函数让鼠标2秒后移动到(500,500)位置
#  use tweening/easing function to move mouse over 2 seconds.
pyautogui.moveTo(1800, 500, duration=2, tween=pyautogui.easeInOutQuad)
#  在每次输入之间暂停0.25秒
pyautogui.typewrite('Hello world!', interval=0.25)
pyautogui.press('esc')
pyautogui.keyDown('shift')
pyautogui.press(['left', 'left', 'left', 'left', 'left', 'left'])
pyautogui.keyUp('shift')
pyautogui.hotkey('ctrl', 'c')


distance = 200
while distance > 0:
    pyautogui.dragRel(distance, 0, duration=0.5) # 向右
    distance -= 5
    pyautogui.dragRel(0, distance, duration=0.5) # 向下
    pyautogui.draIn gRel(-distance, 0, duration=0.5) # 向左
    distance -= 5
    pyautogui.dragRel(0, -distance, duration=0.5) # 向上
```

当pyautogui.FAILSAFE = True时，如果把鼠标光标在屏幕左上角，PyAutoGUI函数就会产生pyautogui.FailSafeException异常。如果失控了，需要中断PyAutoGUI函数，就把鼠标光标在屏幕左上角。要禁用这个特性，就把FAILSAFE设置成False：

```py
import pyautogui
pyautogui.FAILSAFE = False
```
通过把pyautogui.PAUSE设置成float或int时间（秒），可以为所有的PyAutoGUI函数增加延迟。默认延迟时间是0.1秒。在函数循环执行的时候，这样做可以让PyAutoGUI运行的慢一点，非常有用。例如：

```py
import pyautogui
pyautogui.PAUSE = 2.5
pyautogui.moveTo(100,100); pyautogui.click()
```
所有的PyAutoGUI函数在延迟完成前都处于阻塞状态（block）。（未来计划增加一个可选的非阻塞模式来调用函数。）


# [WebDriverAgent](https://github.com/facebook/WebDriverAgent)
WebDriverAgent is a WebDriver server implementation for iOS that can be used to remote control iOS devices. It allows you to launch & kill applications, tap & scroll views or confirm view presence on a screen. This makes it a perfect tool for application end-to-end testing or general purpose device automation. It works by linking XCTest.framework and calling Apple's API to execute commands directly on a device.

自动控制 iOS 上的 Web 的。

# 参考
* http://watir.github.io/docs/home
* https://muxuezi.github.io/posts/doc-pyautogui.html
