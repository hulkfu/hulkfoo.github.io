---
layout: post
title: Ruby原理
---

对Ruby真的很感兴趣，研究起了如何实现的。

# ripper

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

从1.9版后才将AST编译成虚拟机bytecode。

## YARV(Yet Another Ruby Virtual Machine)

YARV是一个栈式的虚拟机。

会为每一个AST node创建一份bytecode。

## local table
每一份bytecode都有自己的local table，来记录变量的信息。

因为在bytecode中，变量没有标签，通过local table来标注：

* <Arg>   标准方法或块的参数。
* <Rest>  没有名字的数组参数，在参数中用 ( * ) 来表示的。
* <Post> 	在数组参数之后的标准参数。
* <Block> 通过&操作符传来的块参数。
* <Opt=i> 有默认值的参数。

## 例子

```ruby
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

```ruby
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

# 参考

* http://blog.honeybadger.io/how-ruby-interprets-and-runs-your-programs/
* Ruby Under a Microscope
* https://www.gnu.org/software/bison/
