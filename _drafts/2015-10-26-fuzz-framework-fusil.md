---
layout: post
title: Fusil：一个模糊测试框架
---

Fusil是一个python写的模糊测试框架，代码在[bitbucket](https://bitbucket.org/haypo/fusil)上托管。有一年多没有更新了，最新版本1.6。

现在还在看Message，如果一个Agent读完了，另一个怎么办呢？

# Agent
Agent将下面所有的模块连了起来。

分析代码一个方法就是如果不这样那么怎么写？

如果下面的不继承自Agent，因为要使用同一个mta来通信，那么就要把重复的代码写一遍。而继承不就是为了抽象重复代码吗？

Agent如其名，代理，主要处理核心业务之外的逻辑，如发送消息。

这些Agent都是测试的一部分，在Application里组装了起来。


## ApplicationAgent

### Application
这个是关键

    """
    Application class is responsible to execute a fuzzer using Fusil:
     - parse the command line
     - setup logging
     - create the project
     - execute the project
     - cl

在main()调用ruanProject：

```



def runProject(self):
    """
    Load, execute and destroy the project.
    """
    # Load project
    self.project = Project(self)
    try:
        # Create the project
        self.setupProject()
        self.registerAgent(self.project)

        # Execute project
        self.executeProject()
        self.unregisterAgent(self.project)
    finally:
        # Destroy project
        self.project.destroy()
        self.project = None
```

可见首先定义了project实例变量，就是初始化自己。


会把所以Agent实例收集到AgentList里：

```
    def __init__(self):
        self.agents = AgentList()
        ApplicationAgent.__init__(self, "application", self, None)
        self.setup()
```

### MTA

    """
    Mail (message) transfer agent:

     - send(): store messages in a mailbox of message category
     - live(): deliver messages in agent mailboxes
    """

## ProjectAgent

### Project

```
    """
    A fuzzer project runs fuzzing sessions until we get enough successes or the
    user interrupts the project. Initialize all agents before a session starts,
    and cleanup agents at the session end.

    Before a session start, the project sleeps until the system load is under
    50% (may change with command line options).
    """
```

## Mangle


