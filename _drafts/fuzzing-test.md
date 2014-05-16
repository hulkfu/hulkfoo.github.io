---
layout: post
title: 模糊测试
---

# 调试器

* 打开要调试的进程。
* 附加调试进程跟踪调试进程的状态。

# windows安装pydbg

1. 编译环境。这里用[mingw](http://www.mingw.org/)。
2. pydasm。它基于[libdasm](http://code.google.com/p/libdasm/)。进入libdasm\pydasm目录编译安装：

```
python setup.py install **build --compiler=mingw32**
```

如果装了VS就不用选择编译器了，否则会报*error: Unable to find vcvarsall.bat*的错误。

3. pydbg。[下载代码](https://github.com/OpenRCE/pydbg)，拷贝到python安装目录的lib文件夹里，并移除里面自带的不好用的pydasm.pyd文件。

# Valgrind


# Python的Fusil

# Ruby的FuzzBert
```ruby
def trap_child_exit
  trap(:CHLD) do
    while_child_exits do |exitval|
      pid = exitval[0]
      status = exitval[1]
      data_ary = @data_cache.delete(pid)

      handle({
        id: data_ary[0],
        data: data_ary[1],
        pid: pid,
        status: status
      }) if status_failed?(status)

      start_new_child
    end
  end
end
```

主要就是靠这段代码来捕捉子进程的异常，然后handle记录的。

相比Fusil，FuzzBert它没有CPU监控，进程监控也不完善，最明显的就是不能自动关闭进程。所有还是比较适合测试Ruby的库等接口，而不适合测试外面的应用。