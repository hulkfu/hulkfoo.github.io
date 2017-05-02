---
layout: post
title: 区块链
permalink: blockchain
---

区块链是另一个世界，新的疆土。在这个数据是分布式的，没有中心，而且是可回溯的。

# 什么？
一个共识，Trusted Machine。去中心化，透明，受监管，才不作恶。

我相应如今的区块链如同二十年前的互联网，当我还在后悔小时候学会上网的第一件事是用搜狐搜到机器猫津津有味的看而不是注册些牛逼的域名时，那么如今我已长大，是否能把握住这次机会呢？

区块链也让互联网如同制造业，能够创造“实物”，而不只是改进生产关系。

# 加密技术
先进的加密技术是区块链技术的磐石。而这个加密技术就是 RSA。

RSA 实现了公钥加密，私钥解密的业务。这样就可以把自己的公钥放出去，一般就是地址，可是是钱包的地址或是网页的地址，总之代表一个资源，可以让其他人发现和找到。当需要修改这个资源时，比如用这个地址完成一笔交易，修改这个网页等等，就需要私钥来认证。

那么怎样才算认证通过了呢？

那比特币的加密算法[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 举例。

# 应用

## 比特币 —— bitcoin

## 以太坊 —— ehtereum

## DApp

Decentralized App

## [Namecoin](https://namecoin.org)
.bit 的域名。

## [ZeroNet](https://zeronet.io/)
使用比特币的加密技术及 BT 技术来存储网页的一个分布式 Web 平台。加密技术确保了只有作者才能修改，BT 技术实现了网页的存储。

每一个浏览网页的人同时也成了网页服务器。

如何工作的：

- 当你想打开一个新的页面时，它会从 BitTorrent 网络请求访问者的 IP。
- 首先会下载一个叫 content.json 的文件，它里面包含了所有其它文件的名字、Hash 及网站拥有者的签名。
- 之后使用网站的地址和拥有者签名验证 content.json。
- 通过验证后就下载其它文件（html, css, js ...)然后用 content.json 文件里对于文件名的 SHA512 验证它们。
- 每一个访问过的网站都会被存下来分享给其它人。
- 如果是网站的所有人（即有地址私钥的人），他可以更改网站，然后对 content.json 文件进行签名后发布到各个节点。当节点通过签名验证 content.json 后，就会下载新的网站并发布给其它节点。

所以，当我们打开 zeronet，其实是打开一个 BitTorrent 客户端，想迅雷那样的，只不过下载的是一个一个网站，而且每个网站都有所属。

可以参考 http://127.0.0.1:43110/Blog.ZeroNetwork.bit/?Post:99:ZeroChat+tutorial 做一个简单的入门。


# 参考
- http://www.8btc.com/ethereum-price
- http://teahour.fm/2016/01/19/talk-with-jan-about-ehtereum.html
