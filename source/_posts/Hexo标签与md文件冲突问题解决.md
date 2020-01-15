title: Hexo标签与md文件冲突问题解决
date: 2020-1-3 18:23:12
categories: 2020年1月
tags: [Hexo]

---

Hexo标签与md文件冲突问题解决

<!-- more -->

今天使用 Github+Hexo 搭建的博客写文章时遇到一个 Bug，查阅资料解决之后，发现很多人都遇到过由于 nunjucks 模板标签导致 MD 文件解析报错的问题，于是记录一下这个问题的解决方法。

# 报错及原因
  今天使用 hexo generate 生成文章时，出现了如下报错：

    INFO  Start processing
    FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
    Template render error: (unknown path) [Line 17, Column 30]
      unexpected token: }}
        at Object._prettifyError

出现上述情况的原因是 Markdown 文件中内容含有的双大括号 标签与 nunjucks 模板引擎的标签发生了冲突。双大括号等这些标签都是模板引擎的占位标签，如果 MarkDown 文件中包含这些标签，且不被 ``` 代码块包裹，那么解析时就会把 MD 文件中的标签动态解析了，于是导致 MD 文件解析时报错。

# 解决方法
## 修改 Markdown 文件
将含有双大括号的内容首尾添加如下标签进行处理：

    {% raw %}
    {{ 双大括号内包裹的内容 }}
    {% endraw %}
用这个标签虽然可以解决问题，但之后再遇到类似的情况每次都需要对 MarkDown 文件进行修改。下面介绍一种更便捷的方法。

## 修改 nunjucks 模块源代码
我们还可以直接在 nunjucks 模块上修改源代码，更改有冲突的渲染标签。
首先打开这个文件，路径如下：
    node_modules/nunjucks/src/lexer.js
找到下述两行代码：

    var VARIABLE_START = '{{';
    var VARIABLE_END = '}}';
将其修改为：

    var VARIABLE_START = '{$';
    var VARIABLE_END = '$}';
将有冲突的模板引擎的占位符更改为其他字符，进行模板解析时就不会与 MarkDown 的内容发生冲突了，且这种方法对所有 MarkDown 文件都是有效的，一劳永逸。
类似的，如果出现类似符号的解析错误时，也可以根据情况将其更改为其他占位符（自定义）。

## 搜索、RSS 插件同步修改

若博客使用 hexo-generator-search 或 hexo-generator-feed 等其他依赖于 nunjucks 模板的插件，那么这些插件的模板处理标签也需要进行同步修改。以搜索插件为例，打开如下路径的文件：
node_modules/hexo-generator-search/templates/search.xml
根据之前修改的 nunjucks 的内容，将此文件的 {{ 和 }} 也更改为 {$ 和 $} 即可。

# 注意：
如果在项目下执行 npm install 更新 nunjucks 模板时，那么之前更改的内容会被还原，需要重新对有冲突的符号进行更改。

# 参考文献
【1】https://www.jianshu.com/p/6032a1a2dc25
