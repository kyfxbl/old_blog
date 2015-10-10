title: struts2典型场景的配置
date: 2013-09-24 11:33
categories: java 
---
本文总结struts2几种典型场景的配置
<!--more-->

# web.xml

```
<filter>
		<filter-name>struts2</filter-name>
		<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>*.action</url-pattern>
	</filter-mapping>  

	<servlet>  
        <servlet-name>CXFServlet</servlet-name>  
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>  
        <load-on-startup>1</load-on-startup>  
    </servlet>  

    <servlet-mapping>  
        <servlet-name>CXFServlet</servlet-name>  
        <url-pattern>/webservice/*</url-pattern>  
    </servlet-mapping>
```

我用的struts2的版本是2.3.4.1，所以这里的<filter>应该是StrutsPrepareAndExecuteFilter，不再是旧的DispatcherFilter 

另外这里的<filter-mapping>，我配的不是
```
/*
```
而是
```
*.action
```
目的是不把特殊路径的请求，交给struts2处理，比如下面的/webservice/*，就不要经过struts2 

# struts.xml

```
<package name="bookManage" extends="json-default" namespace="/book">

		<interceptors>

			<interceptor name="loginInterceptor" class="loginInterceptor" />

			<interceptor-stack name="secureStack">
				<interceptor-ref name="loginInterceptor" />
				<interceptor-ref name="defaultStack" />
			</interceptor-stack>

		</interceptors>

		<default-interceptor-ref name="secureStack" />

		<global-results>
             <result name="login" type="redirectAction">../login.action</result>
        </global-results>

		<action name="list" class="bookAction" method="list">
			<result name="success">../jsp/bookManage/bookList.jsp</result>
		</action>

		<action name="delete" class="bookAction" method="delete">
			<result name="success" type="redirectAction">list.action</result>
		</action>

		<action name="originAjax" class="bookAction" method="originAjax" />

		<action name="pluginAjax" class="bookAction" method="pluginAjax">
			<result type="json">
				<param name="excludeNullProperties">true</param>
			</result>
		</action>

	</package>
```

以上的配置有简化，分别针对4种不同的场景

## Action处理后跳转到jsp页面

```
<action name="list" class="bookAction" method="list">
			<result name="success">../jsp/bookManage/bookList.jsp</result>
		</action>
```

这里默认的resultType是dispatcher 

对应的Action写法

```
public String list() {
		books = bookService.getAllBooks();
		return SUCCESS;
	}
```

## Action处理后，跳转到另外一个Action

相当于Servlet规范中的redirect

```
<action name="delete" class="bookAction" method="delete">
			<result name="success" type="redirectAction">list.action</result>
		</action>
```

这里的resultType是redirectAction 

对应的Action写法

```
public String delete() {
		bookService.deleteBookById(id);
		return SUCCESS;
	}
```

## 在Action中直接写响应，不跳转

```
<action name="originAjax" class="bookAction" method="originAjax" />
```

这里就没有<result>元素 

对应的Action写法

```
public void originAjax() throws IOException {
		HttpServletResponse response = ServletActionContext.getResponse();
		PrintWriter writer = response.getWriter();
		writer.print("hello " + ajaxField);
		writer.flush();
		writer.close();
	}
```

## 通过json插件，返回json字符串，不跳转

```
<action name="pluginAjax" class="bookAction" method="pluginAjax">
			<result type="json">
				<param name="excludeNullProperties">true</param>
			</result>
		</action>
```

这里的resultType是json，这种方式本质上和第三种一样 

对应的Action写法
```
public String pluginAjax() {
		ajaxField = "hello " + ajaxField;
		return SUCCESS;
	}
```

另外，这里面配置了拦截器、拦截器栈、全局result，可以把这些东西提取到公共的package里，让其它的业务子package来extends