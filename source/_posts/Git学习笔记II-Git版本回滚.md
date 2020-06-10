title: Git学习笔记II-Git版本回滚

date: 2019-9-5 14:51:12

categories: 2019年9月

tags: [Git]

---

Git版本回滚


<!-- more -->
在当前branch上多次commit代码并且push后，发现不符合要求，需要回滚到特定的版本。

## 回滚方法一
在IDEA中打开Version Control-》log，选择相应的commit记录右键-》Revert Commit，即可退回提交记录，再push。


## 回滚方法二

步骤如下[^1]：

1、查找commitId

首先用命令行打开git项目路径，输入git log命令查看commit记录，如下：

    $ git log
找到commitId是9a0d02d1578ea064479296ad7efa70c5cb1a3717记录，这是执行上面命令后打印出来的信息：

    commit 9a0d02d1578ea064479296ad7efa70c5cb1a3717
2、找到需要回滚的commit，输入git reset --hard {commitId}，将本地文件回滚：

    $ git reset --hard c503cffa099332911d4fce2fc1399cb4bc3ba9d6
    HEAD is now at c503cffa0 add a constellation test case
3、此时本地文件已经回滚到刚刚commit 9a0d02d1578ea064479296ad7efa70c5cb1a3717之后的状态，但是服务器仍然没有改变，需要继续远程回滚：

    $ git push -f
执行，最终提示一系列内容，远程回滚成功


## 遇到问题
但是有时候发现还是报错[^2]：


    You are not allowed to force push code to a protected branch on this project
信息提示我无法强制 push 代码到一个受保护的分支？？ 哪怕我已经是 master 了，还是强推不了

## 解决方法
去 gitlab 的该项目的配置项看了一下(在 Settings 的 Repository 设置项的 Protected Branches)， 原来这个项目中有对 master 分支做了 protected 保护， 不允许强制推送，这个好像是项目创建的时候就默认的设置：

所以如果要强制推送到 master 的话，这边要先取消掉 protected 分支。所以点击 Unprotect 按钮

这时候就没有保护分支了， 然后这时候强制推送就成功了。 

然后为了安全，我们再重新设置为保护分支

这样就恢复成原来的样子了。
# 后记
虽然在我的 pc 上已经将这个 master 分支回滚了。 但是后面发现在我的另一个同事的 IDE 上，master 分支 git pull 之后，竟然还是旧的代码，而不是之前回滚之后的那个版本？？
    
    F:\airdroid_code\go\src\auto_pack>git pull
    Already up to date.
而且已经没办法再 pull 任何东西了，看了一下 gitlab ，发现 master 分支的代码确实是回滚之后的代码了。所以我怀疑有可能是 master 分支的 commit 版本太新了， 比当前线上的 master 的 commit 版本还新，所以才会 pull 不到任何东西。
所以解决方法，就是先把这个分支回滚到某一个更旧的 commit ，然后再 git pull ，这样才有东西：
    
    F:\airdroid_code\go\src\auto_pack>git reset --hard 3dfc6e8d
    HEAD is now at 3dfc6e8 readme
    
    F:\airdroid_code\go\src\auto_pack>git pull
    Updating 3dfc6e8..ad9e3f2
    Fast-forward
    main.go   | 18 +++++++++++++++---
    model.go  |  9 ++++++++-
    upload.go |  5 ++++-
    3 files changed, 27 insertions(+), 5 deletions(-)
果然这样就可以了。
所以结论就是，本地的分支 commit 如果比线上回滚后的分支 commit 还要新的话，那么 git pull 是拉不到任何东西的，所以就要先回滚到比线上的分支还旧的 commit ，然后再 git pull。


[^1]:https://blog.csdn.net/weixin_38569499/article/details/83017699

[^2]: https://kebingzao.com/2019/04/18/git-protected/