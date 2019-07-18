---
title: Linux配置问题小结
date: 2019-6-3 12:47:45
categories: 2019年6月
tags: [Linux]

---
 

记录Linux配置问题的解决方案，在Ubuntu 18.04中安装terminator 并在右键菜单中添加open in termintor;解决了搜狗输入法在Ubuntu 18.04 中无法安装和安装后无法使用的问题;解决了Linux下无线网卡rtl8821CE/rtl8723ce驱动无法驱动的问题。

<!-- more -->



## 实现功能
1. Ubuntu 18.04 安装terminator 并在右键菜单中添加open in termintor
2. 解决了搜狗输入法在Ubuntu 18.04 中无法安装和安装后无法使用的问题
3. 解决了Linux下无线网卡rtl8821CE/rtl8723ce驱动无法驱动的问题。

## Terminator安装和右键菜单添加解决方案
  题外话，安利下terminator，是Linux环境下一款非常赞的命令行管理工具，前段时间微软宣布推出名为Windows Terminal的新款命令行工具，在个人看来也是姗姗来迟的将terminator能实现的功能在windows中实现了。

  安装terminator

    sudo apt install terminator

  网上大多用dconf系统配置编辑器将terminator设为默认，但在我的系统中这样在文件位置中右键打开时还是只能打开默认的终端，用起来很不方便，直到找到了这种方法（https://blog.csdn.net/bestBT/article/details/81221378）。
  安装Nautilus-actions

    sudo add-apt-repository ppa:daniel-marynicz/filemanager-actions
    sudo apt-get install filemanager-actions-nautilus-extension
使用fma-config-tool 来配置。
1.新建动作，在action（动作）界面中勾选第二项Display item in location context menu。在Context label中输入想写的名字“在Terminator中打开”。
2.在Command（命令）界面中输入路径/usr/bin/terminator，参数中输入--working-directory=%d/%b。
3.这样操作完在terminator中打开还在右键的二级菜单中，所以点右上角齿轮标志（编辑您的首选项），在运行时首选项-》Nautilus菜单布局中取消勾选Create a root "FileManager-Actions" menu，这样就可以愉快的使用Terminator了。

## 搜狗拼音输入法安装问题解决方案
一开始嫌麻烦所以用Linux自带的Ibus系统下的输入法，但各种问题实在忍不了，所以还是装了搜狗拼音法，中间遇到几个问题只找到这个靠谱的解决方法（https://ywnz.com/linuxjc/1476.html）。

在根据指导方法安装搜狗拼音输入法后，会遇到很多依赖问题，一路折腾后会发现搜狗拼音依赖的fcitx-libs-qt这个包在ubuntu18.04的源里面是不存在的。替代品似乎是libfcitx-qt0，但也很难搞。
### 下载旧版
搜狗官方的最新版（sogoupinyin_2.2.0.0102_amd64.deb）是不能用的，原因不作深究。需要下载一个旧版本（sogoupinyin_2.1.0.0086_amd64.deb）。
将该deb包放置于一个空目录。
### 创建软件包目录

    mkdir -p extract/DEBIAN

### 解包

    dpkg-deb -x sogoupinyin_2.1.0.0086_amd64.deb extract/

    dpkg-deb -e sogoupinyin_2.1.0.0086_amd64.deb extract/DEBIAN
### 修改control文件
用任意文本编辑器打开extract/DEBIAN/control 找到Depends行，删除fcitx-libs和fcitx-libs-qt，保存退出。
其中要找的Depends行长这个样子

    Depends: fcitx (>= 1:4.2.8.3-3~), fcitx-frontend-gtk2, fcitx-frontend-gtk3, fcitx-frontend-qt4, fcitx-module-kimpanel, im-config, libopencc2 | libopencc1, lsb-release, unzip, x11-utils, libc6 (>= 2.8), libgcc1 (>= 1:4.1.1), libgdk-pixbuf2.0-0 (>= 2.22.0), libglib2.0-0 (>= 2.16.0), libidn11 (>= 1.13), libnotify4 (>= 0.7.0), libqt4-dbus (>= 4:4.8.0), libqt4-declarative (>= 4:4.8.0), libqt4-network (>= 4:4.8.0), libqtcore4 (>= 4:4.8.0), libqtgui4 (>= 4:4.8.0), libstdc++6 (>= 4.6), libx11-6, zlib1g (>= 1:1.2.0)

### 打包

    mkdir build

    dpkg-deb -b extract/ build/

执行完毕build目录下会生成一个新的deb包。

### 安装

先彻底卸载掉搜狗输入法，并删除以下配置文件：

.sogouinput

.config/fcitx

.config/sogou-qimpanel

.config/SogouPY

.config/SogouPY.users

.config/fcitx-qimpanel

然后安装我们重新打包的输入法即可。重启之后，搜狗输入法恢复正常。
最后要记得在设置-》区域和语言-》管理已安装的语言-》语言支持-》语言-》键盘输入法系统中选择Fcitx系统，因为搜狗输入法等都是在Fcitx系统框架下的。



## LINUX下无线网卡rtl8821CE/rtl8723de驱动无法驱动解决办法

### 1. 确保linux内核版本大于 4.14
  如何查看linux 内核版本 ：终端 uname -sr

  如果内核版本低于 4.14：升级linux内核 ubuntu可以参照 https://www.linuxidc.com/Linux/2017-03/141940.html

  升级完记得重启

### 2. 下载linux中8821CE/rtl8723de的驱动源码

git原地址（rtl8821CE）：https://github.com/endlessm/linux/tree/master/drivers/net/wireless/rtl8821ce

git原地址（rtl8723de）：https://github.com/endlessm/linux/tree/master/drivers/net/wireless/rtl8723de

或者本地下载 https://free-1253146430.cos.ap-shanghai.myqcloud.com/rtl8821ce.zip

（rtl8723de 的话 自己 去git上下吧）

### 3.编译驱动
  ***解压rtl8821ce.zip***

  ***修改文件Makefile***

    export TopDIR ?= $(srctree)/drivers/net/wireless/rtl8821ce

从这行 “export TopDIR ?= 后面改成当前目录 例如我的：

    export TopDIR ?= /home/horsun/Downloads/rtl8821ce

***保存修改***

  分别进行：

      make
      sudo make install
      sudo modprobe -a 8821ce

遇到问题

    modprobe: ERROR: could not insert '8812au': Exec format error
***执行***

    make clean
    make
    sudo make install
    sudo modprobe 8812au
