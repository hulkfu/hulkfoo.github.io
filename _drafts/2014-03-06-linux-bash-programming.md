---
layout: post
title: Linux Bash 编程
---

bash是大部分Linux的默认shell，就是shell，没有复杂的功能，适合做胶水。

## 语法规则

* 给变量赋值等号两边不能有空。
* 取变量值时前面要加$。
* 取函数的返回值则用$()夹着函数名。

## 常用全局变量

* $# 命令行参数的个数
* $@ 所有命令行参数
* $1 第一个命令行参数，依次类推


所以：

将命令行输入的参数转成数组，可以

```bash
cmds=($@)

for arg; do
  echo $arg
done
```

## function

```bash
function hi {
  echo hi
}

ahi() {
  echo another hi
}

echo $(hi), baby!
```

* http://linuxconfig.org/bash-scripting-tutorial
