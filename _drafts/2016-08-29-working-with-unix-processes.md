---
layout: post
title: 使用Unix进程
---

这是读《Working with Unix Processes》的笔记，它的例子使用Ruby写的。

# 系统调用
Unix系统分用户空间和系统空间，系统调用正是它们间的桥梁。

* Section 1: General Commands
* Section 2: System Calls
* Section 3: C Library Functions
* Section 4: Special Files

```sh
man 2 getpid
```

Ruby的 Process.pid就是映射到了getpid(2):

```rb
puts Process.pid
```

pid代表着运行的进程，文件描述代表着打开的文件。

在Unix中，一切都是文件，包括硬件、网络链接socket和文件。为了区分，不是文件的“文件”用资源代替好了。

每一个进程打开的资源都有一个唯一的ID，内核正是通过这个ID来跟踪这个进程使用的资源的。

```rb
passwd = File.open('/etc/passwd')
puts passwd.fileno  #=> 8

hosts = File.open('/etc/hosts')
puts hosts.fileno  #=> 9

passwd.close
null = File.open('/dev/null')
puts null.fileno  #=> 8
```

而且释放的ID会被下次接着重新使用。被关闭的资源没有了ID，内核也不会继续跟踪了。

## 标准流

资源ID最小是3,那么0到2呢？

```rb
puts STDIN.fileno   #=> 0
puts STDOUT.fileno  #=> 1
puts STDERR.fileno  #=> 2
```

# 运行环境
每一个进程从父进程继承运行环境。

```bash
$ MESSAGE='wing it' ruby -e "puts ENV['MESSAGE']"
```

```rb
ENV['MESSAGE'] = 'wing it'
system "echo $MESSAGE"
```

上面bash和ruby间可以传递变量。

# 进程有参数
每个进程都有ARGV参数，意识是arguments vector，保存着从命令行传给当前进程的参数。

```sh
$ cat argv.rb
p ARGV.class
p ARGV
$ ruby argv.rb foo bar -va
Array
["foo", "bar", "-va"]

# did the user request help?
ARGV.include?('--help')
# get the value of the -c option
ARGV.include?('-c') && ARGV[ARGV.index('-c') + 1]
```

# 进程有名字
在Ruby里，是$PROGRAM_NAME或$0。
