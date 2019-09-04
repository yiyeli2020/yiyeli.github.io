title: Git学习笔记I
date: 2019-9-4 17:08:12
categories: 2019年9月
tags: [Git]

---

Git切换新分支


<!-- more -->
# 查看本地已有的分支

    git branch

# 本地建立新分支并推送到远程仓库

## 创建本地分支

  git checkout -b 新分支名字

执行该指令后，会在本地创建一个新分支，该分支是从当前分支上建出的，所以所有文件内容都和当前分支一模一样，这是正常的。创建成功后，将自动切换至新分支上。

## 推送本地分支到远程仓库

  git push --sest-upstream origin 分支名字


# 将远程git仓库里的指定分支拉取到本地（本地不存在的分支）
首先更新远程分支列表

    git remote update origin --prune
    或者

    git remote update origin -p

然后将指定分支拉取到本地

    git checkout -b 本地分支名 origin/远程分支名

这个将会自动创建一个新的本地分支，并与指定的远程分支关联起来。

如果出现提示：

    fatal: Cannot update paths and switch to branch 'dev2' at the same time.
    Did you intend to checkout 'origin/dev2' which can not be resolved as commit?

表示拉取不成功。我们需要先执行

    git fetch

然后再执行

    git checkout -b 本地分支名 origin/远程分支名
即可。

# git fetch与git pull的区别

FETCH_HEAD： 是一个版本链接，记录在本地的一个文件中，指向着目前已经从远程仓库取下来的分支的末端版本。

commit-id：在每次本地工作完成后，都会做一个git commit 操作来保存当前工作到本地的repo， 此时会产生一个commit-id，这是一个能唯一标识一个版本的序列号。 在使用git push后，这个序列号还会同步到远程仓库。


有了以上的概念再来说说git fetch

git fetch：这将更新git remote 中所有的远程仓库所包含分支的最新commit-id, 将其记录到.git/FETCH_HEAD文件中

git fetch更新远程仓库的方式如下：

    git fetch origin master:tmp
    //在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支
    git diff tmp
    //来比较本地代码与刚刚从远程下载下来的代码的区别
    git merge tmp
    //合并temp分支到本地的master分支
    git branch -d temp
    //如果不想保留temp分支 可以用这步删除

## 如果直接使用git fetch，则步骤如下：

创建并更新本地远程分支。即创建并更新origin/xxx 分支，拉取代码到origin/xxx分支上。

在FETCH_HEAD中设定当前分支-origin/当前分支对应，如直接到时候git merge就可以将origin/abc合并到abc分支上。

## git fetch origin

只是手动指定了要fetch的remote。在不指定分支时通常默认为master

## git fetch origin dev

指定远程remote和FETCH_HEAD，并且只拉取该分支的提交。

## git pull :

首先，基于本地的FETCH_HEAD记录，比对本地的FETCH_HEAD记录与远程仓库的版本号，然后git fetch 获得当前指向的远程分支的后续版本的数据，然后再利用git merge将其与本地的当前分支合并。所以可以认为git pull是git fetch和git merge两个步骤的结合。

git pull的用法如下：

    git pull <远程主机名> <远程分支名>:<本地分支名>
    //取回远程主机某个分支的更新，再与本地的指定分支合并。

因此，与git pull相比git fetch相当于是从远程获取最新版本到本地，但不会自动merge。如果需要有选择的合并git fetch是更好的选择。效果相同时git pull将更为快捷。

# git 命令集锦
## git 设定
    git config --global user.name

    git config --global user.email

    git config --global color.ui true

    git config --global alias. <命令名称>

    比如：git config —global alias.st status

## git常用
    git init
    其反操作：rm -rf .git

    git clone

    git status

    git status -s：仅显示已修改的文档名称

    git status -s -b：显示分支名称

    git diff

    git add .

    git add -A

    git commit -m "message"

    git commit --amend "message" 修改上一次 commit 內容

    git push

    git pull

    git log

    git log --graph 查看分支合并图

    git log --pretty=oneline

    git reflog 查看命令历史

    rm 删除本地file

    git rm 删除版本库file

## git分支操作
    git branch

    git branch -r 显示远端分支

    git branch -a 显示所有分支

    git branch <分支名称> 建立分支

    git branch -m <旧分支名称> <新分支名称> 修改分支名字

    git branch -d <分支名称>

    git checkout

    git checkout -b <分支名称>：表示 建立並切換 至该分支

    git merge <分支名称>

    git reset --merge 放弃merge

## 远端操作
    git remote 显示远端数据库列表

    git remote add <名称> 添加远端数据库

    git checkout <本地分支名称> origin/<远端分支名称>

## 取远端分支建立本地端分支

    git push <分支名称>

## 在远端建立分支 / 上传(或更新)內容至远端分支

    git fetch <分支名称>

## 查看远端数据库分支的修改內容

    git pull <分支名称> 合并(或更新)远端至本地端分支

    注：pull = fetch + merge

    git push :<分支名称> 刪除远端分支

    git remote set-url <名称> <新连接位址>

## 修改远端数据库地址

    git remote rename <旧名称> <新名称>

## 修改远端数据库名称

## 暂存
    git stash 暂存现在的修改狀況

    git stash list 列出暂存清单

    git stash apply 取出上一次暂存

    git stash pop 取出上一次暂存(该暂存会被移除)

    git stash@{id} 指定特定暂存

    git stash clear 清空所有暂存

## 操作提交记录
    git commit --amend 修改上一次的commit

    git reset HEAD 放弃该修改记录 (reset)

    git reset --soft HEAD^ 取消上一次 commit，並 保留 修改纪录

    git reset --hard HEAD^ 取消上一次 commit，並 刪除 修改纪录

# 参考资料：

【1】 https://blog.csdn.net/riddle1981/article/details/74938111
