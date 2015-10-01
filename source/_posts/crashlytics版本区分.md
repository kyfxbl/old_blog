title: crashlytics版本区分
date: 2014-05-25 12:20
categories: iOS
---
crashlytics区分版本的方法
<!--more-->

用了一段时间Crashlytics，我有一个疑问。因为在测试阶段，我们会不断的修改代码-编译-测试，并且工程已经集成了Crashlytics，所以每次编译的时候，由于Crashlytics build script的存在，会有很多app binary和dSYM上传到他们的服务器上。那么当正式版发布的时候，我不知道Crashlytics能不能区分不同的dSYM从而正确地符号化crash log。并且，由于测试版和正式版的版本号一样，也不确定crash log在dashboard里是怎么分类的 

于是我就发邮件给Crashlytics的技术人员，他答复说： 

1、Crashlytics可以正确关联上不同的app binary和dSYM，关联的依据不是工程的版本号，而是uuid。也就是说，每次生成的crash log，app binary，dSYM都有一个UUID，从而可以关联起来，从而保证符号化正确 

2、关于dashboard的问题，他建议给测试版一个单独的版本号，比如1.6.0-test，这样在dashboard里，就会分类到不同的version filter下，不会和正式版的crash report混在一起