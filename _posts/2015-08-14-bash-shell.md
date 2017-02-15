---
layout: post
title: Bash Shell
---

* $#，参数个数。
* $0, 脚本文件的名字
* $1，就是第一个参数，$2是第二个，依次类推。
* $@，所有参数。

```sh
#!/bin/bash


```

# 变量
等号赋值，等号两边没有空格，不加local就是全局变量，变量前加$取出变量值。

```sh
#!/bin/bash
a="hello"
echo "$a world"
```

# function

```sh
hi() {
  echo $1
}

hi Jack
```

对，在函数里，$1就变成了它的参数，而不是脚本的参数了，${10}表示第十个参数。


```sh
add() {
  sum=$(($1+$2))
  echo "$1 + $2"
  return $sum
}

add 1 2
echo "= $?"
```

返回值用$?获得。

# 控制

# 循环

# 管道和 exec

# 参考

* http://blog.longwin.com.tw/2014/09/bash-shell-scrip-argv-argc-2014/
