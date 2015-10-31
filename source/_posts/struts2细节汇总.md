title: struts2细节汇总
date: 2013-09-24 11:11
categories: java 
---
本文总结一些struts2的细节问题
<!--more-->

# 关于struts2验证框架的流程 

struts2里的校验，实际上是由几个拦截器，和几个接口共同完成的 

如果Action继承自ActionSupport类，那么就实现了Validateable接口和ValidationAware接口，提供了validate()方法、hasError()方法，以及一组添加错误信息的方法（最常用的是addFieldError()方法） 

默认包的拦截器栈有一部分是这样的： 

```
params --> conversionError --> validation --> workflow
```
 
首先params拦截器和conversionError拦截器先起作用，把HTTP请求参数放入ValueStack中，如果有错误，会调用ValidationAware接口的添加错误的方法（其中一个是addFieldError） 

然后到了validation拦截器，它会执行验证框架，如果有错误，也调用ValidationAware上的方法 

最后到了workflow拦截器，第一个阶段它会判断Action是否实现了Validateable接口，如果是的话，则调用validate()方法，如果有错误，就调用ValidationAware的方法。然后第二阶段，它会调用hasError()方法，看看是否有错误，如果有的话，返回INPUT 

所以，实际上可以同时使用struts2的校验框架，和Action上的validate()方法。对于workflow拦截器第二阶段的工作（检查错误）来说，它并不清楚，错误来自于哪里，是来自于校验框架，还是来自validate()方法，对workflow拦截器来说没有区别，只要有错误，就返回INPUT 

# 关于i18n 

使用国际化资源文件，标准的做法是
```
<s:text name="homepage.greeting" />
```

这里假设资源文件里有一个key是homepage.greeting 如果不想用s:text标签的话，应该怎么办呢？

实际上，s:text标签调用的是TextProvider接口的getText()方法 

而ActionSupport实现了TextProvider接口，所以如果Action是继承自ActionSupport，那么它也就实现了TextProvider接口。同时，Action对象会被放入值栈，所以用OGNL表达式，是可以直接取到资源文件中的国际化文本的，方法是 
```
${getText("homepage.greeting")}
```

以上这句OGNL表达式，效果相当于

```
<s:text name="homepage.greeting" />
```
 
# struts2插件加载体现的一种设计思路 

struts2加载配置是遵循如下顺序： 

```
struts-default.xml -> struts-plugin.xml -> struts.xml
```
 
其中，struts-default是框架默认提供的，struts.plugin是插件提供的，struts是用户自定义的 

struts2框架启动时，会收集以上所有配置文件，然后进行聚合汇总，得到一个总体的配置信息 

这种思路很常见，比如MAVEN，也是首先提供了一个超级POM，和项目自定义POM进行聚合。这种设计思路，是值得借鉴的

# ResultType

struts2定义了多种resultType，包括dispatcher和redirectAction等 

在struts-default.xml文件中，定义了所有的resultType类型
```
<result-types>
            <result-type name="chain" class="com.opensymphony.xwork2.ActionChainResult"/>
            <result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>
            <result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>
            <result-type name="httpheader" class="org.apache.struts2.dispatcher.HttpHeaderResult"/>
            <result-type name="redirect" class="org.apache.struts2.dispatcher.ServletRedirectResult"/>
            <result-type name="redirectAction" class="org.apache.struts2.dispatcher.ServletActionRedirectResult"/>
            <result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>
            <result-type name="velocity" class="org.apache.struts2.dispatcher.VelocityResult"/>
            <result-type name="xslt" class="org.apache.struts2.views.xslt.XSLTResult"/>
            <result-type name="plainText" class="org.apache.struts2.dispatcher.PlainTextResult" />
        </result-types>
```

有空可以跟进去看看源码，这里重点说的是默认的dispatcher和redirectAction的区别 

