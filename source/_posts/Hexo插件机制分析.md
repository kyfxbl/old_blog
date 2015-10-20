title: Hexo插件机制分析
date: 2015-08-19 15:43:55
categories: 源码阅读 
---
最近想开发一个rest API的框架，需要用到插件机制。正好前段时间在玩Hexo，觉得它那套机制还不错，于是参考了一下。本文总结一下它的实现思路
<!--more-->

# CLI启动

有种流行的做法是把cli和实现分离，比如grunt-cli和grunt。hexo也是采取这种方式，hexo-cli专门处理命令行，hexo才是具体的实现。可以像bash一样执行hexo-cli的命令

## 启动脚本

```
#!/usr/bin/env node

'use strict';

require('../lib')();
```

## 搜索路径，初始化Hexo

上面的脚本，实际上执行的是lib/index.js，核心代码如下。为了方便阅读，省略了与流程无关的代码：
```
findPkg(cwd, args).then(function(path){

    if(!path){
        return runCLICommand(args);
    }

    var modulePath = pathFn.join(path, 'node_modules', 'hexo');

    return fs.exists(modulePath).then(function(exist){
      
      if (!exist){
        return process.exit(1);
      }

      var Hexo = require(modulePath);
      hexo = new Hexo(path, args);
      log = hexo.log;

      return hexo.init().then(runHexoCommand);
    });
```

首先，Hexo大量使用了bluebird，包括上面代码中的fs也不是node核心模块的fs，而是经过promise化的API，所以习惯了callback风格的人可能会看得晕头转向，怎么全是各种return。本文就不介绍bluebird了，基本上就是前一个function执行完之后，会进入下一个then方法

findPkg具体的代码不展开了，目的是从cwd（当前目录，也就是执行hexo xxx命令的目录）递归向上查找package.json里是否包含Hexo属性，如果有的话，就把此目录作为Hexo的根目录

如果找不到根目录，就执行hexo-cli自带的3个基础命令（init, help, version）；如果找到了根目录，就require hexo module，然后实例化，调用init函数，最后执行具体的命令，如new，generate等

## cli用到的模块

hexo-cli思路很简单，麻雀虽小五脏俱全，读它的源代码也很有意思。比较有收获的是了解了几个库的用法
```
var minimist = require('minimist');
var abbrev = require('abbrev');
var tildify = require('tildify');
var chalk = require('chalk');
```

### minimist

minimist是命令行处理的组件，比如下面这个命令：
```
$ init blog --verbose --cwd /usr/local/
```

会处理成：
```
{ _: [ 'init', 'blog' ], verbose: true, cwd: '/usr/local/' }
```

以--开头的参数，会处理成key/value；其他的参数会以数组的形式保存。后续可以很容易地从中按顺序取出参数，或者判断某参数是否存在

### abbrev

abbrev也是个命令行处理组件：
```
var commands = ["generate", "init", "help"];
var shorthands = abbrev(commands);
```

会得到：
```
{ g: 'generate',
  ge: 'generate',
  gen: 'generate',
  gene: 'generate',
  gener: 'generate',
  genera: 'generate',
  generat: 'generate',
  generate: 'generate',
  h: 'help',
  he: 'help',
  hel: 'help',
  help: 'help',
  i: 'init',
  in: 'init',
  ini: 'init',
  init: 'init' }
```

可以方便用户输入命令

### tildify

tildify可以把用户的目录处理成~
```
var path = "/Users/apple/git_local/";
var short = tildify(path);// ~/git_local/
```

似乎没什么用

### chalk

chalk可以给stdout增加文字特效，比如改变文字颜色，增加下划线等

# Hexo执行

## 执行构造函数

从hexo-cli的这行代码开始，转入hexo执行：
```
var Hexo = require(modulePath);
hexo = new Hexo(path, args);
```

标准的javascript OO编程的惯例，设置了一大堆this.xxx = xxx，后续把这个实例作为参数传递，可以通过this.xxx取到实例变量

这里跟插件机制有关的地方，是以下的代码：
```
var extend = require('../extend');

this.extend = {
    console: new extend.Console(),
    deployer: new extend.Deployer(),
    filter: new extend.Filter(),
    generator: new extend.Generator(),
    helper: new extend.Helper(),
    migrator: new extend.Migrator(),
    processor: new extend.Processor(),
    renderer: new extend.Renderer(),
    tag: new extend.Tag()
};
```

