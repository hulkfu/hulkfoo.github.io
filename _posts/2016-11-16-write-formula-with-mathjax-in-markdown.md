---
layout: post
title: 用Mathjax在Markdown里插入公式
---

这里使用Mathjax，配着kramdown来生成，代码高亮用rouge

首先安装kramdown：

```rb
gem install kramdown
```

然后配置_config文件：

```
markdown: kramdown
highlighter: rouge
```

在模板插入Mathjax的js代码：

```html
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

这样就可以用Lex语法写公式啦：

```lex
$$\sum_{i=1}^n \frac{1}{i^2} \quad and \quad \prod_{i=1}^n \frac{1}{i^2} \quad and \quad \bigcup_{i=1}^{2} R$$
```

生成：

$$\sum_{i=1}^n \frac{1}{i^2} \quad and \quad \prod_{i=1}^n \frac{1}{i^2} \quad and \quad \bigcup_{i=1}^{2} R$$

# 参考
- http://stackoverflow.com/questions/10987992/using-mathjax-with-jekyll
