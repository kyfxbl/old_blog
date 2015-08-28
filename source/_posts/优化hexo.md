title: 优化hexo
date: 2015-07-25 16:16:51
categories: 其它
---
最近开始玩hexo，它并非一个博客系统，而是一个静态博客生成工具。我参考了网上的一些文章，也做了一些优化。本文总结一下搭博客以来的一些经验。
<!--more-->

# hexo的介绍文章和教程

基础概念介绍和购买域名这些，本文就不涉及了，网上很多文章已经介绍了。推荐以下几篇：
[如何使用Jacman主题](http://wuchong.me/jacman/2014/11/20/how-to-use-jacman/)
[hexo系列教程](http://zipperary.com/2013/05/28/hexo-guide-1/)
[hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

# 添加好用的非默认插件

hexo环境搭建非常简单，只需要有node就可以
```
$ npm install hexo -g
```
然后执行hexo初始化，就会得到一个可工作的博客目录
```
$ hexo init dir_name
$ cd dir_name
$ npm install
```
但是这样仅安装了默认的插件，我推荐也安装以下4个插件，可以压缩js，css，并生成sitemap和rss
```
$ npm install hexo-clean-css --save
$ npm install hexo-uglify --save
$ npm install hexo-generator-feed --save
$ npm install hexo-generator-sitemap --save
```

# 导入csdn博客

我之前已经在csdn写了好几年博客，所以想把上面的文章迁移到新网站，没有找到现成的插件，就自己写了一个。如果你也有这个需求，可以试一下
```
$ npm install hexo-migrator-csdn --save
```
安装之后，执行
```
$ hexo migrate csdn username
```
如果要指定迁移某一篇博客
```
$ hexo migrate csdn user_name topic_id
```

# 选择一个主题

hexo一个有趣的地方在于，和MVC一样，它的内容和样式是完全分离的。/source/_post/*.md是数据，而网站样式是由theme决定的。因此只需要更换主题，就可以使整个网站改头换面。或者当你修改主题的时候，也不需要对博客内容做任何调整

主题有很多，我个人使用的是jacman，稍微修改了一点（主要是更新了百度统计和百度搜索）。安装主题也只需要一行命令，以本站使用的jacman为例：
```
$ git clone https://github.com/wuchong/jacman.git themes/jacman
```
或者，如果你想使用我的分支：
```
$ git clone https://github.com/kyfxbl/jacman.git themes/jacman
```
然后在全局的_config.yml中设置主题为jacman即可
```
theme: jacman
```
装好主题以后，会在目录中找到主题自己的_config.yml，里面配置的是主题相关的参数。基本上所有参数都有注释，都试试就行了

# 默认增加目录和标签

默认的脚手架在scaffolds/post.md，一开始只预设了title和date，建议把categories和tags加上，就不需要每次都重新填了
```
title: {{ title }}
date: {{ date }}
categories: 
tags:
---
```
每次hexo n之后，填上你需要的目录和标签就可以，填写标签需要用数组的方式，例如本文：
```
title: 优化hexo
date: 2015-07-25 16:16:51
categories: 其它
tags: [hexo, Jacman]
---
```

# 给文章添加摘要

把摘要和正文用
```
<!--more-->
```
分隔开即可，主题会自动处理这个标记，生成“阅读全文”按钮

# 添加站内搜索

站内搜索比较麻烦一点。默认情况下，主题里的几个search配置项都是关闭的，这里我选择百度站内搜索，就需要做如下配置：
```
baidu_search:
  enable: true
  id: "xxxxxxxxx" // 你的百度站内搜索id
  site: "xxxxxxx" // 你的二级域名，比如so.kyfxbl.com/cse/search
```
## 配置id和域名

首先需要到百度站内统计的后台，申请一个id填到这里，注意需要___加上引号___，因为百度的id是纯数字，hexo解析的时候会溢出。然后site建议填你自己的二级域名，然后从dns里转发到百度的CNAME

这里配置的参数，最终是用来生成右上角的搜索框，在header.ejs里：
```
<% } else if(theme.baidu_search.enable){ %>
    <form class="search" action="<%- theme.baidu_search.site %>" target="_blank">
	    <label>Search</label>
	    <input name="s" type="hidden" value= <%= theme.baidu_search.id %> >
	    <input type="text" id="bdcsMain" name="q" size="30" placeholder="<%= __('search') %>">
		<br>
    </form>
<% }
```
最后其实是跳转到百度站内搜索的结果url：http://zhannei.baidu.com/cse/search?s=xxx&q=xxx

## 添加代码

实际上我不清楚为什么要添加这段代码，核心是上面的那段url。不过还是按照百度官网的提示加了以下代码

首先给输入关键字的input增加
```
id="bdcsMain"
```
然后在after_footer.ejs的末尾增加
```
<!-- baidu search begin -->
<%- partial('baidu-search') %>
<!-- baidu search end-->
```
最后增加baidu-search.ejs
```
<% if (theme.baidu_search.enable){ %>

<script type="text/javascript">
    (function(){
        document.write(unescape('%3Cdiv id="bdcs"%3E%3C/div%3E'));
        var bdcs = document.createElement('script');
        bdcs.type = 'text/javascript';
        bdcs.async = true;
        bdcs.src = 'http://znsv.baidu.com/customer_search/api/js?sid=' + '<%= theme.baidu_search.id %>' + '&plate_url=' + encodeURIComponent(window.location.href) + '&t=' + Math.ceil(new Date()/3600000);
        var s = document.getElementsByTagName('script')[0];
        s.parentNode.insertBefore(bdcs, s);
    })();
</script>

<% } %>
```
这部分代码jacman的主干里没有，我的分支已经加上了

## 在百度后台配置

现在已经可以通过搜索框跳转到百度站内统计的结果页面，但是还需要对此页面进行配置，才能比较美观。另外站内的页面，也需要提交给百度收录，才能得到搜索结果。这些都需要在百度后台操作，跟hexo没有关系
[百度站内统计](http://zn.baidu.com/)
[百度站长平台](http://zhanzhang.baidu.com/)