这段代码的变量名取得不太好，容易误导。其实this.extend和extend是完全不同的东西

extend是一个局部变量，extend.Console，extend.Generator等，是构造函数：
```
function Console(){
  this.store = {};
  this.alias = {};
}

Console.prototype.register = function(name, desc, options, fn){
    // 注册插件的逻辑
};
```

而this.extend.console是用上述构造函数创建的实例，后续通过register函数来注册插件，它本身也是插件的容器，执行的时候从内部的store取出插件（通常是一个function）执行

## 执行init方法

接下来这行代码是核心，使hexo初始化，包括加载内部插件，外部插件，都是在init函数里完成的：
```
hexo.init()
```

init函数的核心代码如下，省略了与插件机制无关的部分：
```
// Load internal plugins
require('../plugins/console')(this);
require('../plugins/filter')(this);
require('../plugins/generator')(this);
require('../plugins/helper')(this);
require('../plugins/processor')(this);
require('../plugins/renderer')(this);
require('../plugins/tag')(this);

// Load external plugins & scripts
require('./load_plugins')(this);
```

上述的代码是注册插件，包括hexo自带的核心内部插件，和开发者扩展的外部插件。具体注册的流程下面再说

## 执行具体命令

hexo初始化之后，就开始执行具体命令，省略无关代码：
```
function runHexoCommand(){
    var cmd = args._.shift();
    return hexo.call(cmd, args);
}
```

注意hexo就是前面实例化的hexo对象，call函数不是Function的call，而是hexo定义的call函数：
```
Hexo.prototype.call = function (name, args, callback) {
    // 调用插件，执行具体命令
};
```

# 注册插件

插件分为内部插件和外部插件，内部插件是hexo自带的，外部插件是其他开发者的扩展

## 注册内部插件

hexo已经提供了核心的插件，在plugins目录里，会注册到对应的模块上。比如console的插件，会注册到hexo.extend.console这个对象上，内部用store存储。

另外，hexo的模块在extend目录里，针对不同的扩展点进行了分离。比如console模块的插件，是用hexo.extend.console注册的；generator模块的插件，是用hexo.extend.generator注册的。所以每个模块可以实现不同的注册逻辑，后续也有不同的执行逻辑

以下是console模块注册插件的核心代码；
```
module.exports = function(ctx){

    var console = ctx.extend.console;
    console.register('clean', 'Removed generated files and cache.', require('./clean'));
    
    // 以下类似，省略
};
```

其中require("./clean")得到了一个function，hexo的插件最终都是一个个function，在特定的时机被调用。这里的this指的是hexo的实例，后面会说
```
module.exports = cleanConsole;

function cleanConsole(args){
  
  return Promise.all([
    deleteDatabase(this),
    deletePublicDir(this)
  ]);
}
```

看下register代码的实现，省略了非核心的部分：
```
Console.prototype.register = function(name, desc, options, fn){

    var c = this.store[name.toLowerCase()] = fn;
    c.options = options;
    c.desc = desc;

    this.alias = abbrev(Object.keys(this.store));
};
```
主要就是把插件保存在store里，同时用abbrev设置了一些shorthands

## 注册外部插件

注册外部插件的代码在load_plugins.js里，外部插件指的是不在hexo核心里，通过npm install的扩展插件，命名规则是必须以hexo-开头。这样的module会被hexo框架识别为hexo外部插件，尝试加载
```
module.exports = function(ctx){

    if (!ctx.env.init || ctx.env.safe){
        return;
    } 

    return Promise.all([
        loadModules(ctx),
        loadScripts(ctx)
    ]);
};
```

