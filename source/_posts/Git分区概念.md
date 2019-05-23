---
title: Git分区概念
date: 2018-9-6 22:04:12  
categories: 2018年9月
tags: [Git]

---
# Git分区概念

复习一下Git的分区概念

<!-- more -->
 
## 1.工作区、暂存区、版本库
Git 本地数据管理，大概可以分为三个区，工作区,暂存区和版本库。 
- 工作区（Working Directory） 
是直接编辑的地方，肉眼可见，直接操作。 
- 暂存区（Stage 或 Index） 
数据暂时存放的区域。 
- 版本库（commit History） 
存放已经提交的数据，push 的时候，就是把这个区的数据 push 到远程git仓库了。

下图各区之间的数据传递流程示意图。

![image](http://ww1.sinaimg.cn/large/0071ouepgy1fv069kity5j30ov0ec765.jpg)

git对比命令

	git diff                工作区 vs 暂存区
	git diff head           工作区 vs 版本库
	git diff –cached        暂存区 vs 版本库
	
1.刚开始，什么操作都没有，三个区的数据是一致的，执行** git diff **命令都为空 
2.编辑文件增加代码，现在工作区内容发生变化，暂存区和版本库内容不变。 
3.执行**git add **操作后，修改同步到暂存区，现在工作区和暂存区数据一致。 
4.执行**git commit** 操作后，修改已经同步到版本库，三区数据再次保持一致。

## 2.为什么要先执行 git add然后 git commit？
很多 Git 的初级教程，几乎都有说先执行 git add ，然后 git commit。

那么为什么要先add然后commit呢？

**git commit**执行时，会提交暂存区的内容； 
**git add** 命令会将我们做的修改添加到暂存区中。 
这就是为什么** git commit** 之前要先执行 **git add** 的原因,如果不先执行add，那么直接执行commit时不会把当前的修改内容提交到代码库中的。

## 3.分区的好处

通过 checkout/stash/reset 等命令，通过不同的参数搭配使用，可以在工作区，暂存区和版本库之间，轻松进行数据的来回切换。

	修改了多个文件，在不放弃任何修改的情况下，其中一个文件不想提交，如何操作？
	（没add操作 : git add 想提交的文件
	已经add: git reset –soft 将文件从暂存区回滚到工作区，重新add）
	
	修改到一半的文件，突然间不需要或者放弃修改了，怎么恢复未修改前文件？
	 (git checkout)
	
	代码写一半，被打断去做其他功能开发，未完成代码保存？
	(git stash 提交到缓存区)
	
	代码写一半，发现忘记切换分支了？
	(git stash & git checkout)
	
	代码需要回滚了？（git reset）

命令 作用
	
	git reset –soft commit_xxid   
	暂存区->工作区 将commit_xxid提交从暂存区回滚到工作区
	
	git reset –mixed    commit_xxid 版本库->暂存区
	将commit_xxid提交从版本库回滚到暂存区
	
	git reset –hard commit_xxid 
	版本库->暂存区->工作区  三个区都清除commit_xxid提交