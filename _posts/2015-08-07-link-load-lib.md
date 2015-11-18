---
layout: post
title: 链接、装载与库
---

读《程序员的自我修养》的笔记。

正好在看Hack News时发现了[Crytial](https://github.com/manastech/crystal)，一个Ruby语法但是可以编译的语言。正好一起研究！

还是比较喜欢程序就一个可执行文件的样子，像exe，ipa，apk等打包发行后用户可以直接使用，不用担心脚本的依赖库。

Ruby的语法确实好，站在巨人的肩膀上，计算机科学会越来越好，想想都激动不已啊！

# 基础知识

  * 内存管理：映射
  * 线程管理

```
gcc hello.c
```

上面命令执行了四步：预处理、编译、汇编和链接。

# 编译

### 语法树


* http://llvm.org/
* http://liancheng.info/llvm-tutorial-cn/html/chapter-1.html
* http://dinosaur.compilertools.net/

# 链接

# 可执行文件
主要是Windows下的PE（Portable Executable）和Linux的ELF（Executable Linkable Format），它们都是COFF（Common File Format）格式的变种。

目标文件就是源代码进行编译后但未链接的文件，Windows下的.obj和Linux的.o文件，它们也安装可执行文件存储。

动态链接库.dll和.so文件及静态链接库.lib和.a文件，也是按照可执行文件格式存储的。

COFF引入了“段”的机制。

## ELF

主要包括头、代码段和数据段。

头，定了了文件的属性等。其中**段表**，描述文件中各个段的数组，它描述了文件中各个段在文件中的偏移位置及段的属性。

代码段，保存的就是程序指令，只读。一般为.text section。

数据段，保存程序的静态变量。有初始值的在.data里，没有初始值的在.bbs里。BBS（Block Started by Symbol）只是为了预留位置，并不分配。

分为代码段和数据段的原因：

* 共享代码段。
* 内存映射属性。代码段只读，数据段可读写。
* CPU需求。

可以使用readelf、objdump、nm等工具来查看elf文件。

一个例子：

```
int printf(const char* format, ...);

int global_init_var = 84;
int global_uninit_var;

void func1(int i) {
  printf("%d\n", i);
}

int main(void) {
  static int static_var = 85;
  static int static_var2;

  int a = 1;
  int b;

  func1(static_var + static_var2 + a + b);

  return a;
}
```

## Symbol —— 符号
有了符号，就不要使用决定的。

符号真是伟大的引入，多了这一层，就不用指定具体的指令和地址了。我们写符号，剩下的由编译器完成。

# 静态链接

* 把编译生成的多个.o文件链接成一个，相同的段内容拼在一起。
* 重新定位符号的地址。

测试例子：

```
/* a.c */
extern int shared;

int main() {
  int a = 100;
  swap(&a, &shared);
}

/* b.c */
int shared = 1;

void swap(int* a, int* b) {
  // 这个很妙
  *a ^= *b ^= *a ^= *b;
}

```

## 空间与地址分配
偏移量

## 符号解析与重定位

ELF文件里有重定位表


# 装载

程序要运行，那么相关的代码和数据就要装入到内存中。

装载的方式：

* 覆盖装入：程序员编程实现哪部分需要装入内容
* 页映射：操作系统自动管理内存页。

## 虚拟内存
一个进程只所以独立，就是因为有了自己的虚拟内存空间，好比一个人有了自己的家，这样才可有安全。当然家里可以住很多人，那就是线程。


# 动态链接

由于静态链接浪费空间，会在内存中保留多个程序副本。而且更新一个模块就需要重新链接全部。

动态链接的基本思想是推迟到运行时再链接。等于又多了一层，动态加载层。

Linux下的.so文件就是Shared Objects，在需要时被动态链接器加载。

在.data段里多了GOT（Global Offset Table），用来动态计算符号地址。

在程序运行时动态载入库：

* dlopen()
* dlsym()
* dlerror()
* dlclose()

# 堆栈

堆用来存储程序运行时长久数据，栈是函数被调用的运行时及临时变量。

## 堆

## 栈
一个函数调用所需要的维护信息，叫**堆栈帧**，内容包括：

* 函数的返回地址和参数
* 临时变量：包括函数的非静态局部变量以及编译器自动生成的其它临时变量。
* 保存的上下文：包括函数调用前后需要保持不变的寄存器。

有栈才有函数调用，要不怎么返回？

# 运行库
像Ruby、Python这样的动态语言有运行库很好理解，而C也是需要的，就是它在执行时需要的环境。

虽然被编译成了二进制，可是不需要实现每个printf函数，都调用共同的库即可了。

CRT（C Runtime Library ）的主要功能：

* 启动与退出：包括入口函数及入口函数所依赖的其它函数等。
* 标准函数
* I/O
* 堆
* 语言实现：语言中的一些特殊功能的实现。比如把C扩展成C++亦或Objective C
* 调式

是的，我们所看到的都在另一个更大的上面。

# 系统调用

# 感想

之前常感慨那些大牛怎么写出一个语言或操作系统，原来都是有理论基础的。

会写代码很简单，不就那几个循环和判断语句吗，可是理解深层次的知识就需要时间了。

