---
title: Git-远程版本库(四)
date: 2017-09-17 23:25:15
tags: [Git]
categories: [Git]
comments: true
---
克隆版本库只是共享代码的第一步。此外，还必须对版本库之间进行关联，为数据交换建立路径。Git通过远程版本库为这些版本库建立连接。
远程版本库（remote）是一个引用或句柄，通过文件系统或网络指向另一个版本库。一旦远程版本库建立，Git就可以使用pull或push在版本库之间传输数据。
Git使用远程追踪分支（remote-tracking branch）跟踪其他版本库中的数据。版本库中的每个远程追踪分支都作为远程版本库中特定分支的一个代理。要集成本地修改和远程追踪分支对应的远程修改，可以建立一个本地追踪分支（local-tracking branch）来建立集成的基础。
# 版本库概念
## 裸版本库和开发版本库
一个Git版本库要么是裸（bare）版本库，要么是一个开发版本库。
开发版本库用于常规的日常开发，在工作目录中提供检出当前分支的副本，相反一个裸版本库没有工作目录，并且不应该用于正常开发。裸版本库可以简单的看做是.git目录的内容。

一般协作开发时都使用裸版本库作为权威库，开发人员从裸版本库中clone、fetch、push。
<!--more-->
```bash
#创建裸版本库
$ git clone --bare old new

$ git init --bare
```
## 版本库克隆
git clone命令会创建一个新的Git版本库。在使用git clone命令时，原始版本库中存储在refs/heads/下的本地开发分支，会成为新的克隆版本库中refs/remotes/下的远程追踪分支。原始版本库中refs/remotes/下分支不会克隆。

Git用默认的fetch refspec配置默认的origin远程版本库：
```bash
fetch = +refs/heads/*:refs/remotes/origin/*
```
建立这个refspec预示你要通过从原始版本库中抓取变更来持续更新本地版本库。在这种情况下，远程版本库的分支在克隆库中是可用的，只要在分支名前面加上origin/前缀。

## 远程版本库
git remote命令可以创建、删除、修改和查看远程版本库。引入的所有远程版本库都记录在.git/config中。
```bash
$ git remote add <name>
$ git remote rename  <oldname> <newname>
$ git remote remove <name>
$ git remote set-url <name> <newurl>
```
>git fetch： 从远程版本库中抓取对象以及相关元数据

>git pull： 跟git fetch类似，但是合并修改到相应的本地分支。

>git push： 推送对象及相关元数据到远程版本库。

>git ls-remote：显示一个给定的远程版本库引用列表

## 追踪分支
* 远程追踪分支与远程版本库相关联，专门用于追踪远程版本库中每个分支的变化。
* 本地追踪分支与远程追踪分支相配对。它是一种集成分支，用于收集本地分支和远程追踪分支中的变更。
* 任何本地的非追踪通常称为本地特性分支。
* 远程追踪分支与远程版本库相关联，专门用于追踪远程版本库中每个分支的变化。

本地特性分支保存在refs/heads中，远程追踪分支保存在refs/remotes中。
```bash
$ git branch -a
* dev
  master
  test
  remotes/origin/dev
  remotes/origin/master

```
如上图，origin/dev、origin/master为远程追踪分支，dev、master为本地追踪分支，test为本地特性分支。

# 远程版本库示例
## 创建权威版本库
权威版本库一般都是裸版本库。
```bash
$ git clone --bare local local.git
Cloning into bare repository 'local.git'...
done.


```
之前有说过裸版本库类似.git文件，如下图：

![创建裸版本库](/images/bare_clone.jpg)

## 添加origin远程库
前面创建的local版本库是git init产生的，因此它没有远程版本库。而clone出来的版本库会自动创建一个origin远程版本库。

给local添加远程版本库
```bash
$ git remote add origin E:/GitRepository/local.git

$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        symlinks = false
        ignorecase = true
        hideDotFiles = dotGitOnly
[remote "origin"]
        url = E:/GitRepository/local.git
        fetch = +refs/heads/master:refs/remotes/origin/master
```
origin为远程版本库名称，名字可自定义。

