title: httpd设置反向代理
date: 2013-09-24 11:25
categories: web 
---
用httpd可以实现反向代理Reverse Proxy 
<!--more-->

官网的说明： 

httpd also allows you to bring remote documents into the URL space of the local server. This technique is called reverse proxying because the web server acts like a proxy server by fetching the documents from a remote server and returning them to the client. It is different from normal (forward) proxying because, to the client, it appears the documents originate at the reverse proxy server. 

实际配置也比较简单 先要加载一大堆跟proxy相关的module，因为我还没研究module，也不知道哪个是，就把看得像的都开了

```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
LoadModule proxy_scgi_module modules/mod_proxy_scgi.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_express_module modules/mod_proxy_express.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
```

最后一个不load会报错，应该是某一个module依赖了它。但是我只是为了测试，用到最简单的跳转功能，应该是不需要开这么多的，后面再仔细研究下，现在先work起来

```
<IfModule proxy_module>
    ProxyPass /wfm/ http://localhost:8080/wfm/
</IfModule>
```

用directive ProxyPass，把所有匹配/wfm/的请求，都直接转发到后端的tomcat上，后端其实是一个servlet应用，context是wfm 

后端有一个HelloWorldServlet

```
protected void doGet(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		PrintWriter writer = response.getWriter();
		writer.write("it works by servlet");
		writer.flush();
		writer.close();
	}
```
直接访问后端的web app也是可以的 

![](http://dl2.iteye.com/upload/attachment/0086/6504/37cb4a6b-e469-3b59-8511-df28d5ccfa72.png)

但是通过反向代理来访问，应该要用httpd的端口 

![](http://dl2.iteye.com/upload/attachment/0086/6506/b0c74df6-af1e-3e38-a84d-2ebcc6357cb7.png)

对于浏览器来说，它并不知道内容实际上是由后端服务器提供的，只会认为自己访问的是httpd server。这就是反向代理名字的由来 

在这个例子里，httpd和tomcat在同一台server上，所以没有什么意义。实际上，一般httpd server会放在公网，而tomcat server放在内网（无公网IP），所以用户是不能直接访问到tomcat server的，对安全很有好处 

如果换一个路径，比如http://localhost/wfm1/，httpd就不会做跳转了，会在本机的file system里找 

![](http://dl2.iteye.com/upload/attachment/0086/6508/3f652df0-4437-3865-b4cb-8a99c397a2b1.png)

通过这个最简单的示例，实现了基本的反向代理，不过要注意以下几点： 

1、除了ProxyPass，还有相关的其他directive，比如ProxyPassReverse、ProxyPassReverseCookieDomain等，可以实现更复杂的配置，后面要研究一下 

2、我感觉这种反向代理，就是简单的HTTP请求转发，应该有更好的方式。比如httpd和tomcat集成，有专门的通信协议AJP，应该会比较好（httpd也提供了proxy_ajp_module，tomcat里也有ajp protocol connector） 

3、开发servlet app的时候，一般会有非常多的页面URL跳转，一定要写成相对路径，这样在做反向代理的时候会比较简单。如果跳转路径写死了，跟IP绑定，那么反向代理就没法做了（浏览器会直接访问servlet server，跳过了proxy）。不过这点跟做不做反向代理没有关系，是web app开发的一般原则