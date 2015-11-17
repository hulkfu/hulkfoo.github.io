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

# 参考

* http://blog.honeybadger.io/how-ruby-interprets-and-runs-your-programs/
