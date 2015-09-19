title: 遍历所有子目录，动态创建grunt transport任务
date: 2013-10-27 16:33
categories: javascript
---
本文介绍动态创建grunt transport任务的方案
<!--more-->

在grunt-cmd-transport的GitHub官网提了issue，[是否支持遍历子目录，提取出变量用于配置](https://github.com/spmjs/grunt-cmd-transport/issues/55)。不过没有得到回答，今天只好自己摸索了一个办法

# 静态的配置

普通的transport配置一般是这样的：

```
transport: {
    options: {
        paths: ['webapps'], // where is the module, default value is ['sea-modules']
        alias: aliasInfo
    },
    employee: {
        options: {
            idleading: 'employee/static/'
        },
        files: [
            {
                cwd: 'webapps/employee/static',
                src: '/*.js',
                dest: '.build/employee/static'
            }
        ]
    },
    pos: {
        options: {
            idleading: 'pos/static/'
        },
        files: [
            {
                cwd: 'applications/pos/static',
                src: '/*.js',
                dest: '.build/pos/static'
            }
        ]
   }
```

主要是由于seajs对路径和模块ID一致性的要求，所以不同路径下的模块，需要不同的idleading，cwd，dest配置

如果模块不多的话，那么为每个模块单独创建一个target（如上）就可以了。但是我们的项目有很多子模块，子目录下又有子目录，需要创建的target太多了

而且如果后续目录结构又发生变化，就需要一直修改Gruntfile，非常不方便

所以就想是否可以去遍历base下的所有目录，并且提取出path，作为参数配置给idleading，cwd，dest。但是grunt-cmd-transport似乎没有这个特性

# 目录结构

![](http://img.blog.csdn.net/20131027160604078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

webapps是作为源码目录，同时也是开发阶段的seajs base（顶级标识从这里开始找）

下面每个模块是一个子目录，如果模块包含子模块，则有更深的一级子目录。在最终的模块下，有service，static，test，分别代表后端代码，前端代码，测试代码。static里又分为css，html，以及直接存放js

# 解决思路

总的思路是遍历所有子目录。提取出目录的path，然后创建新的transport target，配置参数并运行。本质上还是多个target，但是target不是静态写在Gruntfile里，而是根据遍历的结果动态创建的

但是还有一个问题，就是有些子目录是不希望grunt去扫描的，比如test，css，html等。那么遍历的时候就有2种做法，一种是直接只扫描参与构建的目录，另一种是扫描所有的目录，然后排除掉不要的

第一种方法当然是更好的，不过一时没想到该怎么做，或许用node glob module可以做到，最终我选择的是第二种办法

# Gruntfile

以下是我的grunt task代码

```
module.exports = function (grunt) {

    var fs = require("fs");

    grunt.initConfig({

        transport: {
            options: {
                paths: ['webapps']// set seajs base, where a module belongs to
            }
        }
        // concat, uglify, clean等不是本文主题，不显示
    });

    grunt.loadNpmTasks('grunt-cmd-transport');

    grunt.registerTask('recursive_transport', 'recursive every directory, and perform transport task', function () {

        var dirList = [];

        (function walk(path) {
            var items = fs.readdirSync(path);
            items.forEach(function (item) {
                if (fs.statSync(path + '/' + item).isDirectory()) {
                    dirList.push(path + '/' + item);
                    walk(path + '/' + item);
                }
            });
        })("webapps");// search all directories

        // 排除包含特定命名的目录，对目录命名有要求，可能引入BUG
        function isExcludeDir(dirName) {
            if (dirName.indexOf('test') > -1) {
                return true;
            }
            if (dirName.indexOf('service') > -1) {
                return true;
            }
            if (dirName.indexOf('css') > -1) {
                return true;
            }
            if (dirName.indexOf('html') > -1) {
                return true;
            }
            return false;
        }

        dirList.forEach(function (item, index) {

            item = item.substr(8);// cut the leading path "webapps/", then leaving "employee/static"

            if (!isExcludeDir(item)) {

                var targetName = "transport.build" + index;// such as 'transport.build0'

                grunt.config.set(targetName + '.options.idleading', item + '/');
                grunt.config.set(targetName + '.files', [
                    {
                        cwd: 'webapps/' + item,
                        src: '*.js',
                        dest: '.build/' + item
                    }
                ]);

                console.log("configuration success for: " + targetName);

                grunt.task.run(targetName.replace('.', ':'));
            }
        });
    });

    grunt.registerTask('default', ['recursive_transport']);
};
```
遍历目录的代码参考了这篇博客：

[nodejs目录遍历](http://www.swordair.com/blog/2012/05/923/)

代码的实现还是比较简单的，只说几个要点：

## 灵异事件

今天遇到一个灵异事件，执行grunt的时候，报什么找不到模块的错误。不知道是不是git版本控制的问题，或者是缓存什么的，总之很奇怪。最后我npm uninstall grunt，npm uninstall grunt-cmd-transport，总之是把跟grunt有关的包都卸载了，然后重新npm install，就可以了

## 与目录命名的耦合

isExcludeDir()这个函数可能有问题，只要路径中包含service，html，css，test，那么这个目录就不会参与构建了。如果刚好有某个子模块包含这几个关键字，比如htmlRender之类的也不行，这个可能会引入BUG。暂时以文档和命名规范的方式规避，后续这个函数还是需要优化一下

## 单线程与异步执行

上面的代码有一个陷阱，在dirList.forEach()内部，看代码似乎是配置一个新的target，然后马上执行transport:target，之后再处理下一个path。实际上不是这样，node是单线程执行的，而grunt.task.run()似乎是一个异步调用。所以实际上会一次性配置完所有的target，然后才开始执行grunt.task.run()

## 必须创建新的target

这个方法其实昨天就想到了，但是运行没有成功，只会反复构建同一个目录

因为昨天的代码写错了，昨天是这么写的：

```
grunt.config.set('transport.build.options.idleading', item + '/');
grunt.config.set('transport.build.files', [
    {
        cwd: 'webapps/' + item,
        src: '*.js',
        dest: '.build/' + item
    }
]);

grunt.task.run('transport:build');
```
这样就只有一个叫作'build'的target，只是同一个target配置了N次（只有最后一次生效），所以只会反复构建同一个目录