---
title: Hexo+github搭建个人博客问题总结

date: 2022-02-17 14:12:12

categories: 2022年2月

tags: [Hexo, Github, Git]


---
 

总结Hexo+github搭建个人博客过程中两个费时较长的问题及其解决方法


<!-- more -->

[TOC]

# Mac 上配置多个git账号

https://www.jianshu.com/p/698f82e72415

此处将公司gitlab地址假设为company.cn

    ssh-keygen -t rsa -f ~/.ssh/id_rsa_gitlab -C "yiyeli@company.cn"
    ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "15652771941@163.com"
    ssh-keygen -t rsa -f ~/.ssh/id_rsa_gitee -C "15652771941@163.com"
    
    ssh -T git@company.cn
    
    ssh -T git@gitee.com
    
    
    ssh -T git@github.com
    
    
    ssh: connect to host github.com port 22: Operation timed out

连接其它git时都成功了，只有github报错，可以换个端口,在github下加一栏 Port 443，但还是没作用
    
    #公司
    Host gitlab.company.corp
    Hostname gitlab.company.corp
    IdentityFile ~/.ssh/id_rsa_gitlab
    User gitlabLoginName
      
    #个人github
    Host github.com
    Hostname github.com
    IdentityFile ~/.ssh/id_rsa_github
    User githubLoginName
    # Port 443
    
    #个人gitee
    Host gitee.com
    Hostname gitee.com
    IdentityFile ~/.ssh/id_rsa_gitee
    User giteeLoginName

## 使用公司gitlab时出错
    
    Total 10 (delta 4), reused 0 (delta 0), pack-reused 0
    remote: git rev-list xxxx--after='2021-11-29 15:55:00'        
    remote: GL-HOOK-ERR: 6xxxx的提交者xxxx@163.com不是公司邮箱，请修改您的邮箱为公司邮箱,若仍无法解决，请提交jira工单:
    remote: error: hook declined to update refs/heads/feature-chinese2022        
    error: failed to push some refs to 'gitlab.company.corp:ce-datamonitorsystem/grafana.git'

从错误原因可以看到是提交的邮箱错误，原因是第一次commit时IDE会弹出一个Git User Name Is Not Defined的窗口，填完用户名和邮箱后会默认将Set properties globally的选项打上勾，这样在配置多个git账户时就会出错，因为邮箱等参数不能是全局一致的，记得每次把这个Set properties globally选项去掉就好了。


# 问题：本地同时配置多个github账号导致博客部署失败

因为本地同时配置有两个github账号，在Hexo+github搭建个人博客部署到账号A成功后用相同方式部署到账号B时执行hexo d时出现问题

	remote: Permission to B/B.github.io.git denied to A.
	fatal: unable to access 'https://github.com/B/B.github.io.git/': The requested URL returned error: 403
	FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
	Error: remote: Permission to B/B.github.io.git denied to A.
	fatal: unable to access 'https://github.com/B/B.github.io.git/': The requested URL returned error: 403

原因是在缺省设置下，github page只有该page对应的账号A才能push，为了解决该问题，在hexo的 _config.yml部署的repo地址改用ssh而不是用https，同时对ssh地址做Host别名替换，原有repo地址为repository: git@github.com:Name/Name.github.io.git，替换为repository: git@B:Name/Name.github.io.git。
在原有的~\.ssh\config中配置内容如下：

	# 该配置用于账户A
	Host A # Host 服务器别名
	HostName github.com  # HostName 服务器ip地址或机器名
	User A # User连接服务器的用户名
	IdentityFile C:\Users\yiye\.ssh\id_rsa # IdentityFile 密匙文件的具体路径
	# 该配置用于账户B
	Host B # Host 服务器别名
	HostName github.com # HostName 服务器ip地址或机器名
	User B # User连接服务器的用户名
	IdentityFile C:\Users\yiye\.ssh\id_rsa_new # IdentityFile 密匙文件的具体路径

这样ssh解析的时候就会自动把B转换为 github.com，push的时候系统就会根据不同的仓库地址使用不同的账号提交

# 问题：历史删除文件仍存在于repository中
原本删除的文件仍存在于生成的blog\public中，这是因为没有执行hexo clean命令

##问题3：Hexo deploy 发布不成功
始终停留在

	nothing to commit (working directory clean)
Hexo的issue中有提到这个问题哦，原因就是第一次设置错了，然后即使正确设置 Repository 再次 Deploy 的时候它也会报错：nothing to commit, working directory clean；error: src refspec master does not match any。所以，当重新设置 Repo 的时候要把 .deploy_git/ 文件夹删掉，让 Hexo 再次初始化，否则 Hexo 只是执行 push 操作，所以会一直报错。解决方法是删除.deploy_git

	rm -rf .deploy_git
	hexo generater
	hexo deploy


