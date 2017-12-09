---
title: git快速入门
date: 2016-03-10 21:16:27
category: 版本控制
---
### （一）创建新仓库
创建新文件夹，打开，然后执行以下命令:
```git
git init
```
### （二）检出仓库
执行以下命令，创建一个本地仓库的克隆版本:
```git
git clone /path/to/repository
```
如果是远程服务器上的仓库，你的命令会是这个样子:
```git
git clone username@host:/path/to/repository
```
### （三）工作流
你的本地仓库由git维护的三棵“树”组成。第一个是你的**工作目录**，它持有实际文件；第二个是**暂存区**，它像一个缓存区域，临时保存你的改动；最后是**HEAD**，它指向你最后一次提交的结果。
![gitguide](/uploads/gitguide.png)
### （四）添加和提交
你可以提成更改（把他们添加到暂存区），使用如下命令:
```git
git add <filename>
或者
git *
或者
git add .
```
这是git基本工作流程的第一步；使用如下命令以实际提交改动:
```git
git commit -am”代码提交信息”
或者
git commit -m”代码提交信息”
```
现在，你的改动已经提交到了HEAD，但是还没有到你的远程仓库。
### （五）推送改动
你的改动现在已经在本地仓库的HEAD中了。执行如下命令以将这些改动提交到远程仓库:
```git
git push origin master
可以把master换成你想要推送的任何分支。
```
如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加:
```git
git remote add origin <server>
这样你就能够将你的改动推送到所添加的服务器上去了。
```
### （六）分支
分支是用来将特性开发绝缘开来的。在你创建仓库的时候，master是默认的分支。一般在其他分支上进行开发，完成后再将他们合并到主分支上。
![gitguide](/uploads/git-branch.png)
创建一个叫”version1”的分支，并切换过去:
```git
git checkout -b version1
切换回主分支:
git checkout master
把新建的分支删掉：
git branch -d version1
除非你将分支推送到远程仓库，不然该分支就是本地分支，不为他人所见的。
执行下面这条命令推送本地分支
git push origin version1
```
### （七）更新与合并
要更新你的本地仓库至最新改动，执行
```git
git pull
```
要在你的工作目录中，获取并合并远程仓库的改动，执行:
```git
git merge <branch>
```
在这两种情况下，git都会去尝试去自动合并改动。但是，有的时候并不成功，并可能出现冲突。这时候就需要你修改这些文件来手动合并这些冲突。改完之后，你需要执行如下命令以将他们标记为合并成功。
```git
git add <filename>
```
在合并改动之前，你可以使用如下命令预览差异:
```git
git diff <source_branch> <target_branh>
```
### （八）替换本地改动
假如你操作失误（当然，希望这种情况永远不会发生)你可以使用如下命令替换掉本地改动:
```git
git checkout —-<filename>
```
假如你想丢弃你在本地的所有改动与提交，可以到服务器上获取最新的版本历时，并将你本地分支指向它:
```git
git fetch origin
git reset —hard origin/master
```

### （九）合并多个commit
有的时候我们可能由于各种原因提交了许多临时的commit，而这些commit拼接起来才是完整的任务。那么我们为了避免太多的commit而造成版本控制混乱，通常我们推荐将这些commit合并成同一个。
```git
git rebase -i hash
```
**参考指南和手册:**
[图解git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)
[git社区参考书](http://git-scm.com/book/en/v2)

