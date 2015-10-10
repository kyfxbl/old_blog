title: html5的appcache
date: 2013-09-24 11:56
categories: web
---
从HTML5开始，支持将页面资源（包括.html、.css、.js、.png）等缓存起来，从而实现离线应用 
<!--more-->

# 准备appcache清单文件 

清单文件可以使用任何扩展名，只需要在web server里注册即可，这里起名为demo.appcache

```
CACHE MANIFEST

CACHE:
5.html
5.css
jquery-1.8.0.js
5.js
```

# 注册appcache清单文件的MIME 

对于tomcat，在%TOMCAT_HOME%/conf/web.xml里配置

```
<mime-mapping>
        <extension>appcache</extension>
        <mime-type>text/cache-manifest</mime-type>
</mime-mapping>
```

# 关联清单文件

```
<!DOCTYPE html>
<html manifest="demo.appcache">

<head>
	<meta charset="UTF-8">
	<link type="text/css" rel="stylesheet" href="5.css" />
	<script type="text/javascript" src="jquery-1.8.0.js"></script>
	<script type="text/javascript" src="5.js"></script>
	<title>5</title>
</head>

<body>
	<div id="myDiv">hello world</div>
	<button id="btn">run</button>
</body>

</html>
```

# 编程控制applicationCache对象

```
$(document).ready(function() {

	$("#btn").click(showCache);

	function showCache(){
		alert(applicationCache.status);
		applicationCache.update();
	}
});
```

# 效果 

通过上面的配置，5.html页面相关的资源就会被支持ApplicationCache的浏览器缓存起来，支持离线操作 

关于applicationCache对象，其实还有一些别的API，支持状态和事件绑定等，详细的文档参见： 

[http://www.thecssninja.com/javascript/how-to-create-offline-webapps-on-the-iphone](http://www.thecssninja.com/javascript/how-to-create-offline-webapps-on-the-iphone) 
[http://www.w3schools.com/html/html5_app_cache.asp](http://www.w3schools.com/html/html5_app_cache.asp)