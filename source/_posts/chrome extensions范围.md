title: chrome extensions范围
date: 2013-09-24 11:22
categories: 其它 
---
chrome的扩展（extensions），和插件（plugins）相比，能做的事情是比较有限的 
<!--more-->

# 能力范围

extensions基本上可以做2类事情： 

1、与原始页面的内容交互，比如获取DOM里的内容，注入javascript脚本执行等 

2、与浏览器交互，比如操作chrome的windows、tabs，访问chrome的书签、历史记录等 

做相应的操作，需要在manifest文件里声明权限，在用户安装的时候，会提示用户，由用户决定是否允许（跟android app是一个意思） 

范围基本局限在上述2点，像访问本地硬盘、调用本地应用等，还是不支持的。如果要做本地应用的一些事情，用extension是做不到的，必须用plugin，基于NPAPI 

# 扩展的形式

扩展主要有以下几种形式： 

## browser action或page action 

这种形式的扩展会在chrome地址栏（omnibox）那里增加一个按钮，允许用户点击 

然后可以弹出一个popup窗口，在popup html里引用的javascript，操作的就是popup的DOM，而不是原始页面的DOM

```
function click(e) {
  chrome.tabs.executeScript(null, {code:"document.body.style.backgroundColor='" + e.target.id + "'"});
  window.close();
}

document.addEventListener('DOMContentLoaded', function() {
  var divs = document.querySelectorAll('div');
  for (var i = 0; i < divs.length; i++) {
    divs[i].addEventListener('click', click);
  }
});
```
如上述的代码，document、window，都是popup里面的DOM元素，而不是原始页面里的元素 

## background javascript 

也可以不设置action按钮，通过background加载长期后台运行的javascript脚本，做一些操作，比如在后台运行一个定时任务等 

## content script 

扩展还可以定义一系列content script脚本，注入到原始页面里执行 

content script是在一个特殊环境中运行的，这个环境叫做isolated world（隔离环境）。它们可以访问所注入页面的DOM，但是不能访问里面的任何javascript变量和函数。对每个content script来说，就像除了它自己之外再没有其它脚本在运行；反过来也是成立的：页面里的javascript也不能访问content script中的任何变量和函数 

隔离环境使得content script可以修改它的javascript环境而不必担心会与这个页面上的其它content script冲突。例如，一个content script可以包含jquery v1，而页面可以包含jquery v2，它们之间不会产生冲突 

另一个重要的优点是隔离环境可以将页面上的脚本与扩展中的脚本完全隔离开。这使得开发者可以在content script中提供更多的功能，而不让web页面利用它们 

尽管content script的执行环境与所在的页面是隔离的，但它们还是共享了页面的DOM。如果页面需要与content script通信（或者通过content script与extension通信），就必须通过这个共享的DOM 

关于chrome extensions的概述，没有比官方overview更好的文章： 

英文版：[http://developer.chrome.com/extensions/overview.html](http://developer.chrome.com/extensions/overview.html) 
360翻译中文版：[http://open.chrome.360.cn/extension_dev/overview.html](http://open.chrome.360.cn/extension_dev/overview.html)