但是此时版本库local中还未建立远程追踪分支。使用git remote update创建远程追踪分支。
```bash
#查看分支
$ git branch -a
  dev
* master

#update
$ git remote update
Fetching origin
From E:/GitRepository/local
 * [new branch]      master     -> origin/master

#查看所有分支
$ git branch -a
  dev
* master
  remotes/origin/master

```
>普通的git remote update命令会导致在这个版本库中的每个remote都被更新，会从每个remote指定的版本库中检查并抓取新提交。除了一般地更新所有remote之外，可以限制只从一个remote获取更新：

>$ git remote update remote_name

>此外，在最初添加远程版本库时，使用-f选项将会立即对该远程版本库执行fetch：

>$ git remote add -f origin repository

## 获取更新

进行开发工作并提交本地版本库之后，需要推送变更到远程版本库。但是有可能在我们开发期间，别人推送了一系列的提交。此时需要在提交前使用git pull获取版本库更新。

git pull操作有两个根本步骤，第一个步骤：拉取（fetch），第二个步骤合并（merge）或变基（rebase）。
### 拉取步骤
拉去步骤中，Git先定位远程版本库，默认origin版本库，
origin版本库信息保存在.git/config中。
```bash
$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        symlinks = false
        ignorecase = true
        hideDotFiles = dotGitOnly
[remote "origin"]
        url = E:/GitRepository/local.git
        fetch = +refs/heads/master:refs/remotes/origin/master
```
Git会使用remote配置中fetch值操作。因此，将抓取远程版本库中的的refs/heads/master分支。

Git会把拉取到的新提交放在远程追踪分支origin/master分支中。

### 合并或变基步骤
git pull的第二个步骤，Git会执行合并（merge）（默认），或变基（rebase）操作。

git merge会合并远程追踪分支的内容到本地追踪分支中。Git通过配置文件中的banch值维护远程追踪分支和本地追踪分支的映射关系。
```bash
[branch "master"]
        remote = origin
        merge = refs/heads/master
```
合并可能会产生冲突，需要自行解决。

git pull --rebase命令会在拉取后进行变基，将本地追踪分支变基到远程追踪分支。

### 合并还是变基
通过合并，每次pull可能会产生一个额外的合并提交，记录更新同时存在于每个分支上。有些人认为这个提交真实反应开发历史记录，而有些人认为这个提交是多余的，打乱提交历史。


变基从根本上改变一系列提交是在何时何地开发的概念，开发历史记录可能会丢失。当然变基仍然需要解决冲突，由于变基的变更只在本地版本库，因此不会影响远程库的历史记录。

>特殊合并操作快速推进不会产生额外的合并提交

## 追踪分支
在克隆操作或把远程版本库添加到版本库时会引入远程追踪分支，使用远程追踪分支的一个简单的检出请求会导致创建一个新的本地追踪分支，并与该远程追踪分支关联（只有分知名与远程版本库中的一个远程分支名匹配时才会如此）。

```bash
#查看所有分支
$ git branch -a
* master
  remotes/origin/dev
  remotes/origin/master
#查看本地分支
$ git branch
* master

```
上图可看到当前版本库的远程分支origin/master有对应本地追踪分支master，而origin/dev则没有对应的追踪分支。
```bash
$ git checkout dev
Branch dev set up to track remote branch dev from origin.
Switched to a new branch 'dev'

```
这样就创建了本地追踪分支dev。


出于某种原因，需要给本地追踪分支起一个不同的名字，可以使用-b参数
```bash
$ git checkout -b mydev --track origin/dev

```
创建本地追踪分支后Git会在config文件中添加branch条目。
```bash
$ cat .git/config
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        symlinks = false
        ignorecase = true
        hideDotFiles = dotGitOnly
[remote "origin"]
        url = E:/GitRepository/local.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
[branch "dev"]
        remote = origin
        merge = refs/heads/dev

```
>$ git branch --track dev origin/dev

>创建本地追踪分支而不检出


## 添加和删除远程分支
要在远程版本库中添加分支和删除分支，需要在git push命令中指定不同形式的refspec。

>refspec语法：[+]源：目标

```bash
#创建分支foo
$ git checkout -b foo
Switched to a new branch 'foo'

#推送分支到远程库origin
$ git push origin foo
Total 0 (delta 0), reused 0 (delta 0)
To E:/GitRepository/local.git
 * [new branch]      foo -> foo


```
>git push origin foo:rfoo
推送foo分支到远程版本库并创建rfoo分支

删除远程分支有两种语法。
>git push origin :foo

>git push origin --delete foo
