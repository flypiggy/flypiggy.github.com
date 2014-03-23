---
layout: post
title: "zsh alias with parameters"
date: 2013-01-06 22:01
comments: true
categories: [Linux]
tags: [zsh]

---

以前经常加一些alias来偷懒减少输入。今天在使用pandoc转换markdown文档时也想使用alias简化输入。如

```
pandoc list.markdown -t html -o list.markdown.html
```

想要省掉一堆-t html -o 什么的参数，最好是我只需要输入

```
pd 文件名
```
就可以完成转换，zsh在alias中加参数的方法稍微有些不一样，查看zsh文档的一堆废话后（csh中怎么写的，zsh和csh不一样，balabala），在.zshrc中加入

```
pd() { pandoc "$1" -t html -o "$1".html; }
```
之后试验一下

```
╭─gavin@Gavin-PC  ~/ligan/Training/demo ‹ruby-1.9.3›
╰─$ ls
list.md
╭─gavin@Gavin-PC  ~/ligan/Training/demo ‹ruby-1.9.3›
╰─$ pd list.md
╭─gavin@Gavin-PC  ~/ligan/Training/demo ‹ruby-1.9.3›
╰─$ ls
list.md  list.md.html
```
大功告成
