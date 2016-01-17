title: tomcat配置https
date: 2016-01-17 18:02:54
categories: java
---
java平台的keystore是对https证书的一种包装，配置方式和http服务器的证书配置略有不同
<!--more-->

最近和一个第三方系统对接，需要安全认证。安全认证有2种方式，一种是在应用层实现，比如通过ws-security或者在报文头增加一些字段等；另外一种是借助https，对应用层透明。本次对接采用的是https的方案

根据部署方式的不同，具体的实现也有区别。一般在tomcat前面会有一个http服务器如nginx来接收https请求并转发，那么需要在nginx上配置证书。但是这次对方是直接用tomcat响应web请求，所以需要在tomcat里配置

一开始我尝试用openssl做了自签名的证书，然后导入keystore，结果并不成功，用chrome访问始终提示以下错误：
```
ERR_SSL_VERSION_OR_CIPHER_MISMATCH
```

搜索了一番也没有找到解决办法。最后只好改成用jdk提供的keytool工具直接制作keystore，然后从keystore导出证书，以下是具体的命令：

# 生成keystore

```
keytool -genkey -alias aisino -keyalg RSA -keysize 2048 -validity 3650 -keystore aisino.keystore

keytool -list -v -keystore aisino.keystore
```

第一个命令是生成keystore，第二个命令是查看生成的结果

然后在tomcat中做如下配置：
```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="200" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" keystoreFile="aisino.keystore" keystorePass="xxxxxx" />
```

不过这是Http11NioProtocol风格的配置，如果用的是Http11AprProtocol，则应该使用openssl风格的配置，见其他文档

经过上述配置之后，就可以在浏览器中通过https协议访问应用服务器了

# 导出为证书

接下来客户端可能需要对应的证书，比如需要在发送https请求时用证书加密，因此需要将刚才生成的keystore信息导出为标准的证书。通过以下命令：
```
keytool -export -alias aisino -keystore aisino.keystore -file ./aisino.cer

keytool -printcert -file aisino.cer
```

第一个命令是导出证书，第二个命令是查看证书的内容。接下来只要把导出的证书给到客户端就可以了