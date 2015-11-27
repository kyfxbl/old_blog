title: nohup及重定向
date: 2014-03-21 20:33
categories: linux
---
nohup的用法
<!--more-->

# 保持进程

```
node xxx.js
```

这行命令当按下control + c，进程就会退出。如果需要进程在后台保持运行，需要

```
node xxx.js &
```

但是如果是通过ssh远程登录到linux主机上，当关闭窗口，进程还是会退出，所以需要用nohup命令

```
nohup node xxx.js &
```
这样即使关闭了窗口，进程还是会继续执行

# 输出重定向

默认stdout和stderr都会重定向到nohup.out

```
nohup node test.js &
```

```
[1] 31767
nohup: 忽略输入并把输出追加到"nohup.out"
```

可以重定向到不同的位置：

```
nohup node test.js >> app.log &
```

```
[1] 31803
nohup: 忽略输入重定向错误到标准输出端
```

用上面的命令，stdout和stderr都重定向到app.log文件，如果希望分离stdout和stderr，可以用下面的命令

```
nohup node test.js 1>>app.log 2>>error.log &
```

```
[1] 31851
```

# 跟node.js console的关系

```
console.log()
```
是输出到stdout，而

```
console.trace()
```
则是输出到stderr