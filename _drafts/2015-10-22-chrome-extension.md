---
layout: post
title: Chrome 扩展插件
---

开发个东西，如果不能让开发者去扩展去玩耍，那么肯定不行。开发者的力量太强大了，他们用自己的双手改造着这个世界。



# 扩展访问页面

要知道，插件与所访问的网页直接是隔离的，即不能相互访问。如果需要访问，Content scripts。

> Content scripts are JavaScript files that run in the context of web pages.

## 静态注入
在manifest文件里声明：

```
{
  "name": "My extension",
  ...
  "content_scripts": [
    {
      "matches": ["http://www.google.com/*"],
      "css": ["mystyles.css"],
      "js": ["jquery.js", "myscript.js"]
    }
  ],
  ...
}
```

上面就将js和css里的代码注入到了访问http://www.google.com/*的页面里。

## 动态注入
使用chrome.tabs.executeScript，需要activeTab权限。

```
chrome.browserAction.onClicked.addListener(function(tab) {
  chrome.tabs.executeScript({
    code: 'document.body.style.backgroundColor="red"'
  });
});

// manifest
"permissions": [
  "activeTab"
],
```

既然能执行，就要有返回值啊：

```
chrome.tabs.executeScript(tabs[tab].id, {
    "code": 'document.getElementById("_Your_ID_Here_").className'
}, function (result) {
    console.log(result[0]);
});
```

如上，在回调函数里就可以取得返回值了，它是个数字。

也可以载入文件:

```
chrome.tabs.executeScript(null, {file: "content_script.js"});
```

这就相当于引用了。

## 运行环境
即使是注入的content script，其也是与网页隔离的。它们能操控DOM，但是不能访问网页里的js变量或函数，看起来就像只有一个静态页面。

同样网页也不能调用注入的js代码。

是的，他们只能通过同样DOM交互。

明白环境隔离很重要。等于是在写两个独立的页面，但静态的东西是一样的。而且content script里的js和extension里的js也是不通的。

* https://developer.chrome.com/extensions/content_scripts

# 参考

# 吐槽
同样是搜索引擎公司，google对世界做了多大的贡献啊，有Android、Chrome、Go、Angular等开源的项目，有让没有网线的地方上网的热气球，有眼镜，有自动驾驶的车。

反过来看看百度，就是搜索引擎，吃了这么多年，也没有见对哪怕中国做出什么贡献，它的存在可有可无。