title: iOS和node上传下载文件
date: 2014-01-21 23:51
categories: iOS
---
上传和下载需要server和client互相配合。同样的客户端代码，可能在servlet里能成功，换成node就不行；反过来也是一样。因为不同的服务端，对http请求的处理可能不同。本文介绍的是服务端使用node，客户端使用NSURLSession的情况
<!--more-->

# 服务端代码

我还没见过哪种实现方式，比node + express更简单的：

```
var express = require("express");

var app = express();

app.use(express.bodyParser({
    uploadDir: __dirname + '/../var/uploads',
    keepExtensions: true,
    limit: 100 * 1024 * 1024,
    defer: true
})).use('/svc/public', express.static(__dirname + '/../public'));

app.post('/svc/upload', function (req, res) {

    req.form.on('progress', function (bytesReceived, bytesExpected) {

    });

    req.form.on('end', function () {
        var tmp_path = req.files.file.path;
        var name = req.files.file.name;

        console.log("tmp_path: "+ tmp_path);
        console.log("name: "+name);

        res.end("success");
    });
});

app.listen(3000);
```

上面就是服务端全部的代码。defer属性设置为true，这样下面的2个生命周期回调才能生效。不过这个服务，直接用CocoaRestClient发POST请求好像不行，似乎需要在http header里加上Content-Type才可以

# 上传的客户端代码

View省略，只介绍关键的ViewController代码

```
@interface YLSUploadViewController : UIViewController<NSURLSessionTaskDelegate>

-(void) doUpload;

@end
```

主要是实现NSURLSessionTaskDelegate协议，因为我们需要其中的生命周期方法来实现进度条

下面是初始化的代码：

```
{
    NSString *boundary;
    NSString *fileParam;
    NSURL *uploadURL;
}

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];

    if (self) {

        boundary = @"----------V2ymHFg03ehbqgZCaKO6jy";
        fileParam = @"file";
        uploadURL = [NSURL URLWithString:@"http://192.168.1.103:3000/svc/upload"];        
    }
    return self;
}
```

这里初始化了几个实例变量，下面是最关键的方法：

```
-(void) doUpload
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){

        NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];

        NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];

        NSData *body = [self prepareDataForUpload];

        NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:uploadURL];
        [request setHTTPMethod:@"POST"];

        // 以下2行是关键，NSURLSessionUploadTask不会自动添加Content-Type头
        NSString *contentType = [NSString stringWithFormat:@"multipart/form-data; boundary=%@", boundary];
        [request setValue:contentType forHTTPHeaderField: @"Content-Type"];

        NSURLSessionUploadTask *uploadTask = [session uploadTaskWithRequest:request fromData:body completionHandler:^(NSData *data, NSURLResponse *response, NSError *error){

            NSString *message = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
            NSLog(@"message: %@", message);

            [session invalidateAndCancel];
        }];

        [uploadTask resume];
    });
}

-(NSData*) prepareDataForUpload
{
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    NSString *uploadFilePath = [documentsDirectory stringByAppendingPathComponent:@"QQ.dmg"];

    NSString *fileName = [uploadFilePath lastPathComponent];

    NSMutableData *body = [NSMutableData data];

    NSData *dataOfFile = [[NSData alloc] initWithContentsOfFile:uploadFilePath];

    if (dataOfFile) {
        [body appendData:[[NSString stringWithFormat:@"--%@\r\n", boundary] dataUsingEncoding:NSUTF8StringEncoding]];
        [body appendData:[[NSString stringWithFormat:@"Content-Disposition: form-data; name=\"%@\"; filename=\"%@\"\r\n", fileParam, fileName] dataUsingEncoding:NSUTF8StringEncoding]];
        [body appendData:[@"Content-Type: application/zip\r\n\r\n" dataUsingEncoding:NSUTF8StringEncoding]];
        [body appendData:dataOfFile];
        [body appendData:[[NSString stringWithFormat:@"\r\n"] dataUsingEncoding:NSUTF8StringEncoding]];
    }

    [body appendData:[[NSString stringWithFormat:@"--%@--\r\n", boundary] dataUsingEncoding:NSUTF8StringEncoding]];

    return body;
}
```

关键是怎么拿到NSURLSessionUploadTask，虽然NSURLSession提供了uploadTaskWithRequest:fromFile:方法，不过经过实践，发现跑不通。NSURLSession似乎不会自动加上Content-Type头，也不会自动在Data中加入boundary，结果就是server端报错：

TypeError: Cannot call method 'on' of undefined     
at /Users/apple/WebstormProjects/uploadAndDownloadServer/lib/main.js:15:14     
at callbacks (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/lib/router/index.js:161:37)     
at param (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/lib/router/index.js:135:11)     
at pass (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/lib/router/index.js:142:5)     
at Router._dispatch (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/lib/router/index.js:170:5)     
at Object.router (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/lib/router/index.js:33:10)     
at next (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/node_modules/connect/lib/proto.js:190:15)     
at next (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/node_modules/connect/lib/proto.js:165:78)     
at multipart (/Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/node_modules/connect/lib/middleware/multipart.js:60:27)     
at /Users/apple/WebstormProjects/uploadAndDownloadServer/node_modules/express/node_modules/connect/lib/middleware/bodyParser.js:57:9

所以我最后的做法是，自己从File中读出Data，并拼上所需的控制符，这都是在prepareDataForUpload()方法里实现的

最后是Delegate method方法，我只需要一个：

```
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
```
这个就很简单了，不多介绍了，有totalBytesSent和totalBytesExpectedSend这2个变量，无论是要做文本提示，还是进度条，都是很容易实现的

不过上面的示例代码，为了方便把自己设置为delegate了。实际项目里，应该把业务逻辑的类设置为upload组件的delegate。因为上传之后应该做什么，应该是在业务组件里控制才对

# 下载的客户端代码

相比上传的代码，下载简单很多：

```
-(void) doDownload
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){

        NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];

        NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];

        NSURL *url = [NSURL URLWithString:@"http://192.168.1.103:3000/svc/public/bigfile.dmg"];

        NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url];
        [request setHTTPMethod:@"GET"];

        NSURLSessionDownloadTask *downloadTask = [session downloadTaskWithRequest:request];// 未设置block

        [downloadTask resume];
    });
}
```

代码只有一点需要注意，即调用的是downloadTaskWithRequest:方法，而不是另一个带block callback的API。因为发现，如果设置了completionHandler，则delegate method不会被调用，但是和上传一样，我们需要delegate method来实现下载进度条

```
@interface YLSDownloadViewController : UIViewController<NSURLSessionDownloadDelegate>
```

其中这个方法可以实现进度条：

```
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
```

下载后的文件，是放在tmp目录下，如果不处理的话，马上就会被移除，所以需要在另一个delegate method里拷贝到最终路径：

```
- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];
    NSString *distFilePath = [documentsDirectory stringByAppendingPathComponent:@"success.dmg"];

    NSString* tempFilePath = [location path];

    NSFileManager *fileManager = [NSFileManager defaultManager];
    if([fileManager fileExistsAtPath:tempFilePath]){
        [fileManager copyItemAtPath:tempFilePath toPath:distFilePath error:nil];
    }

    [session invalidateAndCancel];
}
```