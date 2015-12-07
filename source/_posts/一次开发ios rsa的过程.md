title: 一次开发ios rsa的过程
date: 2013-12-10 15:17
categories: iOS
---
昨天需要把android版的用户注册功能，移植到ios版上。android版会将用户填写的手机号和密码，用RSA加密后发到server
<!--more-->

# 尝试直接使用modulus和exponent加密

android版没有使用证书，是直接用modulus和exponent就加密了

```
RSAPublicKeySpec publicKeySpec = new RSAPublicKeySpec(new BigInteger(modulus), new BigInteger(publicExponent));
return (RSAPublicKey) keyFactory.generatePublic(publicKeySpec);
```

```
byte[] originData = plaintext.getBytes();		
Cipher ci = Cipher.getInstance("RSA");
ci.init(Cipher.ENCRYPT_MODE, publicKey);
byte[] encryptedData = ci.doFinal(originData);
return new String(Hex.encodeHex(encryptedData));
```

一开始因为不想修改server端decrypt的代码，所以就打算在ios也用同样的方法做。但是经过多方尝试，难度极大，找了无数帖子都不行。在当前的环境ios7 + xcode5下，建议大家就死了这条心吧。还是不死心的网友，可以看看下面这几个帖子，再尝试一下，反正我已经放弃

[ios rsa link1](http://stackoverflow.com/questions/19459359/ios-rsa-encrypt-using-public-key-with-modulus-and-exponent)

[ios rsa link2](http://stackoverflow.com/questions/6665832/iphone-rsa-algorithm-with-modulus-and-exponent)

[ios rsa link3](http://stackoverflow.com/questions/7255991/rsa-encryption-public-key)

[ios rsa link4](http://stackoverflow.com/questions/14607635/encrypting-a-string-with-rsa-using-only-the-modulus-and-exponent-in-ios)

# 使用证书（公钥）加密

这个就非常简单了，网上的例子很多，把我们自己的例子也贴出来：

```
+(NSString*) encryptWithRSA:(NSString*)plainText
{
    SecKeyRef publicKey = [self getPublicKey];
    size_t cipherBufferSize = SecKeyGetBlockSize(publicKey);
    uint8_t *cipherBuffer = malloc(cipherBufferSize);
    uint8_t *nonce = (uint8_t *)[plainText UTF8String];
    SecKeyEncrypt(publicKey,
                  kSecPaddingOAEP,
                  nonce,
                  strlen((char*)nonce),
                  &cipherBuffer[0],
                  &cipherBufferSize);
    NSData *encryptedData = [NSData dataWithBytes:cipherBuffer length:cipherBufferSize];
    return [encryptedData base64EncodedString];
}

+(SecKeyRef) getPublicKey
{
    NSString *certPath = [[NSBundle mainBundle] pathForResource:@"public_key" ofType:@"der"];
    NSData *certificateData = [[NSData alloc] initWithContentsOfFile:certPath];

    SecCertificateRef certificate = SecCertificateCreateWithData(kCFAllocatorDefault, (__bridge CFDataRef)certificateData);
    SecPolicyRef policy = SecPolicyCreateBasicX509();
    SecTrustRef trust;
    OSStatus status = SecTrustCreateWithCertificates(certificate,policy,&trust);
    SecTrustResultType trustResult;
    if (status == noErr) {
        status = SecTrustEvaluate(trust, &trustResult);
    }
    return SecTrustCopyPublicKey(trust);
}
```
是一个类方法，传进来明文，返回值是加密好的NSString，以base64编码。如果我们的server是用java写的，那这个方法应该就没问题了。但是我们的server是用node写的，并没有原生用RSA解密的方法，从网上找了一个据说最好的模块ursa，最后也没有解决问题，报各种奇怪的错误，无奈之下也放弃了。下面附上ursa的链接：

[URSA](https://github.com/Obvious/ursa)

# 用base64编码了事

总之，用modulus + exponent在IOS上找不到办法，用证书在node上没调通。最后也不想在这个细枝末节上再继续纠结了，决定也不加密了，就客户端用base64编码，server解码得了。一下就弄好了，下面也把代码贴上来

```
+(NSString*) encodeWithBase64:(NSString*)plainText
{
    NSData *data = [plainText dataUsingEncoding:NSUTF8StringEncoding];
    return [data base64EncodedString];
}
```
```
decodedInfo = new Buffer(info, 'base64').toString();
```

# node.js处理base64

普通字符串转base64：

```
var a = new Buffer('key1=value1&key2=value2').toString('base64');
```

base64转普通字符串：

```
new Buffer(a, 'base64').toString();
```