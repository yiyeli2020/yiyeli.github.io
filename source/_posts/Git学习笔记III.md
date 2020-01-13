title: Git学习笔记III
date: 2020-1-13 18:12:12
categories: 2020年1月
tags: [Git]

---

使用Git进行工程开发时需要的常见规范，常见的Git命令， Git 客户端SourceTree简介


<!-- more -->
# 背景
在团队开发中，遵循一个合理、清晰的Git使用流程，是非常重要的。否则，每个人都提交一堆杂乱无章的commit，项目很快就会变得难以协调和维护。

# Git的特点
Git有很多优点，其中很显著的一点，就是版本的分支（branch）和合并（merge）十分方便，但如果管理不善，分支太多会导致杂乱不堪。

Vincent Driessen提出了一个分支管理的策略，我觉得非常值得借鉴。它可以使得版本库的演进保持简洁，主干清晰，各个分支各司其职、井井有条。

# Git分支管理策略
主分支：应该有且只有一个主分支，所有提供给用户使用的正式上线版本都是从该主分支上发布。

开发分支：日常开发任务在develop分支上进行。
如果想要对外发布，就需要提交merge request，申请从develop分支merge到master中进行合并。

临时性分支：除了发布和开发，我们还需要一些临时性分支，用于 功能分支（feature）、预发布功能（release）分支、修正bug（fixbug）分支等，以及敏捷开发中的Sprint分支，在使用完成后，都需要将其快速删除，使得代码库的分支始终只会有 develop 和 master 分支。

# 当前功能分支命名规范
功能分支（feature，命名规范 feature-XXX）
Sprint分支（命名规范 sprint-XXX），在Sprint结束之后，合并至develop并进行预发布；
预发布功能分支（release）：在正式发布之前，我们需要一个预发布版本用来做测试，可以考虑将测试Lain环境中的分支都指向release分支而非develop；
修正bug（fixbug）分支：修补bug分支是从Master分支上面分出来的。修补结束以后，再合并进Master和Develop分支。它的命名，可以采用fixbug-XXX的形式。
根据JIRA问题创建出来的分支（jira-XXX）：当存在JIRA问题，并以此作为修改代码的依据时，可以使用这种方式的分支命名。
# Git Commit信息的编写
如果有对应JIRA问题，编写格式可以直接以JIRA问题为准（这样可以在IDEA中查看代码时能够快速链接到该问题的描述）

    Git JIRA 提交信息
    JIRA-56
    http://jira/XXX

如果没有JIRA问题，首先尝试在JIRA系统上建立问题并按照第一条原则。如果确实没有必要，则需要在提交时指定问题类型，基本的模板为：


    Git 普通问题提交信息
    <type>(<scope>): <subject>
    type

    feat：新功能（feature）
    fix：修补bug
    doc：文档（documentation）
    style： 代码风格或格式（不影响代码运行的变动）
    refactor：重构（即不是新增功能，也不是修改bug的代码变动）
    test：增加测试
    build: 构建过程或辅助工具的变动
    chore：其它类型提交,无关src目录以及test目
    scope
    用于说明此次提交影响的范围
    subject  commit 目的的简短描述。
    以动词开头，使用第一人称现在时，比如change，而不是changed或changes
    第一个字母小写
    结尾不加句号（。）

# Git 标准流程
Git分支管理流程遵循以下几点：

developer没有任何权限能直接将codepush到develop、release和master分支。
developer开发feature时先建立issue之后再从develop创建相应的feature分支。
feature开发完毕后提交测试，测试可对此feature分支需要按照之前设计的case进行测试，并将bug录入到JIRA中。
上线日前1天从develop分支创建release分支命名规则遵守：release_yyyy/MM/dd
与产品沟通最后确认上线内容，并于涉及到的项目组沟通确认上线日配合验证测试。
将所有需要上线的feature分支提交merge request到release分支此时由中级或高级developer进行code review。
代码合并后测试进行主干的测试。确保上线前一天封版（理论上上线当天不进行代码改动）。
上线当上午发送上线通知邮件。

