---
layout: post
title: 栈溢出
permalink: stackoverfollow
---

栈在数据结构里是一种先进先出的结果，将这种数据结果应用到操作系统上，就有了操作系统最重要的系统栈。

系统栈管理着程序的调用情况，由每个函数的栈帧组成。每一个栈帧包括：

- 局部变量。
- 前栈的栈顶和栈底（实际只保存前栈的底部，顶部用堆栈平衡计算得到），本帧被弹出后恢复上一个栈帧。
- 函数返回地址，保存当前函数调用前的断点。

在 CPU 内部，与系统栈相关的主要寄存器：

- ESP(Extended Stack Pointer)，栈指针寄存器，其内存放一个指针，永远指向系统栈最上面一个栈帧的栈顶。
- EBP(Extended Base Pointer)，基础指针寄存器，期内的指针永远指向系统栈最上面一个栈帧的底部。
- EIP(Extended Instruction Pointer)，指令寄存器，永远指向下一条执行的指令。

所谓栈溢出，填充局部变量数据是按照其从低到高的是顺序，所以如果分配了10个byte，而进来12个，那么多的两个就会覆盖高位的已有数据，而不是新的内存空间。而这个已有数据可能就是存的函数的返回地址，覆盖重写成想要跳转的函数地址就完成了一次利用。

# shellcode
shellcode 是指广义上植入进程的代码，而不仅仅是获得 shell。exploit 是针对特定漏洞所要进程的操作，即漏洞利用，从而最终能让 shellcode 执行。


# 参考
- 《The Shellcoder's Handbook -- Discovering and Exploiting Security Holes》
