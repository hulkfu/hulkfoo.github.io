---
layout: post
title: 树莓派
permalink: raspberrypi
---
我使用的是 Raspberry 3B。

大部分的控制都可以通过 raspi-config 解决。

默认用户名 pi，密码是 raspberry，可以用 passwd 命令修改。

也就装机的时候需要显示器点几下，之后网络配置好后，就可以 ssh 和 vnc 连了。

# 装系统
到[官网](https://www.raspberrypi.org/downloads/)下载 NOOBS。

之后解压到 FAT 或 FAT32 格式的 tf 卡里的跟目录下。

开机就会自动引导安装了。

完整版的 NOOBS，装了 debian 系统，里面带的默认软件真不少啊！

Ruby、Python、Java、Node

# 连到隐藏 WiFI

简单的用 raspi-config 里 network-options 进行配置。

也可以编辑 wpa_supplicant.conf 文件：

```bash
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```

注意 ssid 和 psk 要用双引号。

```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev

# Whether to allow wpa_supplicant to update (overwrite) configuration
#
# This option can be used to allow wpa_supplicant to overwrite configuration
# file whenever configuration is changed (e.g., new network block is added with
# wpa_cli or wpa_gui, or a password is changed). This is required for
# wpa_cli/wpa_gui to be able to store the configuration changes permanently.
# Please note that overwriting configuration file will remove the comments from
# it.
update_config=1

# AP scanning/selection
# By default, wpa_supplicant requests driver to perform AP scanning and then
# uses the scan results to select a suitable AP. Another alternative is to
# allow the driver to take care of AP scanning and selection and use
# wpa_supplicant just to process EAPOL frames based on IEEE 802.11 association
# information from the driver.
# 1: wpa_supplicant initiates scanning and AP selection; if no APs matching to
#    the currently enabled networks are found, a new network (IBSS or AP mode
#    operation) may be initialized (if configured) (default)
# 0: driver takes care of scanning, AP selection, and IEEE 802.11 association
#    parameters (e.g., WPA IE generation); this mode can also be used with
#    non-WPA drivers when using IEEE 802.1X mode; do not try to associate with
#    APs (i.e., external program needs to control association). This mode must
#    also be used when using wired Ethernet drivers (including MACsec).
# 2: like 0, but associate with APs using security policy and SSID (but not
#    BSSID); this can be used, e.g., with ndiswrapper and NDIS drivers to
#    enable operation with hidden SSIDs and optimized roaming; in this mode,
#    the network blocks in the configuration file are tried one by one until
#    the driver reports successful association; each network block should have
#    explicit security policy (i.e., only one option in the lists) for
#    key_mgmt, pairwise, group, proto variables
# Note: ap_scan=2 should not be used with the nl80211 driver interface (the
# current Linux interface). ap_scan=1 is optimized work working with nl80211.
# For finding networks using hidden SSID, scan_ssid=1 in the network block can
# be used with nl80211.
# When using IBSS or AP mode, ap_scan=2 mode can force the new network to be
# created immediately regardless of scan results. ap_scan=1 mode will first try
# to scan for existing networks and only if no matches with the enabled
# networks are found, a new IBSS or AP mode network is created.
# ap_scan=1

network={
        scan_ssid=1
        ssid="Your Hidden SSID"
        psk="Your SSID's Password"
        key_mgmt=WPA-PSK
        }
```

# 开启 ssh 服务
默认是关闭的。

As of the November 2016 release, Raspbian has the SSH server disabled by default.

方法一：

It can be enabled manually from the desktop:

- Launch Raspberry Pi Configuration from the Preferences menu
-Navigate to the Interfaces tab
- Select Enabled next to SSH
- Click OK

方法二：

Alternatively, raspi-config can be used in the terminal:

- Enter sudo raspi-config in a terminal window
- Select Interfacing Options
- Navigate to and select SSH
- Choose Yes
- Select Ok
- Choose Finish

方法三：

Alternatively, use systemctl to start the service

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

方法四：

Enable SSH on a headless Raspberry Pi (add file to SD card on another machine)

For headless setup, SSH can be enabled by placing a file named ssh, without any extension, onto the boot partition of the SD card from another computer. When the Pi boots, it looks for the ssh file. If it is found, SSH is enabled and the file is deleted. The content of the file does not matter; it could contain text, or nothing at all.

If you have loaded Raspbian onto a blank SD card, you will have two partitions. The first one, which is the smaller one, is the boot partition. Place the file into this one.

# 开启 VNC 服务

```bash
# 完整版的 debian 系统已经有了，只需要配置开启
sudo apt-get update
sudo apt-get install realvnc-vnc-server realvnc-vnc-viewer

# 进行配置， Interfacing Options > VNC > Yes
sudo raspi-config
```

设置分辨率：

In the desktop menu, go to Preferences > Raspberry Pi Configuration and click the "Set Resolution" button. Or, from the terminal, run sudo raspi-config and choose Advanced Options > Resolution.

If you have a monitor attached, it will show a list of modes supported by the monitor. If you don't have a monitor attached, it will show a list of the most common modes.

If you want a mode that isn't listed, you will need to edit /boot/config.txt as described here: https://www.raspberrypi.org/documentation/configuration/config-txt/video.md

You will need to reboot your Pi for the new mode to take effect.


之后在客户端去[realvnc](https://www.realvnc.com/download/viewer/)下载 VNC Viewer。

# 开发

GPIO 图：

![Raspberrypi 3B GPIO](/img/rpi/pi3b_GPIO.png)

## Python
Python 使用 [RPi.GPIO 库](https://pypi.python.org/pypi/RPi.GPIO)

```python
import RPi.GPIO as GPIO

# 使用虚拟针脚，代码不会因板子的不同而做修改
GPIO.setmode(GPIO.BOARD)
# 使用实际的物理针脚
GPIO.setmode(GPIO.BCM)

# You need to set up every channel you are using as an input or an output. To configure a channel as an input:
# (where channel is the channel number based on the numbering system you have specified (BOARD or BCM)).
GPIO.setup(channel, GPIO.IN)

# To set up a channel as an output:
GPIO.setup(channel, GPIO.OUT)


# You can also specify an initial value for your output channel:
GPIO.setup(channel, GPIO.OUT, initial=GPIO.HIGH)

chan_list = [11,12]    # add as many channels as you want!
                       # you can tuples instead i.e.:
                       #   chan_list = (11,12)
GPIO.setup(chan_list, GPIO.OUT)

# 读一个针脚的电平，默认是此模式
GPIO.input(channel)

# output multi
chan_list = [11,12]                             # also works with tuples
GPIO.output(chan_list, GPIO.LOW)                # sets all to GPIO.LOW
GPIO.output(chan_list, (GPIO.HIGH, GPIO.LOW))   # sets first HIGH and second LOW

# 清除
GPIO.cleanup(channel)
GPIO.cleanup( (channel1, channel2) )
GPIO.cleanup( [channel1, channel2] )

# https://sourceforge.net/p/raspberry-gpio-python/wiki/Inputs/ 还有更多的设置
```


## Ruby
Ruby 有 [pi_piper 库](https://github.com/jwhitehorn/pi_piper)。

Pi Piper brings event driven programming to the Raspberry Pi's GPIO pins.

### GPIO

The GPIO pins (or General Purpose I/O pins) are the primary "do anything" pins on the Raspberry Pi. Reading inputs from them is as simple as:

```ruby
require 'pi_piper'
include PiPiper

watch :pin => 23 do
  puts "Pin changed from #{last_value} to #{value}"
end

#Or

after :pin => 23, :goes => :high do
  puts "Button pressed"
end

PiPiper.wait
```

Your block will be called when a change to the pin's state is detected.

When using pins as input, you can use internal resistors to pull the pin up or pull down. This is important if you use open-collector sensors which have floating output in some states.

You can set resistors when creating a pin passing a :pull parameter (which can be :up, :down or :off, which is the default).

```ruby
pin = PiPiper::Pin.new(:pin => 17, :direction => :in, :pull => :up)
```

This way, the pin will always return 'on' if it is unconnected or if the sensor has an open collector output.

You can later alter the pulling resistors using PiPiper#pull!

Additionally you can use pins as output too:

```ruby
pin = PiPiper::Pin.new(:pin => 17, :direction => :out)
pin.on
sleep 1
pin.off
```

please note, in the above context "pin" refers to the GPIO number of the Raspberry Pi.

### SPI

```ruby
PiPiper::Spi.begin do
  puts write [0x01, 0x80, 0x00]
end

#If you are using an operating system that supports /dev/spidev0.0 like the adafruit distro you can also write to the spi using PiPiper::Spi.spidev_out
# Example writing red, green, blue to a string of WS2801 pixels
PiPiper::Spi.spidev_out([255,0,0,0,255,0,0,0,255])
```

### Pulse Width Modulation (PWM)

PiPiper allow to use the hardware PWM channel of the bcm2835..... value should be between 0 and 1, clock between 0 and 19.2MHz, mode blaanced or markspace and range something greater than 0. Supported pin are : 12, 13, 18, 19, 40, 41, 45, 52, 53 but only 18 is on the header..

```ruby
pwm = PiPiper::Pwm.new pin: 18 #, range: 1024, clock: 19.2.megahertz, mode: :markspace, value: 1, start: false
pwm.value= 0.5
pwm.off # works with stop
pwm.on  # aliased start
```

apparently the clock is rounded to the next 2^n divider of 19.2MHz

我看了库的代码，没有操作 GPIO 的底层库，说是用的 [这个C库](http://www.airspayce.com/mikem/bcm2835/index.html)，可以没有见，待研究。

# 服务器

## [seafile](https://github.com/haiwen/seafile-rpi)

用 Python 写的，官方教程也清晰。

## samba 服务器
个人觉得如果只在家使用，samba 就够了。

# 感想
一个巴掌大的树莓派，完全能够满足上网、娱乐、学习、编程、搭服务等功能，真是太强大了！

而且接口也丰富，除了暴力出的 GPIO 可以进行嵌入式开发，还有 4 个 USB，蓝牙、WiFi、HDMI、声音、以太网等接口。

而电源只需要 2A 5V 的安卓接口充电器。

下来带 tf 卡，也就二百多块。

真的很适合进行二次开发，而且硬件也是开源的，真的量产也有保证。关键是在树莓派这个平台上的操作系统的完善支持，充分发挥了硬件的性能。

这么方便的系统，可以好好的坐开发，让我只能回忆之前玩 51、AVR、ARM 开发板的经历了。

当然由于运行着这么大的系统，RPi 并不适合适时系统，如果需要，可以选择 [arduino](https://www.arduino.cc/)，它使用的是 AVR 单片机。

# 参考
- http://raspi.tv/2017/how-to-auto-connect-your-raspberry-pi-to-a-hidden-ssid-wifi-network
- https://stackoverflow.com/questions/37312501/how-do-i-detect-and-connect-to-a-hidden-ssid-on-my-raspiberry-pi-3-raspbian
- https://w1.fi/cgit/hostap/plain/wpa_supplicant/wpa_supplicant.conf
- https://www.raspberrypi.org/documentation/remote-access/ssh/
- https://raspberrypi.stackexchange.com/questions/56421/how-to-increase-resolution-on-latest-raspbian-pixel-while-connected-to-vnc-clien
- https://eltechs.com/raspberry-pi-nas-guide/
