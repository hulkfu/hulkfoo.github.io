---
layout: post
title: rbenv和pyenv
---

从rvm迁移到了rbenv，发现pyenv原来是fork的rbenv。

在Mac下直接homebrew安装，再加个自动补全到.bash_profile里。

用着放心方便，下面拿rbenv来说明。

# 原理
通过在PATH环境变量里最开始插入shim可执行接口，来根据version来选择合适的ruby版本。像下面：

```bash
PATH=~/.rbenv/shims:/usr/local/bin:/usr/bin:/bin
```

# 使用
主要就是版本选择了。

1. rbenv shell，设定当前shell的ruby版本。
2. .ruby-version，在当前脚本的执行目录或上层目录里设置ruby的版本号，直到文件系统的根目录，找到
就会是以这个版本执行。
3. rbenv local，设定当前目录的ruby版本，会生成一个.ruby-version文件记录版本。
4. rbenv global，设定系统默认版本，在~/.rbenv/version里记录。


优先级从高到低。

~/.rbenv/shimes里的ruby可执行文件，就根据上面的参数或设置来选择版本。

# 问题
* 需要使用sudo的话，把shims下的bin文件软链接到 /usr/local/bin 目录下即可，否则sudo找不到。

rbevn install 2.3.1 error:

rbenv configure: error: C compiler cannot create executables

```
apt-get install build-essential
```

# 感想
Ruby和Python并不非要二选一，两者可以兼得啊！而且可以相互学习！

# 参考
* https://github.com/rbenv/rbenv
* https://github.com/yyuu/pyenv
