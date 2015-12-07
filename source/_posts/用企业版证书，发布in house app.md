title: 用企业版证书，发布in house app
date: 2015-02-07 23:31
categories: iOS
---
苹果企业版证书，虽然不能上app store，但是可以实现在网页上直接点击下载，而且不需要知道设备的UDID，合理使用的话还是很方便的。昨天用这种方式发布成功了，本文总结一下步骤
<!--more-->

首先，在xcode中export的时候，可以看到有3个选项：

![](http://img.blog.csdn.net/20150207231411114?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

第一个是提交到app store；第二个是打出IPA，然后就可以通过iTunes安装；第三个就是打出企业版的IPA，可以直接通过网页安装。这种发布方式也叫做in house

以下是发布in house app的步骤

# 申请苹果企业版开发账号

网址是：[enterprise program](https://developer.apple.com/programs/ios/enterprise/)，一年$299

申请通过之后，还要在后台配置AppId，Certificates，Profiles等

# 修改xcode配置

主要需要配置2个地方，第一个是General里的team，配置成enterprise program所在的team，这步一般都不会忘记

另一个是比较容易遗漏的地方，<span style="color:#ff0000">需要在Build Settings里配置Code Signing</span>

![](http://img.blog.csdn.net/20150207232055155?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

配置OK以后，就可以打出in house方式的IPA包了

# 设置下载链接

有了ipa，就可以放到网页上了，同时还需要一个plist文件，<span style="color:#ff0000">必须跟ipa同名</span>

下载链接很简单：

```
<a href='itms-services://?action=download-manifest&url=https://www.xxx.com/ipa/ipa_name.plist'>点击安装</a>
```

plist从xcode6开始不会自动生成了，需要手工编辑。内容类似：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>http://www.xxx.com/ipa/ipa_name.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>http://www.xxx.com/ipa/icon_120.png</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>http://www.xxx.com/ipa/icon_120.png</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>app bundle id</string>
				<key>bundle-version</key>
				<string>1.0.0</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>your app name</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```
然后，<span style="color:#ff0000">这个plist必须通过https协议访问</span>。如果是http，会有错误提示，“无法安装应用，因为XXX的证书不可用”。然后该网站的服务器证书，<span style="color:#ff0000">也不能是自签名证书</span>，必须是CA签发的证书，否则也不能成功安装。所以还需要想办法弄一个证书，可以去买一个，也可以去申请一个免费的