---
title: Git-使用快速入门(三)
date: 2017-09-17 21:27:47
tags: [Git]
categories: [Git]
comments: true
---
# 创建初始版本库
首先从一个空的文件下创建版本库。
```bash
$ cd public
$ echo 'init git repository' > text
$ git init
```
git init会创建一个名为.git的隐藏目录。

# 将文件添加到版本库
git init创建新的Git版本库，最初Git版本库是空的。需要使用git add命令添加文件到版本库中。
>git add . 把当前目录及子目录中文件添加到版本库中

在执行add前，运行git status命令查看版本库状态
```bash
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        text

nothing added to commit but untracked files present (use "git add" to track)

```
此时text文件还是未追踪的状态，执行git add命令之后，Git会在索引中暂存（staged）这个文件。
<!--more-->
```bash
$ git add text

#再次执行git status
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   text

```
接下来执行git commit可提交text到版本库。
```bash
$ git commit -m "add text"
[master (root-commit) 5147532] add text

```
>git commit时Git需要知道你的名字和email

>可以使用git config命令在配置文件中保存

>git config user.name "name"

>git config user.email "email"

修改text文件后，因为text文件已经存在版本库中。因此可以直接执行commit命令提交文件。
```bash
$ git commit text -m "修改"
The file will have its original line endings in your working directory.
 1 file changed, 1 insertion(+), 1 deletion(-)
```
# 查看提交
git log可以查看提交记录。
```bash
$ git log
commit 3c4fc9aea919825b51a43a10e4cabe67aac9b0b7
Author: liangshenlong <youremail@host,com>
Date:   Sun Sep 17 22:32:41 2017 +0800

    修改

commit 5147532990a37fdde5268063358ff74c37156fc7
Author: liangshenlong <youremail@host,com>
Date:   Sun Sep 17 22:27:12 2017 +0800

    add text


```
查看指定提交的详细信息，可以使用git show命令
```bash
$ git show 3c4fc9aea91
commit 3c4fc9aea919825b51a43a10e4cabe67aac9b0b7
Author: liangshenlong <youremail@host,com>
Date:   Sun Sep 17 22:32:41 2017 +0800

    修改

diff --git a/text b/text
index be23fc5..a82746d 100644
--- a/text
+++ b/text
@@ -1 +1 @@
-init git repository
+ttt

```
另一种方式是使用show-branch，提供当前分支间接的单行摘要。
```bash
$ git show-branch --more=5
[master] 修改
[master^] add text

```
参数"--more=5"表示5个提交，因为只有2个提交，因此只显示两行。
# 查看提交差异
git diff可以查看两个提交之间的差异，因为两次提交都只是修改text，因此比较这两个提交就可以查看text的差异。
```bash
$ git diff 51475 3c4fc
diff --git a/text b/text
index be23fc5..a82746d 100644
--- a/text
+++ b/text
@@ -1 +1 @@
-init git repository
+ttt

```
# 版本库内文件的删除和重命名
在版本库中删除文件与添加文件类似。git rm命令可以文件。

例如删除版本库中的text1文件
```bash
$ git rm text1
rm 'text1'

#执行git status查看状态
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        deleted:    text1

#执行commit提交变更
$ git commit -m "删除text1"
[master f4db54e] 删除text1
1 file changed, 1 deletion(-)
delete mode 100644 text1

```
>git rm --cached 文件名 可删除索引中文件

git mv命令可以重命名文件，操作与rm类似。
# 创建版本库副本
git clone命令可以创建版本库副本
```bash
#本地克隆
$ git clone public public_clone
#克隆远程库
$ git clone url localdir/localname
```

# 配置文件
Git支持不同层次的配置文件。按照优先级递减的顺序，如下：
>.git/config

>版本库特定的配置设置，可用--file选项修改，是默认选项。这些设置是最高优先级的。

>~/.gitconfig

>用户特定的配置设置，可用--global选项修改。

>/etc/gitconfig

>系统范围的设置配置。根据实际安装情况，这个系统设置文件可能在替他位置。

```bash
#建立username用于所有版本库
$ git config --global user.name "name"
#建立username用于特定版本库
$ git config user.name "name"

#查看配置
$ git config -l
core.symlinks=false
core.autocrlf=true
color.diff=auto
color.status=auto
color.branch=auto
color.interactive=true
help.format=html
http.sslcainfo=D:/Git/mingw64/ssl/certs/ca-bundle.crt
diff.astextplain.textconv=astextplain
rebase.autosquash=true
credential.helper=manager
user.name=liangshenlong
user.email=314267619@qq,com
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
core.hidedotfiles=dotGitOnly

```
也可以直接打开config文件查看，修改文件
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
[user]
        name = liangshenlong
        email = email@host.com

```
直接修改config文件也可以添加，修改配置信息。
