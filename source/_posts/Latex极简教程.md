
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


到了2021年这个目录也要换成21年的，安装包可以在https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/mac/mactex/里下载BasicTex.pkg

特别注意，如果你是 macOS 用户，那么今年的安装与往年的有点不一样，主要是 texlive 的文件结构稍有改变，在配置 PATH、MAN、INFO 等环境变量时，应特别注意。以下是 2021 版 TeXlive 的 macOS 环境变量配置： 

```
    export MANPATH=/usr/local/texlive/2021/texmf-dist/doc/man:${MANPATH}
    export INFOPATH=/usr/local/texlive/2021/texmf-dist/doc/info:${INFOPATH}
    export PATH=/usr/local/texlive/2021/bin/universal-darwin:${PATH}
    alias tlmgr='sudo /usr/local/texlive/2021/bin/universal-darwin/tlmgr'
```

PATH 里的 universal-darwim 代表此后无论是 Intel Core 还是 M1 芯片，都将得到适配。不过 M1 的兼容性此处未验证。特别强调，下文的配置是全平台通用的，无论你的操作系统是 macOS 还是 Windows，抑或是 Linux，只要不是太小众，应该都没问题。全网只有一个 texlive 的 iso 镜像，并没有特殊适配某个操作系统或某种 CPU 体系结构的版本。


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