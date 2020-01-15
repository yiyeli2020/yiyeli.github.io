title: Hexo的spfk主题修改
date: 2020-1-15 11:34:12
categories: 2020年1月
tags: [Hexo]

---

Hexo的spfk主题修改

<!-- more -->

新年新气象，使用Hexo的spfk主题博客很久了，想要做一些主题上的改动，准备在保持简洁风格前提下适当加一些有趣的小功能。

# 添加字数统计和阅读时长

先在博客目录下执行以下命令安装 hexo-wordcount 插件：

    $ npm i --save hexo-wordcount
同样的，以 spfk 主题为例，在 \themes\hexo-theme-spfk\layout\_partial\post 目录下创建 word.ejs 文件，在 word.ejs 文件中写入以下代码：

    <div style="margin-top:10px;">
        <span class="post-time">
          <span class="post-meta-item-icon">
            <i class="fa fa-keyboard-o"></i>
            <span class="post-meta-item-text">  字数统计: </span>
            <span class="post-count"><%= wordcount(post.content) %>字</span>
          </span>
        </span>
        &nbsp; | &nbsp;
        <span class="post-time">
          <span class="post-meta-item-icon">
            <i class="fa fa-hourglass-half"></i>
            <span class="post-meta-item-text">  阅读时长: </span>
            <span class="post-count"><%= min2read(post.content) %>分</span>
          </span>
        </span>
    </div>

