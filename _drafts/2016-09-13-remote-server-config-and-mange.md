---
layout: post
title: 远程服务器配置和管理
---
Ruby 确实适合干管理，瞧Chef和Puppet都是用它写的。

每次有了新服务器，上去一顿配置和安装环境是否很重复，所以聪明的程序员有了新的软件。

# Chef

不用安装Ruby的原因是它自己嵌入了一个Ruby，这样也能不与外界环境冲突。

概念:

* recipe（食谱）：要执行的任务清单
* cookbook（烹饪书）：更有条例化的管理食谱，比如将模板分出去。
* workstation：就是你手头的工作机，在上面写cookbook，并能管理网络。
* Chef server：保存cookbook和nodes的信息的中心节点。
* node：一个工作的节点，运行cookbook里的任务。
* knife: 与Server通信的工具，需要从Server下载名为knife.rb的配置文件。
* chef-client：在node上跑一次，就会拉取Server上的manual-list，并执行，是被动调用的，不在后台常驻。

## 远程管理node的步骤

1. 首先安装[chef-dk](https://downloads.chef.io/chef-dk/)。

2. 在搭建Chef Server，可以先用[Chef官方的测试](https://learn.chef.io/manage-a-node/ubuntu/set-up-your-chef-server/)，下载knife.rb和user.pem文件（注意：那个pem是User的，而不是Organizations的）。

3. node要能够ssh上，那么就可以“做菜”了：

```sh
# 第一次在node上运行bootstrap，
# 1.自动安装chef-client
# 2.下载能够链接Server的私钥到/etc/chef/client/pem
# 3.执行chef-client
knife bootstrap ADDRESS --ssh-user USER --sudo --identity-file IDENTITY_FILE --node-name node1 --run-list 'recipe[learn_chef_apache2]'

# 之后就只需要执行chef-client了
knife ssh ADDRESS 'sudo chef-client' --manual-list --ssh-user USER --identity-file IDENTITY_FILE
```

## 感想

可见流程是workstation上传任务到Server，再从workstation用knife让node去执行（其实就是在node上运行
chef-client），亦可自己登录到node上执行‘sudo chef-client’。

这让我想起了部署Rails的Capistrano，它也算是远程服务器管理，但是主要为发布网站，更新代码设计的。所以不需要中间的Server，
直接ssh到node上，然后拉取代码，执行本地的任务。这样也就没有管理界面了。

Chef和Capistrano合作应该不错吧，[瞧早有人发现了：）](http://www.nickhammond.com/deploying-your-chef-infrastructure-with-capistrano/).

Chef确实时候安装软件等，然后管理很多node。

# Puppet


# 参考

* https://learn.chef.io/tutorials/
