---
layout: post
title: 编程语言实现模式
---

读《编程语言实现模式》，Terence Parr著，一直对“The Pragmatic Programmers”系列的书存在好感，因为写书的人都是实践派，里面有真货。

# Lexer
词法解析器。

提前代码中的有用部分，输出到数据结构，数组或树，方法下一步处理。

# Parser
语法解析器。

在Lexer的过程中，判断语法是否正确，正确的话按语义执行操作。如解析出[a,b,c]，就把a,b,c存成数组。



* https://github.com/chriswailes/RLTK
* https://github.com/manastech/crystal