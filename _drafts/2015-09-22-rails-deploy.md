---
layout: post
title: Rails Deploy
---

不得不说，Rails的发布是个坎，它涉及到自动部署、Nginx配置、缓存、代理等，需要对这个Web系统概念有一个明确的了解，否则真的搞不定。

往往遇到搞不定的情况，就是自己的步子迈的太大，理解这一步的基础还没有打好。比如还没有把rails应用手动部署成功，就想着用capistrano自动部署。

手动部署，就要对Rack、Rake、Bundler有所了解，然后还需要Linux、Nginx、数据库等的知识。



# 参考
* https://www.phusionpassenger.com/library/walkthroughs/deploy/ruby/
* http://bundler.io/rationale.html
