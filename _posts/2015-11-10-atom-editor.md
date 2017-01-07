---
layout: post
title: Atom 编辑器
permalink: atom
---

号称21世纪的编辑器用着真不是盖的。

它其实就是一个浏览器，Linux 下 **ctrl-shift-i** 就会调出它的调试窗口。

浏览器天生的适合做编辑器啊，因为有 css，能够很好的渲染出文字，有 html，可以显示界面的框架。看来
界面被 HTML 统一是在所难免了。

编辑器就是让我们自己去依赖的，能有实现自己任何的想法，所有有了：

- 快捷键，常用功能一键即达。
- 代码片段，不重复劳动，同时又是自己的编程积累。
- 自定命令，让编辑器执行自己想要的操作。

在 Atom 都有。

# keymaps
快捷键很非常重要，它关系到最直接的编程体验，最好的感觉是手不用离键盘，把所有搞定。

快捷键方案有很多家，没有最好的一种，都可以参考，只有适合自己的，毕竟是自己在用键盘写代码。

有人喜欢 vim 的编辑模式和命令模式间的频繁切换，而我喜欢 emacs 的直接，而且在 sh 里也是通用的。
我配合用的是 [atom-emacs-plus](https://github.com/aki77/atom-emacs-plus) 这个插件，然后自定义。下一步我会把这些整合到一起发个 package，这样用着就比较方便了。

快捷键有个原则：常用的要快。这也是符合信息论的。

对快捷键排个序：

- 直接按键。比如a,b,c,d等字母及符号的输入。
- tab。各种感觉前面的输入而自动的功能，比如缩进、snippets等。
- ctrl-字母。26个字母键就在手里。定义上下左右，翻页，undo，redo等常用的文本编辑功能，还有通向其他的命令界面，比如 fuzzy-finder、open-file、tree等。一定要保证这些被有效用完。
  - 而这里面又以能单手操作最便捷，我一般左手 ctrl 键。
- alt-字母。一般被软件 menu 占用，但都是浪费。改成自己需要的就好，我喜欢是一些命令。一般是对 ctrl-字母 的补充。
- ctrl-shift-字母。一般是对已有的 ctrl-字母 命令的扩展或反义。
- ctrl-alt-字母。这个一般被系统快捷键占用了。当然可以改了。
- ctrl-x ctrl-字母。一般是文件的命令，比如保存等。
- ctrl-x 字母串。一般是编辑器的命令，比如 split、选择pane等。

当然因人和键盘而异，我喜欢 emacs，用的是 HHKB2，Cap 改成了 Ctrl.
系统是 Mac 和 Linux 通用，所有尽量不使用 Cmd 键。

而且快捷键只有自己用了才知道是否合适，而且是在不短改进中的，追求的目的就是 —— 爽！不爽呢？那就改！它是你完全可控的，不用将就自己，放弃这个权利。这是一个学习的过程，把常用的换上来，不常用的换下去。

# Snippets
可以自定义代码片段，一个 trigger 健，后接 tab 键就出来了。
可以在 ~/.atom/snippets.cson 里定义，默认 Atom 也带了主流语言的 snippets，可以在已经安装的 packages 里查看。

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
- 当多个相同变量同时出现的话，就会被同时编辑。
- 多行用 """ 开始和结尾。
- 一个 snippet 有四部分：
  - 作用的代码
  - 名字
  - prefix
  - body


## 框架
编写 snippets 需要指定其作用域，就是用 css 选择器选出来。

- body
  - atom-workspace

# Package

Atom 使用一个 package 的过程：

- Atom starts up
- Atom starts loading packages
- Atom reads your package.json
- Atom loads keymaps, menus, styles and the main module
- Atom finishes loading packages
- At some point, the user executes your package command your-name-word-count:toggle
- Atom executes the activate method in your main module which sets up the UI by creating the hidden modal view
- Atom executes the package command your-name-word-count:toggle which reveals the hidden modal view
- At some point, the user executes the your-name-word-count:toggle command again
- Atom executes the command which hides the modal view
- Eventually, Atom is shut down which can trigger any serializations that your package has defined

active 方法是必须有的，它只在第一次调用时执行，初始化实例变量，包括UI等，还有注册 command 及 command 执行的事件。

```coffee
# Events subscribed to in atom's system can be easily cleaned up with a CompositeDisposable
@subscriptions = new CompositeDisposable

# Register command that toggles this view
@subscriptions.add atom.commands.add 'atom-workspace', 'your-name-word-count:toggle': => @toggle()
```

上面的代码用 atom.commands.add 在 'atom-workspace' 空间里注册了命令 "your-name-word-count:toggle"，
当触发这个命令（快捷键或命令面板）时，它会去执行 toggle 方法。

然后用 @subscriptions.add 将这个命令添加到命令事件集合里，
这样其他命令就可以知道它被执行，也可以在这里订阅其他命令。

虽然命令在 active 里注册了，但怎么触发 active 方法呢？在 package.json 里注册：

```js
"activationCommands": {
  "atom-workspace": "your-name-word-count:toggle"
}
```

这样 Atom 就会只加载这个方法名，只有当用户第一次触发时才调用 active。


参考官方的教程就很容易写一个package了。

我写完之后，最后一apm publish，竟然给发布上去了：[ascii-art](https://atom.io/packages/ascii-art)。

# 感想
Atom 是 github 搞出来的，而 github 最初是基于 Rails 的，可以看到 Atom 受 Rails 的影响很大，
比如 用 CoffeeScript 和 Less，这是 Rails 默认的 js 和 css 的高级语言。

# 参考
- http://flight-manual.atom.io/using-atom/sections/snippets/
- https://atom.io/docs/v1.1.0/hacking-atom-package-modifying-text
- http://xiaolai.li/2016/06/17/makecs-atom-advanced/
- http://flight-manual.atom.io/
