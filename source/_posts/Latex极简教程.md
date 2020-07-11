
title: Latex极简教程

date: 2020-7-10 23:55:12

categories: 2020年7月

tags: [Latex]

---

在mac上快速安装使用Latex

<!-- more -->


# 快速安装
步骤参照[^1]
Latex 是个复杂但是很强大的排版工具，在 MAC 系统上如果不想安装 3G 大的 MacTex 的话，可以试试 BasicTex。

安装
MacTex 安装包非常大，而且自带了很多图形应用。我更喜欢用命令行，所以选择 BasicTex。使用 Homebrew 安装非常简单，一条命令即可。
```
brew cask install basictex
```
安装完还不能直接使用，还需要把 texlive 添加到环境变量中，才能找到相关的命令。
```
export PATH=/usr/local/texlive/2020basic/bin/x86_64-darwin:$PATH
```
然后就是安装相关的包，以及更新包,要cd到目录/usr/local/texlive/2020basic/bin/x86_64-darwin中运行下列命令。
```
sudo tlmgr update --self --repository http://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet
sudo tlmgr install latexmk --repository http://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/tlnet
```
安装包的时候推荐使用清华的 CTAN 镜像，不然的话下载速度实在太慢了。

# Atom开发
配合Atom来运行latex，参照[^2]

安装好插件后会报错缺失一些包可以更新插件或者npm install相应的包

# 命令行直接编译

```
xelatex demo.tex
```

# 遇到错误 ``! LaTeX Error: File `xifthen.sty' not found``

那么去搜索一下它被包含在哪个包中，比如 xifthen.sty 就是包含在 xifthen 中。那么使用 tlmgr 安装即可。
```
sudo tlmgr install xifthen
```

[^1]:https://www.cnblogs.com/ouyangsong/p/9348175.html 

[^2]:https://blog.csdn.net/violet_echo_0908/article/details/78160273