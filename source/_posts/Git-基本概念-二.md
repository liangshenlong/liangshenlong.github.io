---
title: Git-基本概念(二)
date: 2017-09-17 10:19:22
tags: [Git]
categories: [Git]
comments: true
---
# 基本概念
## 版本库
Git版本库（repository）只是一个简单的数据库，其中包含所有用来维护与管理项目的修订版本和历史信息。跟大多数VCS一样，一个版本库维护项目的整个生命周期的完整副本。不同的是Git版本库不仅提供版本库所有文件的完整副本，还提供版本库本身的副本。

在版本库中，Git维护两个主要的数据结构：对象库（Object store）和索引（index）。所有这些版本库数据存在在工作目录根目录下.git隐藏文件夹中。
<!--more-->
## Git对象类型
对象库是Git版本库实现的基础。它包含原始数据文件和所有日志信息、作者信息、日期，以及其他用来重建项目任意版本或分支的信息。

Git放在对象库中的对象只有4种类型：块（blob）、目录树（tree）、提交（commit）和标签（tag）
>块（blob）

>文件的每一个版本表示为一个块（blob）。blob是“二进制大对象”（binary large object）的缩写，用来指代某些可以包含任意数据的变量或文件，同时其内部结构会被程序忽略。一个blob保存一个文件的数据，但不包含任何关于这个文件的元数据，甚至连文件名也没有

>目录树（tree）

>一个目录树对象代表一层目录信息。它标记blob标识符、路径名和在一个目录里所有文件的一些元数据。它也可以递归引用其他目录树或子树对象，从而建立一个包含文件和子目录的完整层次结构

>提交（commit）

>一个提交对象保存版本库中每一次变化的元数据，包括作者、提交者、提交日期和日志信息。每一个提交对象指向一个目录树对象，这个目录树对象在一张完整的快照中捕获提交时版本库的状态。大多数提交都有父提交，一些提交可能有多个父提交。最初的提交或者跟提交（root commit）是没有父提交的。

>标签（tag）

>一个标签对象分配一个任意的且人类可读的名字给一个特定对象，通常是一个提交对象。

随着时间的推移，所有信息在对象库中会变化和增长，项目的编辑、添加和删除都会被跟踪和建模。为了有效的利用磁盘空间和网络宽带，Git把对象压缩现在打包文件（pack file）中，这些文件也在对象库中。
## 索引
索引是一个临时的、动态的二进制文件，它描述整个版本库的目录结构。更具体的说，索引捕获项目在某个时刻的整体结构的一个版本（最后一次提交和下次提交之间的时刻）。
开发人员执行Git命令在索引中暂存（stage）变更。变更通常是添加、修改或删除某些文件。索引为记录和保存这些变更，直到提交这些变更。

当然也可以删除或替换索引中的变更
## 可寻址内容名称
Git对象库被组织及实现成一个内容寻址的存储系统。对象库中的每个对象都有一个唯一名称，这个名称是向对象的内容应用SHA1得到的SHA1散列值。文件的任何变化都会导致SHA1值变化。

SHA1的值是一个160位的数，通常表示为一个40位的十六进制数。
>5159c4c8529657ecea8b26f265dbec9156fb830

## Git追踪内容
Git的内容追踪表现为两种关键的方式

1.Git对象库基于对象内容计算SHA1值，因此两个文件即使路径不同在Git库中也只会保存一份blob对象。

2.当文件从一个版本到另一个版本的时候，Git内存数据库会有效的存储每个文件的每个版本，而不是它们的差异。

# 对象库图示
![git对象图](/images/git对象图.png)

# 文件管理
Git将所有文件分成3类：已追踪、被忽略的以及未追踪的。

>已追踪的（Tracked）

>已追踪的文件是指已经在版本库中的文件，或者已暂存在索引中的文件。如果想将新文件添加为已追踪文件，执行 git add 文件名

>被忽略的（Ignored）

>被忽略的文件必须在版本库中被声明为不可见或被忽略，即使出现在工作目录也不会被追踪。Git通过.gitignore文件管理被忽略文件列表，子目录的.gitignore优先级大于父目录的优先级

>未追踪的（Untracked）

>未追踪的文件是指那些不在前两类中的文件。

