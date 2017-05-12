---
layout: post
title: 去中心化
permalink: decentralize
---
去中心化，就是点对点通信，不需要经过第三方。

随着比特币的思潮，去中心化再次兴起，之前的 BitTorrent 已经势不可挡了。

# 网络

## [FreeNet](https://freenetproject.org/)

## [ZeroNet](https://zeronet.io/)
ZN 支持数据库。

使用比特币的加密技术及 BT 技术来存储网页的一个分布式 Web 平台。加密技术确保了只有作者才能修改，BT 技术实现了网页的存储。

每一个浏览网页的人同时也成了网页服务器。

如何工作的：

- 当你想打开一个新的页面时，它会从 BitTorrent 网络请求访问者的 IP。
- 首先会下载一个叫 content.json 的文件，它里面包含了所有其它文件的名字、Hash 及网站拥有者的签名。
- 之后使用网站的地址和拥有者签名验证 content.json。
- 通过验证后就下载其它文件（html, css, js ...)然后用 content.json 文件里对于文件名的 SHA512 验证它们。
- 每一个访问过的网站都会被存下来分享给其它人。
- 如果是网站的所有人（即有地址私钥的人），他可以更改网站，然后对 content.json 文件进行签名后发布到各个节点。当节点通过签名验证 content.json 后，就会下载新的网站并发布给其它节点。

所以，当我们打开 zeronet，其实是打开一个 BitTorrent 客户端，想迅雷那样的，只不过下载的是一个一个网站，而且每个网站都有所属。而且还打开一个 43110 的本机端口用来当本地应用的 UI。

可以参考 http://127.0.0.1:43110/Blog.ZeroNetwork.bit/?Post:99:ZeroChat+tutorial 做一个简单的入门。


## [MaidSafe](https://maidsafe.net/)

## [NameCoin]
Dot-Bit 域名。

# 文件

## [Resilio](https://www.resilio.com/)
就是之前的 btsync，这里[下载](https://www.resilio.com/platforms/desktop/)。
