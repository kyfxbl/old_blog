title: nginx与iOS实现https双向认证
date: 2014-05-13 15:35
categories: iOS 
---
iOS下实现与nginx服务器的https双向认证
<!--more-->

# 服务端配置

nginx关键配置如下：

```
listen       443;
server_name  localhost;
ssl on;
ssl_certificate      /usr/local/opt/nginx/certificates/server.cer;
ssl_certificate_key  /usr/local/opt/nginx/certificates/server.key.pem;
ssl_client_certificate /usr/local/opt/nginx/certificates/ca.cer;
ssl_verify_client    on;
```

比较容易弄错的是
```
ssl_client_certificate
```
这是签发客户端证书的根证书的路径。因为可以签发很多客户端证书，只要是由该根证书签发的，服务端都视为认证通过

配置完成以后，一般需要把80端口的http请求跳转到443端口，否则用户可以通过80端口以http方式访问，就失去了安全保护的意义

# 客户端代码

在网上找了很多ios client certificate的帖子，要么是代码不全，要么是很老的帖子，还是用的NSURLConnection，delegate method都deprecated了，最后找了一个代码片段，改了一下

```
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];

NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];

NSURL *url = [NSURL URLWithString:@"https://localhost/svc/portal/setting"];

NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url];
[request setCachePolicy:NSURLRequestReloadIgnoringLocalCacheData];
[request setHTTPShouldHandleCookies:NO];
[request setTimeoutInterval:30];
[request setHTTPMethod:@"GET"];

NSURLSessionDataTask *task = [session dataTaskWithURL:url completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
    NSString *message = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"%@", message);
}];

[task resume];
```

上面的代码片段，是用NSURLSessionDataTask发起https请求。在ssl握手阶段，会调用2次下面的delegate method

```
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *credential))completionHandler
{
    NSString *method = challenge.protectionSpace.authenticationMethod;
    NSLog(@"%@", method);

    if([method isEqualToString:NSURLAuthenticationMethodServerTrust]){

        NSString *host = challenge.protectionSpace.host;
        NSLog(@"%@", host);

        NSURLCredential *credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
        completionHandler(NSURLSessionAuthChallengeUseCredential, credential);
        return;
    }

    NSString *thePath = [[NSBundle mainBundle] pathForResource:@"client" ofType:@"p12"];
    NSData *PKCS12Data = [[NSData alloc] initWithContentsOfFile:thePath];
    CFDataRef inPKCS12Data = (CFDataRef)CFBridgingRetain(PKCS12Data);
    SecIdentityRef identity;

    // 读取p12证书中的内容
    OSStatus result = [self extractP12Data:inPKCS12Data toIdentity:&identity];
    if(result != errSecSuccess){
        completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, nil);
        return;
    }

    SecCertificateRef certificate = NULL;
    SecIdentityCopyCertificate (identity, &certificate);

    const void *certs[] = {certificate};
    CFArrayRef certArray = CFArrayCreate(kCFAllocatorDefault, certs, 1, NULL);

    NSURLCredential *credential = [NSURLCredential credentialWithIdentity:identity certificates:(NSArray*)CFBridgingRelease(certArray) persistence:NSURLCredentialPersistencePermanent];

    completionHandler(NSURLSessionAuthChallengeUseCredential, credential);
}

-(OSStatus) extractP12Data:(CFDataRef)inP12Data toIdentity:(SecIdentityRef*)identity {

    OSStatus securityError = errSecSuccess;

    CFStringRef password = CFSTR("the_password");
    const void *keys[] = { kSecImportExportPassphrase };
    const void *values[] = { password };

    CFDictionaryRef options = CFDictionaryCreate(NULL, keys, values, 1, NULL, NULL);

    CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
    securityError = SecPKCS12Import(inP12Data, options, &items);

    if (securityError == 0) {
        CFDictionaryRef ident = CFArrayGetValueAtIndex(items,0);
        const void *tempIdentity = NULL;
        tempIdentity = CFDictionaryGetValue(ident, kSecImportItemIdentity);
        *identity = (SecIdentityRef)tempIdentity;
    }

    if (options) {
        CFRelease(options);
    }

    return securityError;
}
```

上面的代码片段，即拷即用，希望能有所帮助。这个方法会被调用2次，第一次是ios app验证server的阶段，第二次是server验证ios app的阶段，即client certificate。关键是每个阶段的校验完成以后，要调用completionHandler方法。网上的大部分帖子都是

```
[[challenge sender] useCredential:credential forAuthenticationChallenge:challenge];
```
这行代码是无效的