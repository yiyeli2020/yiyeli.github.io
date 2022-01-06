---
title: Grafana汉化指南

date: 2022-01-06 19:12:12

categories: 2022年1月

tags: [Grafana]

---

Grafana汉化指南,仅供学习使用。

<!-- more -->


[TOC]

主要步骤参考自[^1]

# grafana汉化步骤：

CentOs 7.6 x64环境

## 第一步：安装nodejs和yarn，nodejs版本要大于12.0.0

如何安装nodejs和yarn？参考以下链接

https://yarn.bootcss.com/docs/install/#centos-stable

或直接使用以下命令安装

    # curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | tee /etc/yum.repos.d/yarn.repo
    
    # curl --silent --location https://rpm.nodesource.com/setup_12.x | bash -
    
    # yum install -y nodejs
    
    # yum install -y yarn



## 第二步：到github（https://github.com/grafana/grafana/tree/v6.7.x）上找到grafana源码

选择对应的分支，直接Dowload ZIP文件。不需要clone整个仓库，太慢太耗时间。



## 第三步：指定yarn源
    
    # yarn config set registry https://registry.npm.taobao.org --global
    
    # yarn config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/ -g
    
    # yarn config get registry
    


## 第四步：安装依赖和构建
    
    # cd grafana-6.7.x
    
    # yarn install
    
    # yarn build



## 第五步：确认能编译成功（无报错），然后将编译后的目录同步到/usr/share/grafana/

首次同步前，先备份/usr/share/grafana/public目录
    
    # mv /usr/share/grafana/public  /usr/share/grafana/public.source

将当前编译目录下的public目录拷贝到/usr/share/grafana/

    # /bin/cp -rp public/ /usr/share/grafana/

重启grafana-server

systemctl restart grafana-server

确认编译环境正常后，开始第六步。



## 第六步：汉化，然后重复第四、第五步。





windows 10设置编译环境

node下载: https://nodejs.org/en/

yarn下载：https://classic.yarnpkg.com/zh-Hans/docs/install#windows-stable

下对安装包后直接双击安装。


    
    yarn config set registry https://registry.npm.taobao.org -g
    
    yarn config set sass_binary_site http://cdn.npm.taobao.org/dist/node-sass -g
    
    yarn config get registry
    
    
    
    yarn install 
    
    yarn build



注：在编译时，系统内存要大于4GB





以下是例出几个汉化的页面。

修改登录页

    app/core/components/Login/LoginForm.tsx
    
    
    
    修改Change Password
    
    app/features/profile/ChangePasswordPage.tsx 
    
    app/features/profile/ChangePasswordForm.tsx



修改偏好设置Preferences

    app/features/org/SelectOrgCtrl.ts
    
    app/core/components/SharedPreferences/SharedPreferences.tsx
    
    app/features/admin/partials/edit_user.html
    
    app/features/profile/ChangePasswordForm.tsx

总结：若时间充裕以及有需求，后续会考虑将Grafana 6.7.4做一个完整的汉化版。

# Grafana侧边栏汉化

步骤补充如下：但如果侧边菜单栏要中文化还需要修改Grafana源码中用go语言编写的代码。

## 第一步：安装go并设置环境变量
 
    # wget https://gomirrors.org/dl/go/go1.13.12.linux-amd64.tar.gz
    # tar xvf go1.13.12.linux-amd64.tar.gz -C /usr/local

设置环境变量
    
    export GOROOT=/usr/local/go
    export PATH=$PATH:/usr/local/go/bin
    export GOPATH=/home/go/src
    export GO111MODULE=on
    export GOPROXY=https://goproxy.io

## 第二步：将下载下来的Grafana源码移到GOPATH对应路径下
如在克隆grafana时非常慢，可以到gitee进行下载

    ##git clone https://gitee.com/mirrors/grafana.git
    cd $GOPATH/src/github.com/grafana/grafana
## 第三步：编译

    go run build.go setup
    go run build.go build    

## 第四步：备份grafana-server二进制文件，并使用新编译的grafana-server二进制文件

    mv /usr/sbin/grafana-server /usr/sbin/grafana-server.source
    systemctl stop grafana-server
    sleep 1
    cp -rp ./bin/linux-amd64/grafana-server /usr/sbin/
    sleep 1
    systemctl start grafana-server
## 第五步：中文化go部分代码后，只需重复第三、四步。

[^1]: https://www.bilibili.com/read/cv6735751?spm_id_from=333.999.0.0 