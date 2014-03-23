---
layout: post
title: "使用rvm更好的管理你的gem"
date: 2013-09-05 01:42
comments: true
categories: [Ruby, 工具]
tags: [rvm, gem, ruby]

---

##rvm一个重要的功能 gemset
rvm作为一种方便的ruby安装和管理方式已经被大家所习惯和接受.但是rvm中另外一个重要的功能gemset使用率却不高.**gemset本身也有很大争议,很多人认为gem的管理交给bundler去做就足够好了,因此使用rbenv**

而我认为gemset相对来说还是比较方便.通过对gem的物理隔离,从而在批量操作gem时候更加放心.比如我曾经手贱使用bundle update升级了所有的gem,之后在给别人演示rails的时候突然惊讶的发现rails版本升到了4.0而吓出一身冷汗 （还没有接触过4.0,还好那次没有出什么丑 ）.


##gemset基本使用
先从gemset基本使用开始讲起<br>
我们平时使用rvm不指定gemset时,默认使用的是当前ruby版本下的default gemset,如`2.0@default`.如果想创建自己的gemset则使用

```
╭─Gavin@ligan  ~ ‹ruby-2.0.0›
╰─$ rvm gemset create rails4
 Gemset 'rails40' created.
```
这样我们就拥有了一个gemset "rails40",想要将该gemset设为默认gemset则使用

```
╭─Gavin@ligan  ~ ‹ruby-2.0.0›
╰─$ rvm use 2.0@rails4 --default
 Using /Users/Gavin/.rvm/gems/ruby-2.0.0-p247 with gemset rails4
```
以上命令可以简化为

```
$ rvm use 2.0@rails4 --create --default
```
我们还可以对gemset进行删除,切换,导入导出,清空,升级等操作,具体请查看[文档](http://rvm.io/)
<!-- more -->

##项目中的gemset使用
我们有了不同的gemset,目的就是为了区分不同项目中不同的环境.在不同的项目中我们可以通过rvmrc, gemfile, ruby-version等来使rvm自动切换到项目当前所需要的环境中.

rvm 1.11之前只支持在项目中添加.rvmrc的方式来切换rvm环境,而在1.22之后则建议使用

```
rvm --ruby-version use 2.0@rails4
```
这样rvm会同时在项目中创建.ruby-version以及.ruby-gemset文件来标识该项目所使用的ruby版本及gemset.同时**这种方式所生成的ruby版本信息同样被rbenv等工具所支持**

完成了以上工作之后我们再进入项目文件夹时,ruby版本及gemset就会自动切换到该项目的依赖环境下,相当于执行了一次`rvm use 2.0@rails4`,而退出该项目文件夹后则会回复默认ruby版本及gemset.

另外,有些时候我们所进入的项目中所包含的gemset信息本地并没有.这个时候我们希望能够自动创建对应的gemset,可以编辑`~/.rvmrc` 或者 `/etc/rvmrc`文件,加入

```
rvm_gemset_create_on_use_flag=1
```
从此在执行`rvm use`时候我们不需要加入`--create`参数便可以直接创建gemset

##rvm gemset体系
rvm中gemset除了default之外,还有一个global gemset.当前ruby版本下,所有的gemset都会继承global这个gemset下的所有gem.每个ruby版本又分别有一个global,default,以及自己创建的gemset.各ruby版本之间相互独立.上面还有一个所有版本公用的global gemset.

有了global这个gemset之后,我们就可以把一些每个gemset都要用,又不想重复安装的gem统统放入global这个gemset中,比如rake, pry, bundler之类.

可以通过编辑 ~/.rvm/gemsets/global.gems,加入

```
bundler
awesome_print
pry
```
这样每次安装新的ruby版本时,该版本ruby的global下都会自动安装好以上几个gem

##说了这么多我就是不想用gemset

```
echo "export rvm_ignore_gemsets_flag=1" >> ~/.rvmrc
```
这样就会忽略所有的gemset信息.永远的使用默认的gemset,其他项目中的gemset信息不会对你造成困扰.
