---
layout: post
title: Linux Bash 编程
permalink: bash
---

Bash 是大部分 Linux 的默认 shell，适合做胶水。Bash 编程是控制 Linux 系统的，而其它语言如C、Python 亦或 Ruby 编程是实现业务逻辑的。

用 Bash 的好处是快，它没有复杂的包引用，而且是直接对 Linux 系统的操作，只不过是把日常用的命令行打包起来。

虽然看上去语法有些怪异，但也特别好学。

# 语法
和其他语言最不同的是每一个关键字都要有空格分离，都是命令。

## 变量

* 给变量赋值等号两边不能有空。因为有空的话，会被当作命令执行。
* 等号赋值，不加 local 就是全局变量
* 取变量值时前面要加$。
* 取函数的返回值则用$()夹着函数名或用 `cmd`。

默认变量，在函数里的话指的是函数参数，否则指的是命令行的参数：

* $#，参数个数。
* $0, 脚本文件的名字
* $1，就是第一个参数，$2是第二个，依次类推。
* $@，所有参数。


```sh
#!/bin/bash
a="hello"
echo "$a world"
```

## 函数

一般函数：

```sh
hi() {
  echo $1
}

hi Jack
```

对，在函数里，$1就变成了它的参数，而不是脚本的参数了，${10}表示第十个参数。


返回值用$?获得：

```sh
add() {
  sum=$(($1+$2))
  echo "$1 + $2"
  return $sum
}

add 1 2
echo "= $?"
```


将命令行输入的参数转成数组：

```bash
cmds=($@)

for arg; do
  echo $arg
done
```

## 控制


```bash
if [ -f $target_filepath ]
then
  echo "$1 already exists and remove it."
  rm $target_filepath
else
  # other
fi
```

可以看到，即使是 if 语句，本质上也是一个一个串联的命令，命令直接是空格分割，没有我们常见的括号等语法样式。

[ -f $file ] 其实等于 test -f $file

## 循环

```bash
for file in "./"/*
do
  realpath=`realpath $file`
  filename=`basename $file`
  echo $realpath
done
```

```bash
while [ $i -le $match ]
do
  # todo
done
```


## 管道和 exec

```bash
# <<< 将后面的内容转成标准输入
at now +10 minutes <<< "rm -rf /tmp/tobedeleted"
```


# 常用指令

## xargs

用管道时，能把显示的内容当作参数分别传入，而不是作为整个文本传入。


# 参考

* http://blog.longwin.com.tw/2014/09/bash-shell-scrip-argv-argc-2014/


* http://linuxconfig.org/bash-scripting-tutorial
