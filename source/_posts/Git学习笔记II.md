title: Git学习笔记II
date: 2019-9-5 14:51:12
categories: 2019年9月
tags: [Git]

---

Git版本回滚


<!-- more -->
在当前branch上多次commit代码并且push后，发现不符合要求，需要回滚到特定的版本。步骤如下：

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
# 参考资料

【1】 https://blog.csdn.net/weixin_38569499/article/details/83017699
