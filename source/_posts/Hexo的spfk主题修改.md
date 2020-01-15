title: Hexo的spfk主题修改
date: 2020-1-15 11:34:12
categories: 2020年1月
tags: [Hexo]

---

Hexo的spfk主题修改

<!-- more -->

新年新气象，使用Hexo的spfk主题博客很久了，想要做一些主题上的改动，准备在保持简洁风格前提下适当加一些有趣的小功能。

# Hexo博客添加live2d卡通人物
因为之前自己对卡通形象不太感冒所以没有在博客里加入卡通人物，这次在live2d里找到了几个还比较有趣的形象。
参见：
https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md
Hexo
## 安装模块:

    npm install --save hexo-helper-live2d
## 配置：

请向Hexo的_config.yml 文件或主题的_config.yml 文件中添加配置:

    live2d:
      enable: true
      scriptFrom: local
      pluginRootPath: live2dw/
      pluginJsPath: lib/
      pluginModelPath: assets/
      tagMode: false
      debug: false
      model:
        use: live2d-widget-model-wanko
      display:
        position: right
        width: 150
        height: 300
      mobile:
        show: true
      react:
        opacity: 0.7

## 模型
有许多方法来使用不同的模型:

a. live2d_models子目录名称
在博客根目录下创建一个 live2d_models 文件夹.

在此文件夹内新建一个子文件夹.

将 Live2D 模型复制到这个子文件夹中.

将子文件夹的名称输入_config.yml 的 model.use 中.

示例

    你的模型叫 mymiku.

    在博客根目录 (应当有_config.yml 、sources 、 themes ) 新建名为 mymiku 的子文件夹.

    将模型复制到 /live2d_models/mymiku/ 中.

    现在, 在这里应当有一个 .model.json 文件 (例如 mymiku.model.json)

    在 /live2d_models/mymiku/ 中.

    将 mymiku 输入到位于_config.yml 的 model.use 中.

b. 相对于博客根目录的自定义路径
可直接输入相对于博客根目录的自定义路径到 model.use 中.

示例: ./wives/wanko

c. npm 模块名
使用现有的的模型
需要先使用 npm install 模型的包名 来安装,

然后将包名输入位于_config.yml 的 model.use 中.


# 参考资料
【1】https://cniter.github.io/posts/b1e9411b.html
【2】https://blog.csdn.net/qq_36759224/article/details/85420403
【3】https://zhuanlan.zhihu.com/p/69213954
【4】https://www.itrhx.com/2018/08/27/A04-Hexo-blog-topic-personalization/
【5】https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md
