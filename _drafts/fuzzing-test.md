---
layout: post
title: 模糊测试
---

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