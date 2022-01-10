---
title: Hexo与Nodejs版本不匹配问题解决方案

date: 2022-01-10 16:12:12  

categories: 2022年1月

tags: [Hexo]

---

Hexo与Nodejs版本不匹配问题解决方案，升级hexo或者降级nodejs

<!-- more -->

[TOC]

# 部署hexo报错

    FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
    TypeError [ERR_INVALID_ARG_TYPE]: The "mode" argument must be integer. Received an instance of Object
        at copyFile (node:fs:2774:10)

# 错误原因
hexo暂时不能在新版的nodejs环境下运行

# 解决方法

降级nodejs即可，卸载现有nodejs，安装旧版nodejs。

如果有nvm的话，可以直接用nvm切换版本

查看当前的node版本

    node -v

目前是
v16.13.1

查看安装过的node版本 

    nvm list

安装某个node版本 

    nvm install <version>
    nvm install 12

使用某个node版本 

    nvm use<version>
    nvm use 12
卸载某个node版本

    nvm uninstall <version>








