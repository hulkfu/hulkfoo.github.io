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



# Tokenization and Parsing
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


# Compilation



# 参考

* http://blog.honeybadger.io/how-ruby-interprets-and-runs-your-programs/
* Ruby Under a Microscope
* https://www.gnu.org/software/bison/
