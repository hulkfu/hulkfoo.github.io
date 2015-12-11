---
layout: post
title: 读《Ruby Under a Microscope》
---

研究Ruby的实现，一方面能更明白的使用Ruby，另一方面能看看数据结构的现实使用。

# 引子

试试下面的代码：

```ruby
require 'ripper'
require 'pp'

code = "x > 1 ? 'foo' : 'bar'"

# 1. 分词（Tokenizing）
Ripper.tokenize code

# 2. 词法解析（Lexing）
pp Ripper.lex code

# 3. 语法解析（Parsing）
pp Ripper.sexp code

# 4. 编译成字节码（Compiling to bytecode）
puts RubyVM::InstructionSequence.compile(code).disassemble

```


# Tokenization and Parsing —— 生成语法树
将文本格式的代码转成抽象语法树。

Ruby使用[Bison](https://www.gnu.org/software/bison/)来完成，但自己写了寻找token的代码。

按照Bison的语法，Ruby的语法定义在parse.y文件里。

## Token


## LALR(Look-Ahead Left Reversed Rightmost Derivation)

* Look Ahead
* Left Reversed
* Rightmost Derivation

## AST(Abstract Syntaxre Tree)
抽象语法树


# Compilation —— 编译

从1.9版后才将AST编译成YARV虚拟机的bytecode。

编译的目的就是为了更快，1.9之前是直接执行的AST。虽然多了一层，但能够进行优化。

## YARV(Yet Another Ruby Virtual Machine)

YARV是一个栈式的虚拟机。

* EP(Envirenment Pointer)
* SP(Stack Pointer)
*

会为每一个AST node创建一份bytecode。

它是双栈的，一个是调用栈，另一个是数据栈。

顾名思义，调用栈存的是method、block、eval等调用帧。而数据栈里是每一个调用帧的参数、变量和返回值等数据。

### dynamic variable access
算是闭包，能够访问上一层的变量。

对人类来说可以很容易，但计算机实现起来，就要就个明确的顺序。

通过**special variables**即svar来控制变量的作用域的。
块和上一层同一个svar，所以块里能够访问外面的变量，这就是所谓的闭包。

而在《Ruby元编程》里提到的三个作用域门 —— class、module和def，把作用域分开了，即拥有了自己的svar。终于在这里找到原因啦！

使用**cref**指针来跟踪语法作用域。
Ruby uses the cref value to keep track of which lexical scope this
block belongs to

Specifically, Ruby uses
the cref value here to implement metaprogramming API calls, such as eval
and instance_eval .

```
----------
| method
----------
| params
----------
| receiver
-----------
```

## local table
每一份bytecode都有自己的local table，来记录变量的信息。

因为在bytecode中，变量没有标签，通过local table来标注：

* <Arg>   标准方法或块的参数。
* <Rest>  没有名字的数组参数，在参数中用 ( * ) 来表示的。
* <Post> 	在数组参数之后的标准参数。
* <Block> 通过&操作符传来的块参数。
* <Opt=i> 有默认值的参数。

例子：

```
2.0.0-p247 :001 > code=<<CODE
2.0.0-p247 :002"> puts 2+2
2.0.0-p247 :003"> CODE
 => "puts 2+2\n"

2.0.0-p247 :004 > puts RubyVM::InstructionSequence.compile(code).disasm
== disasm: <RubyVM::InstructionSequence:<compiled>@<compiled>>==========
0000 trace            1                                               (   1)
0002 putself          
0003 putobject        2
0005 putobject        2
0007 opt_plus         <callinfo!mid:+, argc:1, ARGS_SKIP>
0009 opt_send_simple  <callinfo!mid:puts, argc:1, FCALL|ARGS_SKIP>
0011 leave            
 => nil
```


```
2.0.0-p247 :005 > code = <<CODE
2.0.0-p247 :006"> 10.times do |n|
2.0.0-p247 :007"> puts n
2.0.0-p247 :008"> end
2.0.0-p247 :009"> CODE
 => "10.times do |n|\nputs n\nend\n"
2.0.0-p247 :010 > puts RubyVM::InstructionSequence.compile(code).disasm
== disasm: <RubyVM::InstructionSequence:<compiled>@<compiled>>==========
== catch table
| catch type: break  st: 0002 ed: 0006 sp: 0000 cont: 0006
|------------------------------------------------------------------------
0000 trace            1                                               (   1)
0002 putobject        10
0004 send             <callinfo!mid:times, argc:0, block:block in <compiled>>
0006 leave            
== disasm: <RubyVM::InstructionSequence:block in <compiled>@<compiled>>=
== catch table
| catch type: redo   st: 0000 ed: 0009 sp: 0000 cont: 0000
| catch type: next   st: 0000 ed: 0009 sp: 0000 cont: 0009
|------------------------------------------------------------------------
local table (size: 2, argc: 1 [opts: 0, rest: -1, post: 0, block: -1] s3)
[ 2] n<Arg>     
0000 trace            256                                             (   1)
0002 trace            1                                               (   2)
0004 putself          
0005 getlocal_OP__WC__0 2
0007 opt_send_simple  <callinfo!mid:puts, argc:1, FCALL|ARGS_SKIP>
0009 trace            512                                             (   3)
0011 leave                                                            (   2)
 => nil
```


## 控制指令
实现起来还是靠jump跳转指令，CPU移移指针就OK啦！

### if else
翻译成汇编：执行条件语句，如果false则跳到false的代码，否则继续执行，然后跳过false的代码，跟着的
是false代码。

### break

在break时，会插入throw指令，通过catch table，迭代着找到break指令所指的地址。

```
10.times do |n|
  puts n
  break
end
puts "continue from here"
```

### return
return其实和break用的同一throw，前者的参数是1,后者的是2.

像rescue、ensure、retry和next等也是靠catch table实现的。

catch table就比如ARM的中断列表。

### send
一旦Ruby找到准备执行的方法，它使用发放分发去执行这个调用。

这需要准备方法的参数，压入一个新的帧到YARV的内部栈里，改变YARV的内部寄存器从而能够执行目标方法。

Ruby的方法被分成了11种类型：

* ISEQ  A normal method that you write using Ruby code, this is the
most common method type. ISEQ stands for instruction sequence.
* CFUNC  Using C code included directly inside the Ruby executable,
these are the methods that Ruby implements rather than you. CFUNC
stands for C function.
* ATTRSET  A method of this type is created by the attr_writer method.
* ATTRSET stands for attribute set.
* IVAR  Ruby uses this method type when you call attr_reader . IVAR
stands for instance variable.
* BMETHOD  Ruby uses this method type when you call define_method
and pass in a proc object. Because the method is represented internally
by a proc, Ruby needs to handle this method type in a special way.
* ZSUPER  Ruby uses this method type when you set a method to be
public or private in a particular class or module when it was actually
defined in some superclass. This method is not commonly used.
* UNDEF  Ruby uses this method type internally when it needs to
remove a method from a class. Also, if you remove a method using
undef_method , Ruby creates a new method of the same name using the
* UNDEF method type.
* NOTIMPLEMENTED  Like UNDEF, Ruby uses this method type
to mark certain methods as not implemented. This is necessary, for
example, when you run Ruby on a platform that doesn’t support a
particular operating system call.
* OPTIMIZED  Ruby speeds up some important methods using this
type, like the Kernel#send method.
* MISSING  Ruby uses this method type if you ask for a method object
from a module or class using Kernel#method and the method is missing.
* REFINED  Ruby uses this method type in its implementation of
refinements, a new feature introduced in version 2.0.

# 对象和类

## RObject

* RBasic: 指向class的指针klass和boolean类型的flags
* numiv: number of instance values
* ivptr: instance values pointer，指向一个array

ruby/include/ruby/ruby.h

```
struct RBasic {
    VALUE flags;
    const VALUE klass;
}

struct RObject {
    struct RBasic basic;
    union {
	struct {
	    long numiv; /* only uses 32-bits */
	    VALUE *ivptr;
            void *iv_index_tbl; /* shortcut for RCLASS_IV_INDEX_TBL(rb_obj_class(obj)) */
	} heap;
	VALUE ary[ROBJECT_EMBED_LEN_MAX];
    } as;
};
```

## Generic Objects
比如Fixnum，直接在RBasic的klass里存数据，然后在flag里设置FIXNUM_FLAG。

generic_iv_tbl stores instance variables for generic objects.

## RClass

flags and klass are the same RBasic values that every Ruby value contains.

m_tbl is the method table, a hash whose keys are the names, or IDs, of
each method and whose values are pointers to the definition of each
method, including the compiled YARV instructions.

iv_index_tbl is the attribute names table, a hash that maps each instance
variable name to the index of the attribute’s value in each RObject
instance variable array.

super is a pointer to the RClass structure for this class’s superclass.
iv_tbl contains the class-level instance variables and class variables,
including both their names and values.

const_tbl is a hash containing all of the constants (names and values)
defined in this class’s scope. You can see that Ruby implements iv_tbl
and const_tbl in the same way: Class-level instance variables and con-
stants are almost the same thing.

Ruby uses origin to implement the Module#prepend feature. I’ll discuss
what prepend does and how Ruby implements it in Chapter 6.

Ruby uses the refined_class pointer to implement the new experimental
refinements feature, which I’ll discuss further in Chapter 9.

Finally, Ruby uses allocator internally to allocate memory for new
instances of this class.


A Ruby class is a Ruby object that also contains method definitions
and attribute names.

Ruby must store the attribute names in RClass , value 在RObject。因为每个实例变量的名字都是一样的。

所以RClass里有method table和属性名。

## rb_classext_struct

## module
module也是RClass，但是没有iv_index_tbl，因为它不能建实例，所以没有实例属性的名字。
也没有refined_class and allocator。

A Ruby module is a Ruby object that also contains method defini-
tions, a superclass pointer, and a constants table.

Ruby creates a copy of the RClass structure
for the Professor module and uses it as the new superclass for Mathematician .
Ruby’s C source code refers to this copy of the module as an included class.
The superclass of the new copy of Professor is set to the original superclass
of Mathematician , which preserves the superclass, or ancestor chain. Figure 6-2
summarizes this somewhat confusing state of affairs.



### Module#include
当一个类include一个模块时，复制这个模块，把它插入成了这个类的superclass，即插入到这个clas和其原先的super class之间。
再include，就再在这个类头上加。

模块不能用"<"来继承，但是可以被另一个模块include，这样就是继承了。在实现上，用的就是RClass的super class指针。

当类include的模块里添加或删除了新的方法时，类的实例里方法也会跟着改变。因为虽然是复制的模块，但是它们指向同一个方法列表。

这是书中152页的说法，但是在2.0.0p247 测试，是会影响的。

~~但是如果被包含的模块里包含的模块里有方法改变，这个类是感受不到的，也是因为类引用的是被复制的模块，而模块也是引用的被复制的模块。~~

很好理解，因为被模块包含的模块，也在继承链中，所以会找到的。

### Module#prepend

Ruby 2.0的新功能。

使用prepend来include，那么这个module就会插入到当前继承链的第一位，当前class成了其父类。
这样就会先调用module里的方法了。

就是靠rb_classext_struct中的origin实现的，拷贝一个RClass，用origin指着，然后把原来RClass里
方法去掉。


## 方法查找
因为Ruby有一个清晰的继承链，包括module和class，所以通过current class和super class就可以完成
迭代循环查找。

set current
class to
receiver
look through
method table in
current class
not
found
method
found?
set current
class to
superclass of
current class
found
call method

## 常量查找



### 继承链查找

### 语法域查找

* nd_next：上一个语法作用域
* nd_clss：当前语法作用域

Somehow Ruby allows you to search up the “namespace
chain” to find constants. This behavior is called using lexical scope to find
a constant.

set cref to
current lexical
scope
look through
constant table
for cref’s class
constant
found?
not
found
set cref parent
lexical scope
found
done

先找scope，再找parent。

search through
lexical scope chain
for each scope’s
class, check for
autoload
search through
superclass chain
for each superclass,
check for autoload
call
const_missing

# Hash

Ruby的Hash存在st.c和st.h里，是a public domain general purpose hash table package written by Peter Moore @ UCB.

### RHash

```
struct st_table_entry {
    st_index_t hash;
    st_data_t key;
    st_data_t record;
    st_table_entry *next;
    struct list_node olist;
};
```

The C code used by the Object class’s implementation of the hash
method gets the C pointer value for the target object—that is, the
actual memory address of that object’s RValue structure. This is essen-
tially a unique ID for that object.



# 块和闭包

rb_block_t

ruby/vm_core.h

```
typedef struct rb_control_frame_struct {
    const VALUE *pc;		/* cfp[0] */
    VALUE *sp;			/* cfp[1] */
    const rb_iseq_t *iseq;	/* cfp[2] */
    VALUE flag;			/* cfp[3] */
    VALUE self;			/* cfp[4] / block[0] */
    VALUE *ep;			/* cfp[5] / block[1] */
    const rb_iseq_t *block_iseq;/* cfp[6] / block[2] */
    VALUE proc;			/* cfp[7] / block[3] */

#if VM_DEBUG_BP_CHECK
    VALUE *bp_check;		/* cfp[8] */
#endif
} rb_control_frame_t;

typedef struct rb_block_struct {
    VALUE self;			/* share with method frame if it's only block */
    VALUE *ep;			/* share with method frame if it's only block */
    const rb_iseq_t *iseq;
    VALUE proc;
} rb_block_t;
```

* A pointer to a snippet of YARV code instructions—the iseq pointer
* A pointer to a location on YARV’s internal stack, the location that was
at the top of the stack when the block was created—the EP pointer

When creating the new block structure, Ruby copies the current value
of the EP into the new block. In other words, Ruby saves the location of the
current stack frame in the new block.

Here’s how Sussman and Steele
defined the term closure in 1975:

In order to solve this problem we introduce the notion of a closure
[11, 14] which is a data structure containing a lambda expression,
and an environment to be used when that lambda expression is
applied to arguments. 2

Blocks are the combination of a function and the environment to use when
calling that function.
This structure meets Sussman and Steele’s definition of a closure:
•
•
iseq is a pointer to a lambda expression—a function or code snippet.
EP is a pointer to the environment to be used when calling that lambda
or function—that is, a pointer to the surrounding stack frame.

Ruby saves
only references to data on the stack—that is, the VALUE pointers. For simple
integer values, symbols, and constants such as nil , true , or false , the refer-
ence is the actual value. However, for all other data types, the VALUE is a
pointer to a C structure containing the actual data, such as RObject . If only
the VALUE references go on the stack, where does Ruby save the structures?
In the heap

## lambda

The lambda (or proc ) keyword converts a block into a data value.

lambda是把块包成数据，这样可以作为参数传输。

# 其它实现

## JRuby


## Rubinius

在报错时，Rubinius能够显示的更深。

The Rubinius kernel  This is the part of Rubinius written in Ruby.
It implements a lot of the language, including the definitions of many
built-in, core classes, such as String and Array . The Rubinius kernel
is compiled into bytecode instructions that are installed onto your
computer.
The Rubinius virtual machine  The Rubinius virtual machine is writ-
ten in C++. It executes the bytecode instructions from the Rubinius
kernel and performs a range of other low-level tasks, such as garbage
collection. The Rubinius executable contains a compiled, machine-
language version of this virtual machine.


C++实现的底层。

使用Ruby实现相关的Ruby类，通过llvm编译成机器语言。

Like JRuby, Rubinius is an alternative implementa-
tion of Ruby. Much of Rubinius’s internal source code
is written in Ruby itself instead of in only C or Java.
Rubinius implements built-in classes, such as Array ,
String , and Integer , just as you would—with Ruby code!

# 垃圾回收

## MRI

Lazy Sweeping


## JRuby & Rubinius

•
•
Note 	
Instead of using a free list, they allocate memory for new objects and
reclaim memory from garbage objects using an algorithm called copying
garbage collection.
They handle old and young Ruby objects differently using generational
garbage collection.
They use concurrent garbage collection to perform some GC tasks at the
same time that your application code is running.


copying garbage collection

Instead of using a free list to track available
objects, copying garbage collectors allocate memory for new objects from
a single large heap or memory segment. When that memory segment is
used up, these collectors copy only the live objects over to a second memory
segment, leaving the garbage objects behind. The two segments are then
swapped, immediately reclaiming all of the memory from the garbage
objects. (Rubinius and the JVM both use complex algorithms based on this
original idea.)

While the semi-space algorithm is an elegant way to manage memory,
it is somewhat memory inefficient. It requires the collector to allocate
twice as much memory as it actually uses because all of your objects might
remain active and could be copied into the second heap. The algorithm is
also somewhat difficult to implement because when the collector moves live
objects, it also has to update references and pointers to them internally.


### semi-space algorithm



# 感想
看这本书，也是对Ruby精妙数据结果设计的学习。这真是计算机科学的精华！

# 参考

* http://blog.honeybadger.io/how-ruby-interprets-and-runs-your-programs/
* Ruby Under a Microscope
* https://www.gnu.org/software/bison/
