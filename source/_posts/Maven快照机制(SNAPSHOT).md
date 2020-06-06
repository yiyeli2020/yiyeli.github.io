---
title: Maven快照机制（SNAPSHOT）

date: 2020-06-05 17:10:12

categories: 2020年5月

tags: [Java, Maven]

---

介绍Maven快照机制（SNAPSHOT）

<!-- more -->


# 📖场景

一个大型的软件应用通常包含多个模块，并且通常的场景是多个团队开发同一应用的不同模块。举个例子，设想一个团队开发应用的前端，项目为app-ui（app-ui.jar:1.0），而另一个团队开发应用的后台，使用的项目是data-service（data-service.jar:1.0）。

现在可能出现的情况是开发data-service的团队正在进行快节奏的bug修复或者项目改进，并且他们几乎每隔一天就要发布库到远程仓库。

现在如果data-service团队每隔一天上传一个新版本，那么将会出现下面的问题：

data-service团队每次发布更新的代码时都要告知app-ui团队。
app-ui团队需要经常地更新他们pom.xml文件到最新版本。
为了解决这种情况， 快照（SNAPSHOT）的概念派上了用场。

# 📖什么是快照（SNAPSHOT）？

快照（SNAPSHOT）是一种特殊的版本，指定了某个当前的开发进度的副本。不同于常规的版本，Maven每次构建都会在远程仓库中检查新的快照。

现在data-service团队会每次发布更新代码的快照到仓库中，比如说data-service:1.0-SNAPSHOT来替代旧的快照jar包。

注意：每次更新jar包时，版本号不变，且后缀必须带上-SNAPSHOT。

# 📖项目快照（Snapshot） VS 版本（Version）

版本（Version）的情况下，如果Maven以前下载过指定的版本文件，比如说data-service:1.0，Maven将不会再从仓库下载新的可用的1.0文件。若要下载更新的代码，data-service的版本需要升到1.1。

快照（Snapshot）的情况下，每次app-ui团队构建他们的项目时，Maven将自动获取最新的快照(data-service:1.0-SNAPSHOT)。

## 备注：

版本（Version）存放在Release发布仓库。快照（Snapshot）存放在Snapshot快照仓库。

## 注意：
版本（Version）的概念，只要不带有-SNAPSHOT的关键字时，都会认为这是一个在Release发布仓库的jar包。其中在Release发布仓库的jar包命名除了具体的版本号之后还可以带上比如：1.0-Release、1.0-rc1等等的字样。

# 📖原理详解

Maven中的仓库分为两种，Snapshot快照仓库和Release发布仓库。Snapshot快照仓库用于保存开发过程中的不稳定版本，Release正式仓库则是用来保存稳定的发行版本。定义一个组件/模块为快照版本，只需要在pom.xml文件中在该模块的版本号后加上-SNAPSHOT即可（注意这里必须是大写），如下所示：
    
    <groupId>com.jsoft.test</groupId>
    <artifactId>testcommon</artifactId>
    <version>0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
Maven会根据模块的版本号（pom.xml文件中的version）中是否带有-SNAPSHOT来判断是快照版本还是正式版本。

如果是快照版本，那么在mvn deploy时会自动发布到快照版本库中，而使用快照版本的模块，在不更改版本号的情况下，直接编译打包时，Maven会自动从镜像服务器上下载最新的快照版本。如果是正式发布版本，那么在mvn deploy时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，编译打包时如果本地已经存在该版本的模块则不会主动去镜像服务器上下载。

所以，我们在开发阶段，可以将公用库的版本设置为快照版本，而被依赖组件则引用快照版本进行开发，在公用库的快照版本更新后，我们也不需要修改pom.xml文件提示版本号来下载新的版本，直接Maven执行相关编译、打包命令即可重新下载最新的快照库了，从而也方便了我们进行开发。

虽然，快照的情况下，Maven在日常工作中会自动获取最新的快照，你也可以在任何Maven命令中使用-U参数强制Maven下载最新的快照构建。命令如下：
    
    mvn clean package -U


#  📖实际使用时遇到的问题

例如引用快照的项目app-ui中无法导入data-service中的DTO，报错：
    
    ERROR] -> [Help 1]
    [ERROR] 
    [ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
    [ERROR] Re-run Maven using the -X switch to enable full debug logging.
    [ERROR] 
    [ERROR] For more information about the errors and possible solutions, please read the following articles:
    [ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException

搜索了相关信息找到的可能的错误原因：

1. setting.xml 中的localRepository标签中的地址不正确

2. 计算机当前用户对localRepository中地址所在的文件夹权限不够（我碰到的就是这个问题）

解决方法如下：

1. 确认setting.xml文件中localRepository中的地址是否存在

2. 修改仓库文件夹的权限为完全控制即可，打开IDEA-》Preference-》Maven，勾选Local repository
   

# 参考资料：
【1】https://www.cnblogs.com/EasonJim/p/6852840.html

【2】https://ayayui.gitbooks.io/tutorialspoint-maven/content/book/maven_snapshots.html

【3】https://blog.csdn.net/w13342233769/article/details/103484005

【4】https://blog.csdn.net/Vicky128/article/details/81285610