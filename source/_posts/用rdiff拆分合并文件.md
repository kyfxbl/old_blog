title: 用rdiff拆分合并文件
date: 2015-03-22 16:41
categories: 方案 
---
我们的一个方案是基于文件做多端数据同步，如何正确、高效地同步文件是基础的保障，本文介绍一下我们的方案
<!--more-->

# librsync

一开始我们使用了国产的libsync库：[libsync](http://blog.csdn.net/liuaigui/article/details/5949921)

基本的流程是：有文件A和B，现在想把文件A“变成”文件B，先对文件A做chunk；然后用chunk和文件B对比，得到delta；最后用文件A和delta做sync操作，A就变成了B的复制。但是实际测试之后发现，在某些场景下，合并得到的文件和原始文件的MD5不一致，说明在过程中，文件已经损坏了

看源代码，没有找到原因。于是直接改用另一个更有名的库，librsync，地址在：[librsync](https://github.com/librsync/librsync)。编译之后会得到2个东西，一个是可执行命令行rdiff，另一个是静态链接库librsync.a，如果需要动态链接库比如.so或.dylib，需要自己改一下Makefile

经过测试，刚才合并出错的2个文件，用rdiff命令执行signature -> delta -> patch之后，MD5完全一致，说明新的库本身是ok的，没有遇到老库的bug。那么接下来还需要做2件事：

1、将librsync库集成到iOS APP里

2、将librsync库集成到server代码里

# 集成进ios平台

我在MBP上没有办法正确执行make，因为始终少popt.h这个头文件，找了很久也没有找到mac平台下能用的popt-devel包。倒是在CentOS上直接yum install popt-devel就搞定了

所以我的librsync.a链接库是在linux下编译出来的，直接放到iOS APP里也用不了。反正有源代码，干脆把相关的.c和.h放到工程里了，随APP一起编译，稍微改了改就编译通过了

调用的代码也很简单，因为在客户端只需要delta，我就只包装了一个函数：
```
-(int) deltaWithSignature:(NSString*)signaturePath newFilePath:(NSString*)newFilePath deltaPath:(NSString*)deltaPath
{
    rs_signature_t  *sumset;
    rs_stats_t      stats;

    FILE *sig_file = rs_file_open([signaturePath UTF8String], [@"rb" UTF8String]);
    FILE *new_file = rs_file_open([newFilePath UTF8String], [@"rb" UTF8String]);
    FILE *delta_file = rs_file_open([deltaPath UTF8String], [@"wb" UTF8String]);

    rs_result result = rs_loadsig_file(sig_file, &sumset, &stats);// 读取chunk文件

    if (result != RS_DONE){
        return result;
    }

    if ((result = rs_build_hash_table(sumset)) != RS_DONE){
        return result;
    }

    result = rs_delta_file(sumset, new_file, delta_file, &stats);

    rs_free_sumset(sumset);
    rs_file_close(delta_file);
    rs_file_close(new_file);
    rs_file_close(sig_file);

    return result;
}
```

# 集成进server端代码

官方自带的Makefile只能打出静态链接库，如果要打出所需的.so文件，还需要自己改一下Makefile。而且我们这个服务是用node写的，就算得到了.so，还是需要用FFI再重新包装一下，才能在node里面调用，同样很费事

所以最后我是在环境里编译好了rdiff命令行，然后代码里直接调用，省去了封装FFI的麻烦。由于是异步调用，性能也没有太大的问题，不过不太好的就是调用过程中的精细控制不好实现，另外集群部署的时候也会麻烦一点，要保证每台机器都装好rdiff
```
var cmd = "rdiff patch " + localRdbPath + " " + uploadPath + " " + syncPath;
console.log(cmd);

exec(cmd, {}, function (err, stdout, stderr) {
    // logic
});
```