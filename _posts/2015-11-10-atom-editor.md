---
layout: post
title: Atom 编辑器
---

号称21世纪的编辑器用着真不是盖的。

# Snippets
可以自定义代码片段，一个 trigger 健，后接 tab 键就出来了。可以在 ~/.atom/snippets.cson 里
定义，默认 Atom 也带了主流语言的 snippets，可以在已经安装的 packages 里查看。

比如下面我定义写 markdown 的 snippet：

```json
'.source.gfm':
  'title':
    'prefix': '---'
    'body': """
    ---
    layout: ${1: post}
    title: ${2: title}
    ---

    $3
    """
```    

可以总结：

- 要填内容的地方用 $1,$2, 等指代，如果有默认值，用 $1{1: default} 来表示。
- 多行用 """ 开始和结尾。
- 一个 snippet 有四部分：
  - 作用的代码
  - 名字
  - prefix
  - body



# Package
参考官方的教程就很容易写一个package了。

我写完之后，最后一apm publish，竟然给发布上去了：[ascii-art](https://atom.io/packages/ascii-art)。

# 参考
- http://flight-manual.atom.io/using-atom/sections/snippets/
- https://atom.io/docs/v1.1.0/hacking-atom-package-modifying-text
