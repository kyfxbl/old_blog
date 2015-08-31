title: https交叉证书
date: 2015-03-22 16:18
categories: 其它
---
为了发布苹果企业版ipa，我们在godaddy上申请了SSL证书。部署之后发现，在PC/Mac平台的各浏览器上都可以正常通过https协议访问，但是在iPad的Safari上就会报错。还需要部署交叉证书，才能支持移动端浏览器
<!--more-->

在这个地址：[SSL checker](https://www.sslshopper.com/ssl-checker.html#hostname=xxx.com)，校验提示以下错误信息：

The certificate is not trusted in all web browsers. You may need to install an Intermediate/chain certificate to link it to a trusted root certificate.

看样子是少部署了交叉证书的原因，检查SSL包，里面有2个crt，用文本编辑器可以打开，都是由

```
-----BEGIN CERTIFICATE-----

xxxxxxxxxx

-----END CERTIFICATE-----
```

组成的文本。原来我只安装了基本证书，而交叉证书在另外一个crt里。最后我把交叉证书也拷贝到第一个证书文件里，顺序是先基本证书，再交叉证书。重启nginx后，iPad上的Safari也可以正常通过https访问了