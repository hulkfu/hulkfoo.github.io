---
layout: post
title: Rails Cable
---

使用 WebSocket 来实现内容实时更新。

在没有 WebSocket 之前，用的是 Client 轮训查看 Server，有更新则拉取进行更新。

# 模型
sub/pub 模型，即订阅-发布。

## 概念

### channel
频道

# 参考
- http://guides.rubyonrails.org/action_cable_overview.html
