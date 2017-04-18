---
layout: post
title: Bundler RVM rbenv gem
---

还是从RVM迁移到了rbenv，主要导火索是用RVM安装新的Ruby版本是一直占用很大内存而且装不了。

转向rbenv才发现之前RVM默认做了很多，比如自动安装了bundler等gem。这样对于初学者当然是好的，可以
很快上手，可也隐藏了很多内部机制，导致很久后才明白bundler的原理，而这些对部署Rails服务是很重要的。

# [rbenv](https://github.com/rbenv/rbenv)


# RVM


# gem


# Bundler

## bundle install
--binstubs[=<directory>]
Creates a directory (defaults to ~/bin) and place any executables from the gem there. These executables run in Bundler's context. If used, you might add this directory to your environment's PATH variable. For instance, if the rails gem comes with a rails executable, this flag will create a bin/rails executable that ensures that all referred dependencies will be resolved using the bundled gems.


# 参考
- http://bundler.io/v1.14/man/bundle-install.1.html
