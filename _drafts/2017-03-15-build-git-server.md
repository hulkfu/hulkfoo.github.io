---
layout: post
title: 搭建 git 服务器
---

```bash
$ sudo adduser git
$ su git
$ cd
$ mkdir .ssh && chmod 700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```


```bash
$ cd /opt/git
$ mkdir project.git
$ cd project.git
$ git init --bare
Initialized empty Git repository in /opt/git/project.git/
```

# 参考
- https://git-scm.com/book/it/v2/Git-on-the-Server-Setting-Up-the-Server
