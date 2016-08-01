---
layout: post
title: 两步验证
permalink: two-step-verification
---

# 一次性密码（One Time Passwords）

一个典型过程：

* Client输入手机号，向Server申请验证码
* Server发送一个随机码
* Client获得验证码，并输入提交
* Server验证Client输入的验证码是否是刚才自己发送的那个

所以，OTP的安全建立于通信信道的安全，一般是手机。

还可以有另外一种方案，不用每次都发送一次性密码：

* Client和Server间建立一个安全通道，Server生成一个密钥，并发送给Client。这样C和S就共有同一密钥了。
* 之后Client再发起认证请求时，它就和Server同时对密钥加一个“计数器”值，然后传给一个密码生成函数，生成一次性密码。
* 这样Client把生成的一次性密码发送给Server，如果和Server生成的一样就认证成功。

可以看出：

* 一次性密码生成函数开源且相同
* 需要保护的是第一次建立安全链接时生成的那个密钥
* 每次改变密钥的算法同步且开源

这就是两步认证了，四两拨千斤，用短的一次性密码（一般6位数）来隐藏复杂的密钥。即使一次性密码被监听盗取，有效期也只有
很短的时间（比如30s）。

所以根据那个“盐”生成方式的不同就有两种算法：

## HMAC-Based One-Time Password Algorithm (HOTP)

> OTP=Truncate(Hash(k∥C))

这里的C是Client和Server记忆的常数，一般初始化为0,然后每次认证后累加。

Hash采用的是 HMAC-SHA-1 算法。

Truncate将结果截短，生成几位的密码。

然后这个方案有问题：

1. 这一次生成的一次性密码是在下一次一次性密码生成时才过期。这样如果本次这个密码被黑客得到，就可以在很长的
窗口期里使用。
2. C的同步问题。


所以有了下面基于时间的一次性密码：

## Time-Based One-Time Password Algorithm (TOTP)
就是将上面的累加的C缓存了基于时间的T。

> T=（Tcurrent−T0）/X

* Tcurrent: 当前的时间。
* T0: 初始化的时间，一般是0.
* X: 一次性密码的有效时间，一般是30.


# google authenticator
TOTP方式，通过扫描生成的二维码，获得双方共同持有的密钥，然后只要双方时间同步，就会根据

1. 密钥：服务器自动生成或用户输入。
2. 时间：手机上的时间和服务器上的时间。

生成相同的验证码。

用户从authenticator获得验证码，去网站验证。

所以关键是时间的同步，可以尝试修改服务器上的时间测试，就会验证失败了！

算法是公开的，密钥只有Server和客户端知道，甚至你自己都不知道。

所以网上也有很多类似的程序。

# 相关库

* [python-totp](https://github.com/sahands/python-totp)
* [rotp](https://github.com/mdp/rotp)


# 参考
* http://sahandsaba.com/two-step-verification-using-python-pyotp-qrcode-flask-and-heroku.html
* https://garbagecollected.org/2014/09/14/how-google-authenticator-works/
* http://blog.miguelgrinberg.com/post/two-factor-authentication-with-flask
