---
layout: post
title: Rails Turbolink
permalink: turbolink
---

[Turbolink](https://github.com/turbolinks/turbolinks) 会阻止 html page 的全部 reload，这也是它的工作原理。

但有时候就会让一些 js 事件不能被触发，比如 page:load， 这是就要用 turbolink 的。

在 Turbolink 5 里就需要监听 ["turbolinks:load" 事件了](http://stackoverflow.com/questions/18770517/rails-4-how-to-use-document-ready-with-turbo-links):

```js
document.addEventListener("turbolinks:load", function() {
  // ...
})
```

或者使用 [jquery.turbolinks](https://github.com/kossnocorp/jquery.turbolinks)，对于 5，需要做些[修改](https://github.com/kossnocorp/jquery.turbolinks/issues/56#issuecomment-201374599):

```coffee
$.turbo.use('turbolinks:load', 'turbolinks:request-start')
```

所以简单的还是监听 turbolinks:load 事件，也能保证其他不需要每次加载的不加载。


## 禁止

在需要禁止的的 link，比如 anchor 的跳转，可以下面声明：

```html
data-turbolinks="false"
```
