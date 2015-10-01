title: Base64
date: 2013-09-24 11:25
categories: 其它
---
Base64编码介绍
<!--more-->

Base64是一种编码的算法，最初是为了解决电子邮件传输的问题。勉强可以认为它是一种加密算法，但是安全系数极低。因为不仅算法是公开的，连密钥也是公开的 

算法是这样的： 

1、对于一个给定的字符或字符串，先按照某种编码字符集（如UTF-8、GBK等）编码，得到二进制码 

2、对二进制码做分组变化，从8个bit一组，改成6个bit一组，最后一组不足6个的，在低位补0 

3、换算成十进制，找到Base64字符表中的字符（范围从0-63，一共64个字符，这也是Base64名字的由来） 

4、若最后一组字符不足4个，则在末尾用"="补足4个字符 

比如对于字符"A" 

1、用UTF-8编码，得到01000001 

2、分组变化，得到010000 010000 

3、十进制是16 16，查Base64编码表，得到Q Q 

4、由于字符不足4个，在末尾补上2个"="，最后得到"QQ==" 

字符"A"用Base64编码以后，得到的就是"QQ=="

如果原来字节数是3的整数倍，那么Base64编码之后就会是4的整数倍，就不需要在末尾补"="了。但是一般没那么巧，所以Base64编码的一个明显的特点，就是末尾的字符经常是"="或者"==" 

再来算下长度，"A"用UTF-8编码后，只占1个字节；Base64编码之后，则占4个字节，是原来的4倍。不过这是比较极端的情况，如果对"AAA"编码，则原来是3个字节，编码后4个字节，是原来的4/3倍。无论何时，Base64编码后的字节数，都是4的整数倍

一般来说，经过Base64编码之后，都会比原来略大，这主要是因为分组变化，从8bit一组拆成了6bit一组的原因。至于最后补位的"="，一般可以忽略不计 

注意，Base64是对字节进行编码，而不是直接对字符编码。所以同样的字符，首先用不同的编码字符集编码，最后得到的Base64编码也是不同的 

Base64编码表的最后2个字符是"+"和"/"，在URL里都是不能用的，所以后来又衍生出了URL Base64，把最后2个字符改成了"-"和"_" 

JDK里也有Base64的实现，不过不推荐使用；一般用commons codec比较好 

最后贴一段示例代码：

```
public static void main(String[] args) throws IOException {

		String encoding = "UTF-8";

		String name = "A";

		byte[] originBytes = name.getBytes(encoding);

		byte[] encodedBytes = Base64.encodeBase64(originBytes, false);

		System.out.println(originBytes.length);

		System.out.println(encodedBytes.length);

		System.out.println(new String(encodedBytes, encoding));

	}
```

更详细的内容，可以看一下《JAVA加密与解密的艺术》第5章，讲得更全面，只是稍微有些错误