然后在 \themes\hexo-theme-spfk\layout\_partial\article.ejs 中适当位置添加以下代码：
添加前：

    <% if (post.link || post.title){ %>
      <header class="article-header">
        <%- partial('post/title', {class_name: 'article-title'}) %>
      </header>

添加后：

    <% if (post.link || post.title){ %>
      <header class="article-header">
        <%- partial('post/title', {class_name: 'article-title'}) %>
      <% if(theme.word_count && !post.no_word_count){ %>
        <%- partial('post/word') %>
        <% } %>
      </header>
最后在主题目录下的 _config.yml 添加以下配置

    word_count: true

另外：要在博客底部显示所有文章的总字数，可以点击此处，根据你博客底部文件的类型选择相应的代码放在适当的位置即可，前提是要安装好 hexo-wordcount 插件，例如我使用 spfk 主题，在 \themes\material-x\layout\_partial 目录下的 footer.ejs 文件中添加如下代码：

    <i class="fas fa-chart-area"></i>
    <span class="post-count">字数统计：<%= totalcount(site) %></span>

添加前的位置：

    <div class="outer">
        <div id="footer-info">
            <div class="footer-left">
                &copy; <%= date(new Date(), 'YYYY') %> <%= config.author || config.title %>
            </div>
            <div class="footer-right">
                <a href="http://hexo.io/" target="_blank">Hexo</a>  Theme <a href="https://github.com/luuman/hexo-theme-spfk" target="_blank">spfk</a> by luuman
            </div>
        </div>

添加后的位置：

            <div class="outer">
                <div id="footer-info">
                    <div class="footer-left">
                        &copy; <%= date(new Date(), 'YYYY') %> <%= config.author || config.title %>
                    </div>
                    <i class="fas fa-chart-area"></i>
                        <span class="post-count">总字数：<%= totalcount(site) %></span>
                    <div class="footer-right">
                        <a href="http://hexo.io/" target="_blank">Hexo</a>  Theme <a href="https://github.com/luuman/hexo-theme-spfk" target="_blank">spfk</a> by luuman
                    </div>
                </div>



# 添加网站运行时间

一个比较好的小功能，可以看见自己的博客运行多久了，时间一天天的增加，成就感也会一天天增加的
在 \themes\hexo-theme-spfk\layout\_partial\footer.ejs 文件下添加以下代码：

    <span id="timeDate">载入天数...</span><span id="times">载入时分秒...</span>
    <script>
        var now = new Date();
        function createtime() {
            var grt= new Date("11/16/2017 21:14:20");//在此处修改你的建站时间，格式：月/日/年 时:分:秒
            now.setTime(now.getTime()+250);
            days = (now - grt ) / 1000 / 60 / 60 / 24; dnum = Math.floor(days);
            hours = (now - grt ) / 1000 / 60 / 60 - (24 * dnum); hnum = Math.floor(hours);
            if(String(hnum).length ==1 ){hnum = "0" + hnum;} minutes = (now - grt ) / 1000 /60 - (24 * 60 * dnum) - (60 * hnum);
            mnum = Math.floor(minutes); if(String(mnum).length ==1 ){mnum = "0" + mnum;}
            seconds = (now - grt ) / 1000 - (24 * 60 * 60 * dnum) - (60 * 60 * hnum) - (60 * mnum);
            snum = Math.round(seconds); if(String(snum).length ==1 ){snum = "0" + snum;}
            document.getElementById("timeDate").innerHTML = "本站已安全运行 "+dnum+" 天 ";
            document.getElementById("times").innerHTML = hnum + " 小时 " + mnum + " 分 " + snum + " 秒";
        }
    setInterval("createtime()",250);
    </script>

# 访问量统计

使用不蒜子提供的服务，因为其域名更改，所以把原有的：

    <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
域名改一下即可：

    <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>

# 安装脚本（必选）
要使用不蒜子必须在页面中引入busuanzi.js，目前最新版如下。

    <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js">
    </script>
不蒜子可以给任何类型的个人站点使用，如果是用的hexo，打开themes/主题/layout/_partial/footer.ejs添加上述脚本即可，当然你也可以添加到 header 中。

# 安装标签（可选）
只需要复制相应的html标签到网站要显示访问量的位置即可。可以随意更改不蒜子标签为自己喜欢的显示效果，内容参考第三部分扩展开发。根据要显示内容的不同，这分几种情况。

## 显示站点总访问量
要显示站点总访问量，复制以下代码添加到你需要显示的位置。有两种算法可选：

算法a：pv的方式，单个用户连续点击n篇文章，记录n次访问量。

    <span id="busuanzi_container_site_pv">
        本站总访问量<span id="busuanzi_value_site_pv"></span>次
    </span>
算法b：uv的方式，单个用户连续点击n篇文章，只记录1次访客数。

    <span id="busuanzi_container_site_uv">
      本站访客数<span id="busuanzi_value_site_uv"></span>人次
    </span>
如果你是用的hexo，打开themes/主题/layout/_partial/footer.ejs添加即可。

# 显示单页面访问量
要显示每篇文章的访问量，复制以下代码添加到需要显示的位置。

算法：pv的方式，单个用户点击1篇文章，本篇文章记录1次阅读量。

    <span id="busuanzi_container_page_pv">
      本文总阅读量<span id="busuanzi_value_page_pv"></span>次
    </span>
代码中文字是可以修改的，只要保留id正确即可。


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
        use: live2d-widget-model-gf
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

但在安装过程中遇到问题：

    make: *** [Release/obj.target/fse/fsevents.o] Error 1
    gyp ERR! build error
    gyp ERR! stack Error: `make` failed with exit code: 2
    gyp ERR! stack     at ChildProcess.onExit (/usr/local/lib/node_modules/npm/node_modules/node-gyp/lib/build.js:262:23)
    gyp ERR! stack     at ChildProcess.emit (events.js:203:13)
    gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:272:12)
    gyp ERR! System Darwin 18.7.0
    gyp ERR! command "/usr/local/Cellar/node/12.6.0/bin/node" "/usr/local/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "build" "--fallback-to-build" "--module=/Users/liyiye/yiye-project/liyiye012.github.io/node_modules/fsevents/lib/binding/Release/node-v72-darwin-x64/fse.node" "--module_name=fse" "--module_path=/Users/liyiye/yiye-project/liyiye012.github.io/node_modules/fsevents/lib/binding/Release/node-v72-darwin-x64" "--napi_version=4" "--node_abi_napi=napi"
    gyp ERR! cwd /Users/liyiye/yiye-project/liyiye012.github.io/node_modules/fsevents
    gyp ERR! node -v v12.6.0
    gyp ERR! node-gyp -v v3.8.0
    gyp ERR! not ok
    node-pre-gyp ERR! build error
    node-pre-gyp ERR! stack Error: Failed to execute '/usr/local/Cellar/node/12.6.0/bin/node /usr/local/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js build --fallback-to-build --module=/Users/liyiye/yiye-project/liyiye012.github.io/node_modules/fsevents/lib/binding/Release/node-v72-darwin-x64/fse.node --module_name=fse --module_path=/Users/liyiye/yiye-project/liyiye012.github.io/node_modules/fsevents/lib/binding/Release/node-v72-darwin-x64 --napi_version=4 --node_abi_napi=napi' (1)

类似问题在https://github.com/nodejs/node-gyp/issues/1547中找到：

执行命令：

    sudo npm i --unsafe-perm

    sudo npm audit fix --force

    sudo npm install live2d-widget-model-gf

但做完这个功能后个人感觉有点花哨，容易分散阅读注意力，就暂时不上线了。


# 参考资料
【1】https://cniter.github.io/posts/b1e9411b.html
【2】https://blog.csdn.net/qq_36759224/article/details/85420403
【3】https://zhuanlan.zhihu.com/p/69213954
【4】https://www.itrhx.com/2018/08/27/A04-Hexo-blog-topic-personalization/
【5】https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md
【6】http://ibruce.info/2015/04/04/busuanzi/
