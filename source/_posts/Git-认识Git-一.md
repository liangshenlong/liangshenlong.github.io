---
title: Git-认识Git(一)
date: 2017-09-16 17:04:15
tags: [Git]
categories: [Git]
comments: true
---
Git是一款免费、 开源的分布式版本控制系统，最早由Linus创建，用于管理Linux内核开发，现已成为分布式版本控制的主流工具。
# Git的诞生
在Git诞生之前，Linux内核开发过程中使用BitKeeper来作为VCS。BitKeeper提供当时的一些开源VCS（如RCS、CVS）所不能提供的高级操作。然而，在2005年春天，当BitKeeper的所有者BitMover公司对他们的免费版BitKeeper加入了额外的限制时（一说是Linux社区一些人试图破解BitKeeper的协议被BitMover公司发现，BitMover公司要收回Linux社区的免费使用权），Linux社区意识到，使用BitKeeper不再是一个长期可行的解决方案。

于是，Git诞生了。
<!--more-->
# VCS历史
下面是一些具有里程碑意义的VCS系统
## 源代码控制系统（SCCS）
SCCS是UNIX上最初的几个系统之一，由M.J.Rochkind于20世纪70年代早期开发。
SCCS提供的数据存储中心称为“版本库”（repository），而这个基本概念一直沿用至今。SCCS同样提供了一个简单的锁模型来保证开发过程的有序。如果一个开发人员需要运行或者测试一个程序，他需要将该程序解锁并检出。然而，如果他想要修改某个文件，他则需要锁定并检出。
## 修订控制系统（RCS）
RCS是由Walter F.Tichy于20世纪80年代早期引入。RCS引入了双向差异的概念，来提高文件不同版本的存储效率。
## 并行版本系统（CVS）
CVS由Dick Grune于1986年设计并最初实现。4年后又被Berliner和他的团队融入RCS模型重新实现，这次实现非常成功。CVS变得非常流行，并且成为[开源社区](http:www.opensource.org)许多年的事实标准。

CVS引入了一个关于锁的新范式。之前的系统需要开发人员在修改某个文件时先锁定它，一个文件同时只允许一个人修改。CVS给予每个开发人员对自己的私有版本写的权利。除非两个开发人员尝试修改同一行，否则不同的改动可以自动合并。这使得开发人员可以并行编程。
## Subversion（SVN）
SVN于2001年问世，不像CVS，SVN以原子方式提交文件改动部分，并更好的支持分支。

## BitKeeper和Mercurial
BitKeeper和Mercurial淘汰了中心版本库的概念，数据的存储是分布式的，每个开发人员都拥有自己可共享的版本库副本。Git则是从这种端点对端点的模型继承而来。
# 安装Git
## 使用Linux上的二进制发行版
在大多数Debian/Ubuntu系统中，Git是很多包的集合，其中的每个包都可以根据需要独立安装
>git-arch

>git-cvs

>git-svn

如果你需要将一个项目从Arch、CVS、SVN转变为Git，或者返回来，则需要安装以上的包
>git-gui

>gitk

>gitweb

如果想要在图形界面或Web浏览器上浏览版本库，需要安装以上包

>git-email

>git-daemon-run

root权限执行apt-get来安装Git包
``` bash
$ sudo apt-get install git git-doc gitweb git-guigitk git-email
git-svn
```
## 获取源代码
https://github.com/git/git.git
## 构建和安装
``` bash
$ cd git
$./configure
$ mark all
$ mark install
```
要安装Git文档
``` bash
$ cd git
$ mark all doc
$ sudo mark install install-doc
```
## 在Windows上安装Git
https://git-scm.com/
