---
layout: post
title: HTTP 协议
permalink: http
---

HTTP是基于TCP的，前者属于应用层，后者是传输层，再往下是网络层和链路层。所以用TCP Socket也可以发出请求，只要按照格式发送。

HTTP 的特点：

- 无状态。
- 经第三方传输。

# 无状态
无状态，就是 user 的每次访问对 server 来说都是一个全新的访问。那么怎么才能把这次访问与之前的访问联系起来呢？比如我们登录一次帐号，在之后的访问中浏览器知道我们是谁的呀！

所以根据逻辑，user 要告诉浏览器我是谁，每次 http 请求都要告诉。这个信息一般存在 cookie 里，对于一次性的也可以存在访问 url 里。

cookie 是被保存在设备上的，user 会把 cookie 里内容读取并附在 HTTP 请求的 header 的 cookie 属性里的，server 设置 cookie 就在 header 里附上 Set-Cookie 属性。

但是如何确认 user 发送过来的 cookie 内容就是 server 传给它的呢，要不它不就能设置成别人的信息了吗？加密。server 发送过来的关键信息是加密的，并且有验证签名。用户一修改，就都不一样了，server 就能识别出。但是如果你的 cookie 被盗了，而且还没有过期，那么别人就可以以你的身份登录了。

# 经第三方传输
即不保证 user 和 server 之间的信息不能被别人看到，而且肯定会被别人看到，因为要经过路由传输。所以为了防止中间人攻击，user 和 server 就要自行加密了。