# 提交
## 绝对提交名
提交的绝对提交名是它的SHA1值，由于输入一个40位长的SHA1数字过于麻烦，因此Git允许使用唯一的前缀的缩短数字
``` bash
$ git log 80d45377b296ee9aace788735a58bf56df17eb8c
$ git log 80d45
```
## 相对提交名
除了第一个跟提交之外，每个提交都有比他更早的提交。如果一个提交有多个父提交，那么它必定是合并产生的提交。

在同一代提交中，^是用来选择不同父提交的。给定一个提交C，C^1是第一个父提交，C^2是第二个父提交，等等

~用于返回父提交之前并选择上一代提交。给定一个提交C,C~1是它的父提交，C~2是它父提交的父提交。

## 提交历史记录
git log输出提交记录

![gitlog](/images/gitlog.png)

随着时间推移，git log历史越来越多，可以通过since..until来限制展示提交的范围。
``` bash
$ git log master~11..master~8
```
这里，git log显示了master~11到maste~8之间的所有提交，此种方式使用了集合减法运算，提交master~8可达的提交集合减去master~11可达的提交集合。

# 引用和符号引用
引用（ref）是一个SHA1散列值，指向Git对象库中的对象。这个对象通常是提交对象。符号引用（symref）间接指向Git对象。

本地特性分支名称、远程跟踪分支名称和标签名都是引用。

每一个符号引用都有一个以ref/开始的明确全程，并且都分层存储在版本库的.git/refs/目录中。目录中基本上有三种不同的命名空间代表不同的引用：refs/heads/ref代表本地分支，refs/remotes/ref代表远程分支，refs/tags/ref代表标签。

``` bash
$ git branch -a
* dev
  master
  remotes/origin/dev
  remotes/origin/master
```
Git自动维护几个用于特殊目的的特殊符号引用。这些引用在任何使用提交的地方使用。
>HAED

>HEAD始终指向当前分支的最近提交。

>ORIG_HEAD

>某些操作，例如合并（merge）和复位（reset），会把调整为新值之前的先行版本的HEAD记录到ORIG_HEAD中。可以使用ORIG_HEAD来恢复或回滚到之前的状态或者做一个比较。

>FETCH_HEAD

>当使用远程库时，git fetch命令将所有抓取分支的头记录到.git/FETCH_HEAD中。FETCH_HEAD是最近抓取的分支HEAD的简写，并且仅在刚刚抓取操作之后才有效。使用这个符号引用，哪怕是一个对没有指定分支名的匿名抓取操作，都可以在git fetch时找到提交的HEAD。

>MERGE_HEAD

>当一个合并操作正在进行时，其他分支的头暂时记录在MERGE_HEAD中。换言之，MERGE_HEAD是正在合并进HEAD的提交。

所有符号引用都可以使用底层命令git symbolic-ref进行管理。

# 分支
分支是从一种统一的、原始的状态分离出来的，使得开发能在多个方向上同时进行，并可能产生项目的不同版本。Git的分支系统是轻量级的、简单的。
## 创建分支
``` bash
#创建分支newbranch
$ git branch newbranch
#创建分支newbranch，以oldbranch分支的HEAD作为开始提交
$ git branch newbranch oldbranch
#创建分支newbranch，以指定提交作为开始提交
$ git branch newbranch 80d45377b296ee9aace788735a58bf56df17eb8c
```
## 查看分支

git branch查看本地分支

git branch -r 查看远程分支

git branch -a 查看所有分支

git show-branch命令提供更详细的输出
``` bash
$ git show-branch
* [dev] 解决右侧菜单图片不显示问题
 ! [master] udpate blog dev
--
*  [dev] 解决右侧菜单图片不显示问题
*  [dev^] 增加文章Git
*  [dev~2] 更新博客
*+ [master] udpate blog dev

```
## 检出分支
使用git checkout命令

合并检出 git checkout -m dev

检出创建分支 git checkout -b dev

* git checkout -- filename是从索引中检出文件

## 删除分支
命令git branch -d branchname可删除分支

Git不会让你删除一个包含不存在与当前分支中的提交的分支。
可以使用-D覆盖Git的安全检查

# diff
git diff命令可比较两个不同文件。
## git diff命令的格式
一个根级别的diff可以比较两个版本库的不同。
可供树或类树对象使用diff的基本来源：
>整个提交图中的任意树对象

>工作目录

>索引

