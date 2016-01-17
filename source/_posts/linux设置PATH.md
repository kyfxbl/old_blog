title: linux设置PATH
date: 2013-10-03 15:08
categories: linux 
---
linux设置PATH的命令
<!--more-->

linux shell执行脚本和windows不一样。windows只要在当前目录里有可执行文件，比如.bat，就可以直接执行xxx.bat。但是在linux下则不行，shell会去查找PATH变量，如果当前目录路径不在PATH中，就必须用完整路径./xxx。所以就比较麻烦，每次都要输入完整的路径，解决的办法是修改PATH变量

一次性的办法
```
export PATH=xxx:$PATH
```

xxx就是要新增的路径，:是linux下的路径分隔符（windows下是;）

但是这样每次重启了以后，都需要重新设置，还是很麻烦，永久性的办法是编辑/etc/porfile文件

```
vi /etc/profile

export PATH=$JAVA_HOME/bin:$MVN_HOME/bin:$NODE_HOME/bin:$MONGO_HOME/bin:$PATH

source /etc/profile
```

