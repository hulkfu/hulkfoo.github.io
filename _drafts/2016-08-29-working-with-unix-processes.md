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

# 退出进程

## exit

退出，并且可以返回一个代码，而且at_exit里的代码将被执行。

```rb
# This will exit the program with the success status code (0).
exit
# You can pass a custom exit code to this method
exit 22
# When Kernel#exit is invoked, before exiting Ruby invokes any blocks
# defined by Kernel#at_exit.
at_exit { puts 'Last!' }
exit
```

## exit!
与exit相比就是不会执行at_exit里的代码。

## abort
程序运行失败时的退出。

```rb
# Will exit with exit code 1.
abort
# You can pass a message to Kernel#abort. This message will be printed
# to STDERR before the process exits.
abort "Something went horribly wrong."
# Kernel#at_exit blocks are invoked when using Kernel#abort.
at_exit { puts 'Last!' }
abort "Something went horribly wrong."
```

## raise
如果异常没有被rescue，那么将abort。


# fork
fork会返回两次，子进程返回nil，父进程返回子进程的pid。

```rb
if fork
  # parent
  puts "entered the if block"
else
  # child
  puts "entered the else block"
end

child_pid = fork do
  # 子进程
end
```
