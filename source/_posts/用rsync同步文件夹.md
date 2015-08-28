title: 用rsync同步文件夹
date: 2015-08-21 15:18:25
categories: linux
---
用rsync命令在不同机器间同步文件夹，及hexo-deployer-rsync一个BUG的规避方法
<!--more-->

# 命令格式

例如，要把本机public目录与服务器上的/home/blog目录同步，用以下命令：
```
$ rsync --delete -avz -e ssh public/ root@121.xx.xx.212:/home/blog
```

如果服务器的ssh端口不是默认的22，则需要给ssh指定端口号，这种情况不常见：
```
$ rsync --delete -avz -e 'ssh -p 22' public/ root@121.xx.xx.212:/home/blog
```

# hexo-deployer-rsync的BUG

如果没有在_config.yml中指定port参数，则无法正确同步，实际上最后执行的命令是：
```
$ rsync --delete -avz -e public/ root@121.xx.xx.212:/home/blog
```

可以发现，指定了-e，但是却少了ssh。出错的代码如下：
```
var params = [
    '-az',
    'public/',
    '-e',
    args.user + '@' + args.host + ':' + args.root
];

if (args.port && args.port > 0 && args.port < 65536){
    params.splice(params.length - 1, 0, 'ssh -p ' + args.port);
}
```

截止到本文，已经有若干人都针对此issue提了pr，但是作者还没有merge。
[default port issue](https://github.com/hexojs/hexo-deployer-rsync/issues/6)

所以目前避免此BUG的方法，是在_config.yml中设置port为22