然后是loadModules函数：
```
function loadModules(ctx){

  var packagePath = pathFn.join(ctx.base_dir, 'package.json');
  var pluginDir = ctx.plugin_dir;

  // Make sure package.json exists
  return fs.exists(packagePath).then(function(exist){
    if (!exist) return [];

    // Read package.json and find dependencies
    return fs.readFile(packagePath).then(function(content){
      var json = JSON.parse(content);
      var deps = json.dependencies || {};

      return Object.keys(deps);
    });
  }).filter(function(name){
    // Ignore plugins whose name is not started with "hexo-"
    if (name.substring(0, 5) !== 'hexo-') return false;

    // Make sure the plugin exists
    var path = pathFn.join(pluginDir, name);
    return fs.exists(path);
  }).map(function(name){
    var path = require.resolve(pathFn.join(pluginDir, name));

    // Load plugins
    return ctx.loadPlugin(path).then(function(){
      ctx.log.debug('Plugin loaded: %s', chalk.magenta(name));
    }, function(err){
      ctx.log.error({err: err}, 'Plugin load failed: %s', chalk.magenta(name));
    });
  });
}
```
从package.json的dependencies里找到所有hexo-开头的模块，然后在node-modules目录里找到对应的模块，将path作为参数调用loadPlugin函数

然后是loadPlugin函数，真正的注册逻辑都在这个函数里：
```
var Module = require('module');
var vm = require('vm');

Hexo.prototype.loadPlugin = function (path, callback) {

    var self = this;

    return fs.readFile(path).then(function (script) {
        // Based on: https://github.com/joyent/node/blob/v0.10.33/src/node.js#L516
        var module = new Module(path);
        module.filename = path;
        module.paths = Module._nodeModulePaths(path);

        function require(path) {
            return module.require(path);
        }

        require.resolve = function (request) {
            return Module._resolveFilename(request, module);
        };

        require.main = process.mainModule;
        require.extensions = Module._extensions;
        require.cache = Module._cache;

        script = '(function(exports, require, module, __filename, __dirname, hexo){' +
            script + '});';

        var fn = vm.runInThisContext(script, path);

        return fn(module.exports, require, module, path, pathFn.dirname(path), self);
    }).nodeify(callback);
};
```
上面这段代码有一个比较特别的地方，用到了node提供的Module和vm模块，这样通过hexo框架require的文件，都可以通过hexo变量访问到hexo的实例，从而能够访问hexo上的各种属性，如log，env等。这种做法很巧妙，令hexo的插件可以直接访问到hexo变量，又没有添加很多限制，值得学习

最后用我写的一个CSDN migrator为例，看下外部插件的写法：
```
hexo.extend.migrator.register('csdn', function(args){

    var username = args._.shift();
    
    // 迁移逻辑
});
```
migrator插件需要注册到hexo.extend.migrator模块下。Hexo框架支持的扩展点已经设计好了，就是extend目录下的那几个，分别都有注册和调用的逻辑。第三方插件应该注册在哪个模块下，需要查看官方文档说明，才能被正确地注册上，以及在正确的时机被调用

# 调用插件

所有插件的调用，都是从runHexoCommand开始的：
```
function runHexoCommand(){
    var cmd = args._.shift();
    return hexo.call(cmd, args);
}
```

进入Hexo的call方法，这里不是Function的call方法，省略无关代码后，核心代码只有2行：
```
Hexo.prototype.call = function (name, args, callback) {
    var c = self.extend.console.get(name);
    c.call(this, args);
};
```

根据用户输入的第一个命令，从hexo.extend.console中找到对应的console插件，并调用。以以下命令为例：
```
$ hexo migrate csdn xxxxxx
```

首先会调用hexo.extend.console上的migrate插件，而migrate插件只是迁移的入口，它内部又会调用具体的migrator插件来完成逻辑。由于调用的形式一般是plugin.call(hexo, args)，所以插件内部的this一般来说指的都是hexo实例：
```
var type = args._.shift();

var migrators = this.extend.migrator.list();// 所有migrator插件

// 没有找到，提示错误
if(!migrators[type]){

    var help = '';

    help += type.magenta + ' migrator plugin is not installed.\n\n';
    help += 'Installed migrator plugins:\n';
    help += '  ' + Object.keys(migrators).join(', ') + '\n\n';
    help += 'For more help, you can check the online docs: ' + chalk.underline('http://hexo.io/');

    console.log(help);
    return;
}

// function.call
return migrators[type].call(this, args);
```

再以clean为例，也是类似的：
```
$ hexo clean
```

```
function cleanConsole(args){

    return Promise.all([
        deleteDatabase(this),
        deletePublicDir(this)
    ]);
}
```
只是clean没有设计任何扩展点，所以内部就完成了所有清理逻辑，没有再调用其他的插件