git diff命令进行树比较时可以通过提交名、分支名或者标签名。
>git diff

>这个命令显示工作目录和索引之间的差异。

>git diff commit

>这个命令会显示工作目录和给定提交之间的差异。

>git diff --cached commit

>这个命令会显示索引中和给定提交的差异。如果省略commit，则默认HEAD。

>git diff commit1 commit2

>这个命令会显示给定的两个提交之间的差异。

## git diff和提交范围
git diff命令支持两点语法来显示两个提交之间的不同。下面两个命令是等价的：
``` bash
$ git diff master dev
$ git diff master..dev
```
>这里的两点语法和git log的不同

## 比较svn和git如何产生diff
SVN会跟踪一系列修订版本，并只存储文件间的差异。所以在比较不同时，svn会将所有的差异合并到工作副本中来产生比较文件。

Git每次提交都是保存整个文件，因此比较的时候直接操作两个版本的完整文件。

# 更改提交
修改提交会导致提交记录改变，如果提交未推送到远程版本库则没有关系，但是如果提交已经推送到远程版本库，那么就不应该再修改提交。

## 使用git reset
git reset命令会把版本库和工作目录改变成已知状态。git reset调整HEAD引用指向给定的提交，默认情况下还会更新索引以匹配该提交。根据需要，git reset命令也可以修改工作目录以呈现给定提交的项目修订版本。

git reset可以覆盖并销毁工作目录中的修改，可能会导致数据丢失。

git reset命令有三个重要选项：
>git reset --soft commit

>soft会将HEAD引用指向给定提交。索引和工作目录的内容保持不变。

>git reset --mixed commit

>mixed会将HEAD指向给定提交。索引内容也跟着改变以符合给定提交的树结构，但是工作目录中内容保持不变。

>git reset --hard commit

>hard会将HEAD引用指向给定提交。索引和工作目录内容都会修改成给定提交的样子。

| 选项 | HEAD | 索引 | 工作目录 |
| :--- | :--- | :--- | :--- |
| --soft | 是 | 否 | 否 |
| --mixed（默认） | 是 | 是 | 否 |
| --hard | 是 | 是 | 是 |
## 使用git cherry-pick
git cherry-pick提交命令会在当前分支上应用给定提交引入的变更。这将引入一个新的独特提交。严格来说，使用git cherry-pick并不改变版本库中的现有历史记录，而是添加历史记录。

git cherry-pick命令通常用于把版本库中一个分支的特定提交引入一个不同的分支中，最常见用法是把维护分支的提交移植到开发分支。
``` bash
#将dev分支上的倒数第三次提交引入当前分支
$ git cherry-pick dev~2
```
在产生新提交的过程中可能需要解决冲突。

git cherry-pick也允许一次选择并应用一个范围的提交。
``` bash
$ git cherry-pick dev~6..dev~2
```
## 使用git revert
git revert提交命令跟git cherry-pick命令大致相同，主要区别是：它应用给定提交的逆过程。此命令用于引入一个新提交来抵消给定提交的影响。常见用途是撤销提交历史记录中的某个提交的影响。
``` bash
#产生一个新的提交，“撤销”dev~2提交产生的影响
$ git revert dev~2
```
## 变基提交
git rebase命令是用来改变一串提交以什么为基础的。常见用途是保持你正在开发的一系列提交相对于另一个分支是最新的，那通常是master分支或者是来自另一个版本库的追踪分支。

![执行rebase前](/images/rebase_before.png)
``` bash
$ git checkout dev
$ git rebase master
```

![执行rebase后](/images/rebase_after.png)

变基操作还有很多其他复杂的应用。

# 储藏
日常的开发工作中，经常因为修复bug、紧急需求、联调等各种突发事件导致开发中断。储藏（stash）可以很好的应对这种问题。

``` bash
#开发中...

#接收到高优先级工作
$ git stash save
# git stash save "message"

#处理完高优先级工作

$ git stash pop
#工作目录恢复save前状态
```
stash使用栈保存数据，save时数据入栈，pop时出栈（该操作会去除储藏的内容并合并到当前状态）。

git stash apply可检出储藏栈中内容，又不删除栈中数据。

git stash drop可以删除储藏栈中内容。

pop命令相当于apply + drop。

git stash list按照保存时间由近及远的顺序列举储藏栈中内容。
