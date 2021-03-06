---
layout: post
title: 消失的代码的鬼魂：Ken Thompson的编译器后门
date: '2017-12-31 13:57'
---

众所周知，开源有一个好处：通过阅读源码，我可以知道这个软件有没有什么不可告人的勾当。找bug很难，看看软件有没有在偷偷的挖矿，这倒很简单。所以说，开源软件普遍带给我们一种安全可信任的感觉。

真的是源码可见就安全吗？万一你的编译器被做了手脚呢？那我就用gcc！我自己编译行了吧！实在不行，我把gcc的代码看一遍！

其实，这样的话也是不能保证程序的安全性的（就算你真的看的懂gcc的代码）。源码分析有意义，其基础在于源码能反映程序的行为。如果不能的话，分析源码是无法排除其安全隐患的。那你要说了，的确，编译器能做手脚。然而，我编译器的源码都看了，没问题呀。这样怎么不能保证安全呢？下面，我就要介绍一种方法，这种方法产生的编译器后面，在源码的任何地方都不会存在。

咱们先从最简单的情况开始：这是一个简单的编译器后门，如果程序里面有涉及登录的地方，就加一个后门。

```python
def compile(code):
  if (looksLikeLoginCode(code)):
    generateLoginWithBackDoor()
  else:
    compileNormally(code)
```

从编译器的源码里面，就能看出来这个后门。当然，任何人看到这则代码，一定会把写代码的人吃了。所有，这么做是不行的。那怎么做呢？

```python
def compile(code):
  if (looksLikeLoginCode(code)):
    generateLoginWithBackDoor(code)
  elif (looksLikeCompilerCode(code)):
    generateCompilerWithBackDoorDetection(code)
  else:
    compileNormally(code)
```

这个编译器高级了！不光会安装后门，如果他发现自己在编译编译器的话，就会把后门检测与向编译器插入后门检测的功能编译到新的编译器中。（好绕啊！）这样，新的编译器在编译代码的时候，就会把后门塞进去了。

我们把老编译器叫做编译器0，编译成的新编译器叫做编译器1。我们现在用编译器1编译一个*源代码里面没有后门*的编译器，生成编译器2。然而，由于编译器1里面有检测编译器以及加入使编译出的编译器在遇到登录代码时生成后门代码的逻辑，所以，用源码里面没有后门的编译器2编译的源码里面没有后门的程序，其二进制可执行文件中照样有后门。生成后门的代码在历史上存在过，然而现在消失了，不存在这个世界上的任何地方（当然，后门代码还以机器码的形式存在编译器里面。准确的说，后门不以任何形式存在在源代码里面）。然而，这个”消失的代码的鬼魂“，仍然在施加其影响力，如同一个幽灵一般。

这个方法，最早是[Ken Thompson提出的](/resources/reflections-on-trusting-trust.pdf)，他还真的实践了一把，在他写的Unix操作系统中埋了个后门，获得了他所有同事的登录密码（他更进了一部，把汇编反编译器也加了个后门，这样就算想看机器码也不是很容易了）。不过，这个方法也不是万能的，有办法可以发现这种形式的后门。方法很复杂，核心思路就是找这门语言的另一个编译器，比较他们的编译结果。毕竟，两个都有后门的概要小很多。[有人](/resources/countering-trusting-trust-through-diverse-double-sampling.pdf)就用这种方法，证明了gcc是安全的

本文是 [Mark C. Chu-Carroll的博文](http://scienceblogs.com/goodmath/2007/04/15/strange-loops-dennis-ritchie-a/)的读书笔记，伪代码就是抄他的。
