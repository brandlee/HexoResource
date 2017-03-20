title: Having Fun with Git
date: 2016-03-05 11:14:17
tags: git
---
Git使用过程中的各种坑，长期不定时更新，enjoy！

<!--more-->
## 吧啦吧啦
先看一幅图,也曾经常干过这种体力活......

 ![](https://cdn-images-1.medium.com/max/800/0*n2QYqEj3coS_yKNl.png)

大部分时间我们只会用到`add`,`commit`,`branch`,`push`/`pull`这些命令。但是如果当出现了一些错误的操作的时候，我们是会按照漫画中所说的，删除本地项目文件夹然后重新`git clone`，那么这篇文章要做的工作就是让你改掉这个陋习，master git, change the world!
## 图示Git流

![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)

几个名词术语：
> Workspace：工作区
> Index / Stage：暂存区
> Repository：仓库区（或本地仓库）
> Remote：远程仓库

## Git命令
### Add
``` bash
# 添加指定文件到暂存区(中间空格隔开)
$ git add [file1] [file2] ...
```
``` bash
# 添加指定目录到暂存区(包括子目录)
$ git add [dir]
```
``` bash
# 添加所有修改过和新添加的文件(change|add)
$ git add .
```
``` bash
# 添加所有文件(change|delete|add)
$ git add --all .
```
### Remove
``` bash
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...
```
``` bash
# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]
```
### Reset(更多详情用力戳[这里](http://stackoverflow.com/questions/348170/undo-git-add-before-commit/348234#348234))
如果在暂存区(staging area)加入了一些错误文件且未提交代码，一条命令即可撤销：
``` bash
# 从暂存区移除一个文件
$ git reset [file]
```
``` bash
# 从暂存区移除所有没有提交的修改
$ git reset
```
### Branch
``` bash
# 列出所有本地分支
$ git branch
```
``` bash
# 列出所有远程分支
$ git branch -r
```
``` bash
# 列出所有本地分支和远程分支
$ git branch -a
```
``` bash
# 新建一个分支
$ git branch [branch-name]
```
``` bash
# 切换到指定分支，并更新工作区
$ git checkout [branch]
```
``` bash
# 新建一个分支,并切换到该分支
$ git checkout -b [branch]
```
``` bash
# 新建一个本地分支,推送到远程
$ git push [remote-branch] [local-branch]:[remote-branch]
```
``` bash
# 新建一个分支,并与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]
```
``` bash
# 在现有分支与指定远程分支之间建立追踪关系
$ git branch --set-upstream [branch] [remote-branch]
```
``` bash
# 删除本地分支
$ git branch -d [branch-name]
```
``` bash
# 删除远程分支
$ git push origin --delete [branch-name]
```
``` bash
# 合并指定分支到当前分支
$ git merge [branch]
```
``` bash
# 选择一个commit合并进当前分支
$ git cherry-pick [commit]
```
## ADVANCED
### Stash
``` bash
# 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致
# 同时，将当前的工作区内容保存到Git栈中
$ git stash
```
``` bash
# 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容
# 由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复
$ git stash pop
```
``` bash
# 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复
$ git stash list
```
``` bash
# 清空Git栈
$ git stash clear
```
## FYI
### [git-pretty](http://justinhileman.info/article/git-pretty/)

![](http://justinhileman.info/article/git-pretty/git-pretty.png)

## Q&A
- [**.gitignore not working**](http://stackoverflow.com/questions/11451535/gitignore-not-working)（更深入理解请戳[这里](https://segmentfault.com/q/1010000000430426)）

 **进行操作之前，记得先将代码commit到本地仓库**
```bash
# on Windows: git rm . -r --cached
git rm -r --cached .
git add .
git commit -m "fixed untracked files"
```
- [**Difference between Merge and Rebase**](http://stackoverflow.com/questions/11451535/gitignore-not-working)
  时间有限，个人还不是很理解，贴上链接，理解之后会贴上实际操作

## 未完待续(长期更新)