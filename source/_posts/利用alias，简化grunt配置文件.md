title: 利用alias，简化grunt配置文件
date: 2013-10-25 20:41
categories: javascript 
---
最近要把一个seajs项目用grunt构建，使用了grunt-cmd-transport和grunt-cmd-concat
<!--more-->

# 关键点

其中有2个要点：

1、<span style="color:#ff0000">源码目录和构建目录，目录结构要保持一致</span>，这个是最重要的。这样require()无论在开发环境还是生产环境，都能加载到需要的模块。至于require()是用相对标识还是用顶级标识，倒是无所谓

```
require("path/to/module");
require("../module/file");
```
这2种都可以，只要目录结构不变就行了

2、用grunt-cmd-transport提取moduleId时，idleading一定要写对，因为seajs要求路径和模块标识要一致

# alias参数的作用

参考这个帖子：[grunt-cmd-transport的alias参数](https://github.com/spmjs/grunt-cmd-transport/issues/54)

如果用spm构建，spm会自动修改require()里的参数，比如：

```
require("utils/src/utils");
```

在package.json中：
```
"spm": {
    "alias":{
        "utils/src/utils":"planx/utils/1.0.0/utils"
     }
}
```
则执行spm build之后，上面那行require()代码会自动变成：
```
require("planx/utils/1.0.0/utils")
```

使用grunt-cmd-transport插件，如果配置了alias，也可以起到自动修改require()中参数的效果

问题是比如

```
require("uiframework/staitc/package")
```

设置alias：
```
alias: {"uiframework/static/package":"abc"}
```

构建的时候会报错：
Warning: can't find module abc Use --force to continue.

所以在执行构建命令的时候，使用--force参数，强制进行。这样transport之后的.js，也会自动转换require()参数

# 利用alias参数简化Gruntfile和require

alias参数还有另一个作用，作用就跟seajs.config()里的alias一样

一般中型的项目，目录结构都会比较深，比如：

```
require("framework/static/util/date");
```

写起来就会很费劲，写成下面这样显然简单多了：

```
require("date");
```

然后在seajs.config()里配置alias：

```
seajs.config({
    alias: {
        'date': 'frame/static/util/date'
    }
});
```
在页面加载时这样就能生效，因为会以seajs的加载路径作为base，搜索到顶级标识

但是用grunt构建时，并没有seajs base存在，因此就需要配置：

```
options: {
    alias: {
        "date": "frame/static/util/date"  
    }
}
```
这样无论是开发还是构建，用alias都没问题了

另外，构建后的require()，已经是完整的顶级标识，不是alias缩写，所以构建后的html里，就不需要再次配置seajs.config()了

# 提取出单独的元数据

不过上面的做法还是不太理想，主要有2个问题：

1、每次增加或者减少alias，都需要到Gruntfile.js和html里修改，很不方便

2、Gruntfile.js和html里的alias配置是重复的，如果漏改一处就会有问题

所以更好的办法，是把alias配置单独放到一个配置文件里，Gruntfile和html一起去读取，这样就没问题了：

```
// alias_info.json

{
    "utils": "uiframework/static/utils/utils",
    "dialog": "uiframework/static/widgets/dialog/dialog",
    "datas": "uiframework/static/utils/datas",
    "keyHandler": "uiframework/static/utils/keyHandler",
    "errorHandler":"uiframework/static/utils/errorHandler"
}
```

```
var aliasInfo = grunt.file.readJSON('alias_info.json');

    grunt.initConfig({

        transport: {
            options: {
                alias: aliasInfo
            },

        // 省略……
```