# 代码提交规范：
首先master 拉取分支----->建立开发分支------>开发完成-→合并develop 分支---->提交测试----->测试完成---->开发分支合并到master分支

## 流程阻塞型功能提测
如果为后端必须零流量进行线上的回归，再开权限。

# 提交测试：
项目开发完毕后，发邮件提测，提测内容如下：

背景：提测项目的背景和目的
修改点：代码变动和参数参数变动，以及配置文件有哪些变动
单元测试的覆盖率报表地址
有代码变动，可将git的diff结果页面的链接附上（是否建立代码的卡关流程，测试看完后，高工review确认合主分支）
测试或验证功能点：本次测试的重点check点，标明和强调（例如功能、兼容性、参数、代码逻辑，防止漏侧改动点）
## 后端提测:
(1).需要指明哪些接口，接口的请求(新接口、老接口、内部接口)的必要参数和接口响应结果的重要业务字段，另外这些字段的重要逻辑需要指明
(2).接口的功能是什么
(3).测试的功能点是什么，需要测试做什么
前端和安卓测试：
(1).交互逻辑有哪些修改,交互的条件有哪些
(2).新增UI页面、前端效果有哪些
(3).交互完成的功能主要是什么
(4).主要测试的功能点是什么
测试范围：哪个端（pc\移动\h5）
自测范围和自测机型：自测功能点要详细说明（自测中验证了什么）

附上各个应用中git基本操作流程
个人可以根据需要，选择适合自己的工具（建议从原理入手，选择命令行）

# 命令行

查看当前分支信息

    git branch -a  （-a 表示列出所有的本地分支以及远程分支）
    git branch -d xxx (删除合并过的分支xxx)
    git branch -D xxx (强制删除分支xxx，及时xxx未被合并过)
切换分支

    git checkout xxx (切换至存在的分支)
    git checkout -b yyy (以当前分支为基础，新建分支yyy，并切换至分支yyy)
更改工作区内容至暂存区

    git add . (添加所有文件至暂存区)
    git add  source/to/file (添加指定文件至暂存区)
    git rm source/to/file (删除工作区和暂存区的指定文件)
    git mv source/to/file destination/to/file (移动工作区和暂存区的指定文件)
提交暂存区内容至本地分支

    git commit -m "<type>(<scope>): <subject>"
同步本地分支至远程分支

    git push origin branch
拉取远程代码至本地

    git fetch
    git rebase master

# SourceTree
    SourceTree 是 Windows 和Mac OS X 下的 Git 客户端，下载地址：https://www.sourcetreeapp.com/
## 提交代码 （git commit & push）
1.提交代码前应仔细看代码变更，不提交的改动可以选择暂存或舍弃。

2.选择要提交的文件后，会显示在已暂存文件列表中。

3.填写提交信息。

4.提交（勾选立即推送可立即推送到远端）

## 本地合并分支（git merge）

提交代码后，如果有冲突无法在远端解决，可以在本地合并，解决冲突。

假设将 feature-test 合并到 master 上，先切换到 master 分支，再右键选择合并 feature-test 到 master。

## 贮藏 （git stash）

贮藏是一项十分有用的功能。当你处在开发阶段，有代码没提交，却需要切换分支或临时解决其他问题时，贮藏是最好的解决办法。

贮藏可以将当前改动暂时存起来，在左侧已贮藏列表显示已储存的代码。

在解决完临时问题后，通过应用贮藏将代码变更应用到当前分支。


SourceTree 的基本操作介绍如上，更多详细操作可以参考以下文档：

官方文档：https://confluence.atlassian.com/get-started-with-sourcetree

SoureTree使用方法：https://www.jianshu.com/p/6d2717c2a3e1



提交merge request之后，如果source branch没有其他用处，可在merge完直接删除掉


# 参考资料