![](http://dl.iteye.com/upload/attachment/0072/8613/613acc08-4be9-3892-89d7-6741fdf5a1e9.png)

这里点击“删除”，会执行一段javascript代码：

```
$(document).ready(function() {
	$(".delete_book").click(deleteBook);
});

function deleteBook() {
	var $deleteButton = $(this);
	var $idSpan = $deleteButton.parent().find(".hidden_book_id");
	var bookId = $idSpan.text();
	var url = "delete.action?id=" + bookId;
	window.location.href = url;
}
```

关键就是会跳转到delete.action?id=xxx这个URL，进入Action的delete()方法

```
@Autowired
	private IBookService bookService;

	private List<Book> books;// 书籍列表

	private String id;// 接收客户端传值

	public String list() {
		books = bookService.getAllBooks();
		return SUCCESS;
	}

	public String delete() {
		bookService.deleteBookById(id);
		books = bookService.getAllBooks();
		return SUCCESS;
	}
```

此时struts.xml的配置是这样的：

```
<package name="bookManage" extends="struts-default" namespace="/book">

		<action name="list" class="bookAction" method="list">
			<result name="success">../jsp/bookManage/bookList.jsp</result>
		</action>

		<action name="delete" class="bookAction" method="delete">
			<result name="success">../jsp/bookManage/bookList.jsp</result>
		</action>

	</package>
```

代码很简单，就不解释了。功能可以实现，不过有个问题，就是用这种配置方式，删除操作之后，URL是这样的： 

![](http://dl.iteye.com/upload/attachment/0072/8620/f1353d57-07c7-3d38-9cfa-18f80cd14884.png)

这时用户如果用F5刷新浏览器，就会发生重复提交的问题。所以可以这样修改，将resultType改成redirectAction

```
<package name="bookManage" extends="struts-default" namespace="/book">

		<action name="list" class="bookAction" method="list">
			<result name="success">../jsp/bookManage/bookList.jsp</result>
		</action>

		<action name="delete" class="bookAction" method="delete">
			<result name="success" type="redirectAction">list.action</result>
		</action>

	</package>
```
这样执行完delete.action后，会跳转进list.action，所以delete()方法也可以删除一行代码

```
@Autowired
	private IBookService bookService;

	private List<Book> books;// 书籍列表

	private String id;// 接收客户端传值

	public String list() {
		books = bookService.getAllBooks();
		return SUCCESS;
	}

	public String delete() {
		bookService.deleteBookById(id);
		return SUCCESS;
	}
```
实际看点击“删除”按钮的前后URL对比 

![](http://dl.iteye.com/upload/attachment/0072/8627/57298ede-dd40-3f45-8ec8-95e5449874f6.png)
![](http://dl.iteye.com/upload/attachment/0072/8629/29a9fdec-25e5-35de-a54d-c0db05c5285b.png)

实现了我们的目的。总结来说，默认的resultType是dispatcher，一般用来处理jsp跳转。如果希望一个action执行之后进入另一个action，可以用redirectAction，如果要多个action共同响应一个请求，可以用chain 

最后补充一点，通过redirectAction进入新的Action，会新创建一个Action的实例

```
public String list() {
		System.out.println("after redirectAction, currentBook is null or not");
		if (null == currentBook) {
			System.out.println("currentBook is null");
		} else {
			System.out.println("currentBook is not null");
		}
		books = bookService.getAllBooks();
		return SUCCESS;
	}
```

```
<action name="delete" class="bookAction" method="delete">
			<result name="success" type="redirectAction">list.action</result>
		</action>
```
输出结果： 

```
after redirectAction, currentBook is null or not 
currentBook is null
```

# OGNL相关技巧

OGNL有一些比较冷门的用法

## 在Result中使用OGNL表达式 

实际上除了在jsp里可以使用OGNL表达式之外，在Result的配置里也是支持的，这点在RedirectAction中尤其好用
```
<result type="redirectAction">
    <param name="actionName">anotherAction</param>
    <param name="param1">hardCodedValue</param>
    <param name="param2">${someValue}</param>
</result>
```

上面的param1和param2会成为请求的参数，其中param1是硬编码的，而param2是从ValueStack中取出的值 

## 在properties文件中使用OGNL表达式 

比如在resource.properties中，有一个greeting.word = hello ${user.name} 

然后在jsp页面使用标签
```
<s:text name="greeting.word" /> 
```

## OGNL会创建实例

如果Action里有一个字段user，然后jsp里提交user.name，则user的name字段会被自动赋值，但是实际上，user字段没有初始化过，为什么不会NPE呢 

这是OGNL在幕后起的作用，user.name是一个OGNL表达式，当OGNL解析器在属性链上发现一个为NULL的属性时，它会尝试创建一个实例并赋值 

对于开发者来说，只需要给这个类一个无参构造方法，并为此字段提供一个setter方法即可 

## OGNL字面量

OGNL表达式还可以用来直接创建List和Map 

```
{1,2,3}
```

这就创建了一个List 

```
#{"key1":"value1","key2":"value2"}
```

类似的，这就创建了一个Map 

这种语法一般是用在jsp页面里 

## OGNL操作符

OGNL还可以使用操作符 

比如
```
${user.age + 1}
```

## OGNL方法调用

```
<s:if test="page.hasNext()"> 
</s:if>
```

## OGNL调用静态方法和字段 

```
@com.huawei.test.Utils@someStaticMethod()
```

不过我认为这种写法是应该尽量避免的

# 好用的struts2-config-browser-plugin插件

放到WEB-INF/lib下，就安装了这个插件，十分简单 

之后访问
```
http://ip:port/app/config-browser/index.action
```

就可以打开当前struts2环境的信息页面 

![](http://dl.iteye.com/upload/attachment/0073/0227/d5ca1dad-8a28-357b-b3c0-a0c690c291ef.png)