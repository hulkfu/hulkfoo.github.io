---
layout: post
title: 自动化操作
permalink: automation
---

自动化操作，就是模仿人的一系列操作。

# [Watir](http://watir.github.io/)

基于[Selenium](http://www.seleniumhq.org/)的浏览器自动控制接口，浏览器行为模拟器。

```rb
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

## 等待

## 经验
有些内容是动态加载的，因此需要等待，但通过判断浏览器的title更快。


# [RAutomation](https://github.com/jarmo/RAutomation)
Windows上像Watir控制浏览器那样控制Windows程序。


# [PyAutoGUI](https://github.com/asweigart/pyautogui)
可以模拟键盘鼠标的输入，所有操作系统通用！
