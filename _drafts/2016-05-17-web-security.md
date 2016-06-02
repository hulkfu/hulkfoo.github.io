---
layout: post
title: Web安全
permalink: websecurity
---

Rails作为一个成熟的框架，已经默认提供了必要的安全机制，从而使得小白也能初始化出安全的站点。但安全还在人心，
只有在开发中时刻注意，才不会留下漏洞。

# Sessions

> HTTP is a stateless protocol. Sessions make it stateful.

Session机制是为了解决web的无状态特性，通过浏览器回传cookies来确认当前的session。

# Cross-Site Scripting (XSS)
跨站执行脚本，等于别人可以在你的网站页面上写自己的代码。十分恐怖！


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

```html
<%= csrf_meta_tags %>
```

# Redirection

# File Upload/Download

上传下载时，对文件名进行过滤，保证其安分呆在指定的目录里。

# 参考
* http://guides.rubyonrails.org/security.html
