---
layout: post
title: 两步验证
permalink: two-step-verification
---


# One Time Passwords

The basic protocol for one-time passwords is as follows:

Alice sends Bob a request for authentication.
Bob generates a random one-time password and sends it to Alice through a trusted channel.
Alice attaches the one-time password to her message and sends it to Bob.
Bob checks to see if the attached one-time password is the same that he sent to Alice.
Here, the security relies on the channel of communication for the one-time password being trusted, in the sense that Bob has reason to believe that Alice (and only Alice) will receive the one-time password. This is often achieved by using cellphones as the channel of communication. That is, Bob will send Alice the one-time password by sending a text message to, or calling, Alice's cellphone.

A different scheme, which removes the need for communicating the one-time password on every authentication attempt is:

Alice and Bob establish a secure and authenticated channel of communication.
Bob generates a random strong secret key and shares it with Alex, both storing it.
For future authentication requests, Alice and Bob pass the shared secret key plus a "counter" value to a cryptographic pseudo-random function and then extract a one-time password from the result
Alice sends the generated one-time password to Bob along with the authentication request.
Bob checks the sent one-time password with the one he generated using the same "counter". If they match, authentication succeeds.
This is the scheme that is implemented in HOTP and TOTP, which only differ by their choice of counter value.

## HMAC-Based One-Time Password Algorithm (HOTP)

## Time-Based One-Time Password Algorithm (TOTP)

# google authenticator
TOTP方式，通过扫描生成的二维码，获得双方共同持有的密钥，然后只要双方时间同步，就会根据

1. 密钥：服务器自动生成或用户输入。
2. 时间：手机上的时间和服务器上的时间。

生成相同的验证码。

用户从authenticator获得验证码，去网站验证。

所以关键是时间的同步，可以尝试修改服务器上的时间测试，就会验证失败了！

算法是公开的，密钥只有Server和客户端知道，甚至你自己都不知道。

所以网上也有很多类似的程序。


# 参考
* http://sahandsaba.com/two-step-verification-using-python-pyotp-qrcode-flask-and-heroku.html
* https://garbagecollected.org/2014/09/14/how-google-authenticator-works/
