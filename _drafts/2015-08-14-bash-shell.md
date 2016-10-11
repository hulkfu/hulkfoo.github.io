---
layout: post
title: Bash Shell
---

* $#，参数个数。
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

# 管道和exec


# 参考

* http://blog.longwin.com.tw/2014/09/bash-shell-scrip-argv-argc-2014/
