---
title: 微信公众平台开发学习笔记I
date: 2020-01-20 15:18:12
categories: 2020年1月
tags: [Wechat]

---

在测试号中配置接口URL和Token遇到的问题及解决方法

<!-- more -->

# 在测试号中配置接口URL和Token

使用ngrok穿透工具为本地生成对外的网址：
https://dashboard.ngrok.com/get-started
注册后得到与本账户绑定的authtoken

    ./ngrok authtoken 1WNVahHoiVcDG9Hr6c4E86yQpbA_eX526mPNsmsXBkMnLLs6
启动HTTP隧道

    ./ngrok http 80

这条命令的意思是将本地80端口对应的服务暴露到外网中。
显示界面：

      Session Status   online                                       
      Account          Yiye Li (Plan: Free)                                 
      Version          2.3.35                             
      Region            United States(us)                                   
      Web Interface     http://127.0.0.1:4040                                             
      Forwarding   http://e5d20d55.ngrok.io -> http://localhost:80                 
      Forwarding   https://e5d20d55.ngrok.io -> http://localhost:80    
      Connections  ttl   opn   rt1   rt5   p50   p90                                     
       0       0       0.00    0.00    0.00    0.00  

第一个是http协议对应的外网地址，第二个是https协议对应的外网地址。这样，凡是访问http://e5d20d55.ngrok.io的请求都将发送到localhost:80。不付费的话每次启动ngrok都会分配一个新的外网域名，所以需要每次更换配置或者更换访问地址。

然后在https://mp.weixin.qq.com/debug/cgi-bin/sandboxinfo?action=showinfo&t=sandbox/index中输入https://e5d20d55.ngrok.io和token，token从本地项目的applicaiton.yml中获取。
但这时提交会发现配置失败，而且查看本地运行日志没有记录，原因是本地项目的端口号为9006，而URL是本地80端口对应的服务暴露到外网中，因此重新运行

      ./ngrok http 9006

然后提交改正后的本地映射地址,结果发现又失败了，原来是本地响应微信发送的token验证接口有url后缀，所以重新生成时要加上对应接口的后缀,例如新生成的是http://c6e2a3bf.ngrok.io，则填http://c6e2a3bf.ngrok.io/wechat,另外本地同时要启动项目才能校验。至此就可以成功提交。


# 参考资料：
【1】https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login
【2】https://mp.weixin.qq.com/debug/
【3】https://developers.weixin.qq.com/doc/offiaccount/Getting_Started/Overview.html
