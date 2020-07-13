+++

title = "用 Drafts 解决少数需求考"
description = "我已经堕入似水流年的虚无，除了隐隐作痛的伤痕，毫无欢娱"
date=2020-07-13

[taxonomies]
categories = ["瞎搞"]
tags = []

[extra]
author = "Clarke"
relative_posts=[]

+++

当你有少数的需求时，大部分的开发者都不会站在你的这边[^1]，比如在选择静态博客的框架时，将目光投向了 [Zola](https://www.getzola.org) ，看着其它框架，比如 [Hugo](https://gohugo.io)，[Jekyll](https://jekyllrb.com) 与大量编辑器比如 Emacs, Vim, VSCode, Drafts 等等有着大量的互动和扩展，Zola 就像一个生下来就被抛弃的孩子[^2]。当你走向少有人走的路，看到甚至连简单的需求都无法满足，深深怀疑是不是自己选择的错的时候，应该明白，也许我可以动手实现。

解决私人的少数简单需求与软件工程不同，需求分析仅仅需要考虑自己片面的需求，设计以及实现过程也是粗犷的野蛮的。这样似乎就很简单，比如在学术生活中，从师兄那儿学到可以利用 PowerPoint 改变画布大小进行海报的设计。"活用" 似乎就是这些野蛮需求解决的核心，这也导致了通往罗马的道路不止一条。

如何在 Drafts 的 Markdown 中插入引文是一个少数的需求，同时是一个例子，直接用 Word 或者 LaTeX 进行写作不好吗？但是 Drafts 是一个灵感采集的工具，有时候就是想随时随地写作，一个任性的需求。但是这甚至已经算是老生长谈的话题，仅仅在少数派就有大量的实现方法，比如[使用 Zotero 在 Markdown 中优雅地处理参考文献](https://sspai.com/post/60825)。甚至有一个明确的流程：

1. 查找文献， 在 Markdown 中插入这几篇文献的 Citation Key
2. 通过 Pandoc 转化。

第一步似乎并不困难，如果你用 Zotero， Better BibTex 就提供了这样的接口：  通过访问 `http://127.0.0.1:23119/better-bibtex/cayw` 就能调出 Zotero 的查询界面，然后只需要找到后回车，就能得到 Citation Key。在 Drafts 里甚至更简单，因为 Drafts 提供了 ShellScripts 的接口，只要得到结果后，再调用 Drafts 的 `插入文本` 的功能就能完成。又或者，你甚至发现了 Alfred 的插件 [Zothero](https://github.com/deanishe/zothero)，甚至可以在 Zotero 关闭的时候进行工作。

但是，如果在移动端的情况呢？Ulysses 提供了解决的一种[方法](https://ulysses.app/answers/reference-managers) —— 利用 Endnote 的 Citation 以及 Word 的 Endnote 插件。因为价格以及多平台的原因，并不是那么想迁移到 Endnote，但是Zotero 的同步并不包括 Citation Key，而生活中还是经常有用 iPad 进行写作的时候，如何做到呢？ Drafts 提供了 [交叉链接](https://docs.getdrafts.com/docs/drafts/cross-linking) 的功能，只要将一条文档作为参考文献的引用库，在正文中链接到这个引用库后，通过脚本实现插入文献即可。

{{imgur(id="a/ZQP7E9b", title="Drafts 插入参考文献")}}

那么如何更新这个参考文献的引用库呢？好在 Zotero 还是提供了 Web 端查找引文，用书签添加引文等功能，决定好要插入的引文时，复制，调用 Drafts 的脚本，算出 Citation Key，完成插入。

第二步似乎也不难，在桌面端，有万能的 Pandoc，不想频繁输入冗长的命令也可以写一个 Drafts 的脚本，将 Darfts 里的内容导出，然后调用 Pandoc 进行转化，最后打开。但是，移动端怎么办？我的答案是利用 Citation.js 和 MathJax 就能实现引文和数学公式环境预览，而成品还是放在桌面端处理吧。

{{imgur(id="a/0v64inm", title="Drafts 转化 PDF")}}

也不是什么难事嘛，难的还是如何把工具转化为真正的动力。技术永远都会革新，当看到前几日的做实验机器人的时候，已经对企图继续攻读博士绝望了，因果推断对于知能而言是迟早的事情，很看好KG，但是我仍然很迷茫，什么才是透过技术不变的东西，什么才是我应该去追逐的事情。


[^1]:或者可以选择 Emacs，我早已经投奔了，elisp 写起来可舒服了，而且这里的老哥各个说话好听。

[^2]:事实上 Zola 有一个较为活跃的社区，只是和 Hugo 和 Jekyll 比起来穷酸了许多。


