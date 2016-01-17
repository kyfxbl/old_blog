title: node的模块加载与管理
date: 2013-10-22 17:16
categories: javascript 
---
node模块管理的总结
<!--more-->

# 参考文档

以下几篇文档比较重要：

[CommonJS module spec](http://wiki.commonjs.org/wiki/Modules/1.1.1)

[CommonJS package spec](http://wiki.commonjs.org/wiki/Packages/1.0#Catalog_Properties)

[npm install](https://npmjs.org/doc/cli/npm-install.html)

[node package.json](https://npmjs.org/doc/json.html#dependencies)

[node module reference](http://www.nodejs.org/api/modules.html)

# node模块

node实现了CommonJS的模块规范和包结构规范

模块规范（module spec）主要是定义不同模块间require，exports，module等API

包结构规范（package spec）主要是定义目录结构，以及package.json的格式。另外node在package.json中还扩展了一些自定义的字段，其中最重要的就是main，后面会提到，当require一个目录时，该字段描述了哪个js文件作为入口

实际上，即使不遵循包结构规范，比如没有package.json，js文件之间相互引用也是可以的，比如：

![](http://img.blog.csdn.net/20131022163605250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里只有一个单独的calculator.js，显然谈不上什么包结构，但是这个模块也完全可以被其他模块引用

```
var cal = require("c:/calculator");
var sum = cal.add(1, 2);
console.log(sum);
```

# 包结构规范

但是，如果想要方便地管理模块，或者提供给别人使用，那么就需要遵循包结构规范。同时node也提供了npm来管理node module

一个典型的node module，通常是一个单独的目录，放在node\_modules下。目录下有lib，bin等子目录，以及package.json描述文件，比如：

![](http://img.blog.csdn.net/20131022164410937?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

package.json是核心，其中描述了该模块的入口，模块依赖的模块等。用npm install命令，可以自动读取分析package.json中描述的依赖，并安装到本地仓库（放在node\_modules下）

# 模块安装

安装模块通常有3种情况

## 全局安装

典型的比如grunt-cli，使用npm install -g xxx命令。有些公用的模块，后续需要用命令行来执行的，一般用这种方式安装

## 自动分析安装所需依赖

这种不需要在命令行参数里指定目标module name，只要执行npm install，就会读取并分析package.json中声明的依赖，然后下载安装

## 将自己开发的模块打包

如下目录

![](http://img.blog.csdn.net/20131022165452015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在test\_npm中，是这样引用test模块的：

```
var test = require("test");
```
这行代码执行不能成功，需要先执行npm install test，把test安装到本地仓库

![](http://img.blog.csdn.net/20131022165804312?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其实除非是为了将自己开发的模块发布到npm registry或是npm source上，<span style="color:#ff0000">一般没必要安装本地模块</span>，因为可以通过相对路径或者绝对路径加载到

# 模块加载规则

这个特别重要，要详细看这篇文档：[node module reference](http://www.nodejs.org/api/modules.html)

大致上有3种情况：

```
require("http");
```
这种是加载核心模块，包括http，fs，net等

```
require("mongo");
require("mysql");
require("express");
```
这种类似加载核心模块，不以"../"、"./"、"/"开头，但是请求的是第三方模块，会从node\_modules里加载，然后依次查找上层目录的node\_modules，直到/node\_modules，如果还是没找到，则抛出错误

```
require("./abc");
require("../def");
require("/ghi");
```
这种是根据路径加载

# 总结

正式开发的项目，显然应该按照node的包结构规范来组织目录。最明显的好处是，可以在package.json里声明依赖，然后就可以很方便地用npm install来安装所需的第三方模块

对于server端开发来说，自己写的代码，用路径来require就挺好，最好不要用模块名来require。否则的话，模块一修改，就需要重新npm install，非常麻烦，又没有明显的好处。相反用路径来require，模块的修改马上就能体现出来，开发很方便

个人感觉，相对路径比绝对路径更好。只要应用的目录规划没变，只是部署的路径改变，相对路径都不需要修改