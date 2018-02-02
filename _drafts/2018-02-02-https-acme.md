---
layout: post
title: HTTPS 证书自动管理环境
---

Automatic Certificate Management Environment (ACME)，用于管理证书，如申请、更新等。

而目前使用的较多的是免费的 [Let's encrypt](https://letsencrypt.org)，[这里](https://letsencrypt.org/docs/client-options/)有它的官方的 ACME 软件列表。

这里主要介绍 [cerbot](https://github.com/certbot/certbot) 和 [acme.sh](https://github.com/Neilpang/acme.sh)，都很好用，也是 Github 上排名最靠前的两名。

相比 cerbot 比较方便。


# cerbot

完全文档在[这里](https://certbot.eff.org/docs)，下面是针对 Ubuntu 17.10 的简单说明。

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx
```

If you're serving files for that domain out of a directory on that server, you can run the following command:

```bash
$ sudo certbot --authenticator webroot --installer nginx
```


If you're not serving files out of a directory on the server, you can temporarily stop your server while you obtain the certificate and restart it after Certbot has obtained the certificate. This would look like:

```bash
$ sudo certbot --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"
```

Automating renewal

```bash
$ sudo certbot renew
```

用 crontab 自动更新，需要 root 帐号下的：

```bash
sudo su
crontab -e
```

加入：

```bash
# 每条凌晨三点更新
0 3 * * * certbot renew --pre-hook "service nginx stop" --post-hook "service nginx start"
```


# acme.sh
