title: servlet filter
date: 2013-09-24 11:24
categories: java
---
struts2的实现核心是filter，本文简要描述filter相关的一些问题
<!--more-->
 
一般应用中如果用到了struts2，则会配置一系列action，不会配置servlet和servlet-mapping，但是struts2依然会匹配到正确的Action 

奥秘在于struts2用一个filter过滤了所有匹配的请求（一般是*.action） 在这个filter中，struts2根据请求的URL，截取出actionName，在自己的配置文件中进行匹配，然后转发到合适的Action类进行处理 

关键在于，在这个filter里，没有依照常规，调用filterChain.doFilter()方法。也就是说，在struts2 filter后面的filter，以及servlet容器默认的filter，都没有执行的机会 下面贴一个简单的例子：
```
@Override
public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {
	System.out.println("hehe");
}
```

如果配置了这个filter，然后访问任何url，都只会在控制台打出hehe，然后显示空白页面 
![](http://dl2.iteye.com/upload/attachment/0086/3141/bcc0fa5a-2b8b-3113-95f5-6403a65ceb8d.png)
![](http://dl2.iteye.com/upload/attachment/0086/3143/ad941eaa-41f7-3ee5-96d9-600b05927df2.png)

然后在这段代码中，允许后面的filter执行
```
@Override
public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {
	System.out.println("hehe");
	arg2.doFilter(arg0, arg1);
}
```

这时候控制台还是会打出hehe，但是同时tomcat也会返回404页面 
![](http://dl2.iteye.com/upload/attachment/0086/3145/17b31d49-ac17-3e63-b2b6-65846ab3523c.png)

因为tomcat自己是实现了一个Filter实现类，根据请求的URL，去匹配合适的servlet。在tomcat初始化的时候，会读取web.xml中的filter配置，或者扫描@WebFilter的类，得到所有的自定义Filter，再加上自己的默认filter，组成filter chain。而在struts2 filter的实现中，则没有给后面的filter执行的机会，也因此避免了servlet容器去扫描servlet，避免了扫描不到的错误 

这是struts2利用filter实现的一个巧妙之处，但是也是带来了一个副作用。即如果同时使用了struts2框架，以及其他用到filter的组件（比如CAS SSO），必须将struts2的配置放在最后，否则所有的后续filter，都没有执行的机会 

另外，我们可以发现，filter是所有http请求的第一关，如果在这里发生阻塞，则会瞬间秒杀整个应用
```
@Override
public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {

	System.out.println("hehe");

	try {
		Thread.sleep(10000);
	} catch (InterruptedException e) {
		e.printStackTrace();
	}
}
```
然后用浏览器同时开2个TAB页发起HTTP请求，控制台会打出2个hehe 
![](http://dl2.iteye.com/upload/attachment/0086/3147/4ea56a07-0207-3421-9f64-247c355f2cee.png)

然后故意将filter中的代码改成阻塞的
```
@Override
public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {

	synchronized (this) {

		System.out.println("hehe");

		try {
			Thread.sleep(10000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```
同样开2个窗口访问，这次只能打出一个hehe，即只有一个请求能够得到响应，其他的请求全部阻塞 
![](http://dl2.iteye.com/upload/attachment/0086/3149/b01244d3-1afa-3166-97db-446eb26c97ee.png)

当然，在struts2中，肯定不会犯这样的低级错误。struts2 filter中的代码，都是非阻塞的，当调用action时，也是以异步执行的方式处理，避免阻塞并发请求 

但是这个现象提示我们，对于全局的filter，是所有请求的入口，在其中一定不能执行耗时过长的阻塞方法，否则对应用的并发性是毁灭性打击。由于自定义filter的场景是非常常见的，所以必须加以注意