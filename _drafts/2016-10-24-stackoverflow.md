---
layout: post
title: 栈溢出
---

栈是在内存中从高位向低位加载数据的，先进先出。

但填充的数据却是按照其从低到高的是顺序，所以如果分配了10个byte，而进来12个，那么多的两个就会
覆盖高位的已有数据，而不是新的内存空间。而这个已有数据可能就是存的函数的返回地址，覆盖重写成
想要跳转的函数地址就完成了一次利用。

# 参考
- 《The Shellcoder's Handbook -- Discovering and Exploiting Security Holes》
