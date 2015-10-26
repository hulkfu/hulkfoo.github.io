---
layout: post
title: Fusil：一个模糊测试框架
---

Fusil是一个python写的模糊测试框架，代码在[bitbucket](https://bitbucket.org/haypo/fusil)上托管。有一年多没有更新了，最新版本1.6。

# Agent
Agent将下面所有的模块连了起来，

## ApplicationAgent

### Application


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

### MTA

    """
    Mail (message) transfer agent:

     - send(): store messages in a mailbox of message category
     - live(): deliver messages in agent mailboxes
    """

## ProjectAgent

## Mangle


