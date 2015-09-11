title: 用ffi调用.dylib库
date: 2015-02-11 00:26
categories: javascript 
---
有一个特性需要调用第三方库libsync，在node.js里可以用ffi来实现
<!--more-->

首先稍微包装了一下，ffi也是支持异步调用的，但是API比较麻烦，包装以后调用起来会比较容易

```
var ffi = require("ffi");

var lib = ffi.Library('./libsync', {
    'file_chunk': ['int', ['string', 'string', 'int']],
    'file_delta': ['int', ['string', 'string', 'string', 'int']],
    'file_sync': ['int', ['string', 'string']]
});

exports.file_chunk = file_chunk;
exports.file_delta = file_delta;
exports.file_sync = file_sync;

// callback(err, result)
function file_chunk(src, chunk, algo, callback){
    lib.file_chunk.async(src, chunk, algo, callback);
}

function file_delta(src, chunk, delta, algo, callback){
    lib.file_delta.async(src, chunk, delta, algo, callback);
}

function file_sync(src, delta, callback){
    lib.file_sync.async(src, delta, callback);
}
```

libsync就是依赖的动态链接库，在linux下是.so文件，而在Mac下是.dylib文件，ffi会根据当前的平台，自动查找合适的后缀：

```
/**
 * The extension to use on libraries.
 * i.e.  libm  ->  libm.so   on linux
 */
var EXT = Library.EXT = {
    'linux':  '.so'
  , 'linux2': '.so'
  , 'sunos':  '.so'
  , 'solaris':'.so'
  , 'freebsd':'.so'
  , 'openbsd':'.so'
  , 'darwin': '.dylib'
  , 'mac':    '.dylib'
  , 'win32':  '.dll'
}[process.platform]
```

所以接下来就是需要把源代码.c，.h编译成.so和.dylib库（开发需要.dylib，生产环境需要.so）

mac下编译dylib文件的命令也很简单：

```
$ gcc -dynamiclib -o c.dylib a.c b.c
```

客户端实际调用的代码：
```
libsync.file_chunk(localPath, chunkPath, 0, function (err, flag) {
    // logic
});
```