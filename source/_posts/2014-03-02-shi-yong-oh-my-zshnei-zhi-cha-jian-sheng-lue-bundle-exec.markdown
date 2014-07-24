---
layout: post
title: '使用oh-my-zsh内置插件省略bundle exec'
date: 2014-03-02 12:09:23 +0800
comments: true
categories: Ruby
tags: [ruby, bundler]
description: 在oh-my-zsh中省略输入烦人的  bundle exec.
---
##bundle exec

在使用bundler管理的项目中,为了统一使用环境,经常要使用`bundle exec`来执行项目中的各种命令.


比如在某些项目中使用的是rake 0.9而系统中安装有rake 10时候如果不加上`bundle exec`就会提示错误.这时候就需要执行

```
bundle exec rake my:task
```

以前使用rvm时可以通过每个项目不同的gemset来统一项目中不同版本的gem.换了rbenv之后完全依靠bunder来管理gem之后每次执行`bundle exec`就有点麻烦了.

有一些工具可以代替`bundle exec`,比如[bundler-exec](https://github.com/gma/bundler-exec)

但是如果正好使用oh-my-zsh的话,则有一个更简单的方法

##oh-my-zsh

oh-my-zsh现在已经几乎成为了标配,这里不做介绍,如果你没有使用它,可以到这里查看[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

只需要在`.zshrc`中的找到这一行

```
plugins=(git bundler)
```

plugins中加入bundler,绝大部分情况下就不需要再手动敲上`bundle exec`了.

[源码看这里](https://github.com/robbyrussell/oh-my-zsh/blob/master/plugins/bundler/bundler.plugin.zsh#L9)
