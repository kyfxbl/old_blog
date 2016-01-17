title: chrome extension例子
date: 2013-09-24 11:22
categories: 其它 
---
作为练手，昨晚开发了一个chrome的扩展。效果是在页面上选中文本，然后在右键菜单里点击按钮，保存到localStorage里，在popup window里可以看到保存的笔记
<!--more-->

数据是保存在localStorage里的，由于localStorage只能保存String，但是应用的数据结构需要用到object，所以要用JSON的方法转化一下，详见代码 

![](http://dl.iteye.com/upload/attachment/0081/5085/2c9c8ec5-6292-3533-8187-07c0f634f05b.png)

扩展用到了background javascript，用来生成右键菜单、处理点击；以及一个popup页面，用来展示保存的笔记；由于没有和页面的直接交互，就不需要用到content script 

首先是manifest
```
{
  "manifest_version": 2,
  "name": "kyfxbl note",
  "description": "This extension notes text content from page",
  "version": "1.4",
  "background": {
    "scripts": ["common.js","uuid.js","background.js"]
  },
  "browser_action": {
    "default_icon": "icon.png",
    "default_popup": "popup.html",
    "default_title": "kyfxbl note"
  },
  "permissions": [
    "tabs",
    "contextMenus",
    "http://*/*"
  ]
}
```

然后是background.js，在chrome启动时就会在后台运行

```
initLocalStorage();

// see http://developer.chrome.com/extensions/contextMenus.html for details
var createMenuProp = {
	"title" : "note",
	"contexts" : [ "selection" ],
	"onclick" : noteIt
};

chrome.contextMenus.create(createMenuProp);

function noteIt(info, tab) {
	var uuid = Math.uuid(16);
	var note = new Note(info.selectionText);
	var object = JSON.parse(localStorage.mynotes);
	object[uuid] = note;
	localStorage.mynotes = JSON.stringify(object);
}
```

最后是popup.js，点击时显示笔记

```
<!doctype html>
<html>

<head>
    <link type="text/css" rel="stylesheet" href="popup.css" />
    <script src="jquery-1.8.0.js"></script>
    <script src="common.js"></script>
    <script src="popup.js"></script>
    <title>kyfxbl note</title>
</head>

<body>
	<div id="wrapper"></div>
	<button id="btn_clear">clear</button>
</body>

</html>
```

```
$(document).ready(function() {

	$("#btn_clear").click(clearLocalStorage);

	renderNotes();

	function renderNotes() {

		$("#wrapper").empty();// 先清空页面

		var notes = JSON.parse(localStorage.mynotes);

		$.each(notes, function(index, value) {

			var $div = $("<div>");
			var content = value.content;
			var $content = $("<span class='content'>" + content + "</span>");
			var $uuid = $("<span class='uuid'>" + index + "</span>");
			var $button = $("<button>delete</button>");
			$button.click(deleteCurrentNote);

			$div.append($content);
			$div.append($uuid);
			$div.append($button);
			$("#wrapper").append($div);

		});
	}

	function clearLocalStorage() {
		localStorage.clear();
		initLocalStorage();
		renderNotes();
	}

	function deleteCurrentNote() {
		var uuid = $(this).prev().text();
		var object = JSON.parse(localStorage.mynotes);
		delete object[uuid];
		localStorage.mynotes = JSON.stringify(object);
		renderNotes();
	}

});
```

感觉google出的东西都比较好学，因为它的架构很清晰，相关的文档比较全，API设计得也比较好 

当学习一个新的框架或者平台的时候，可以按三步走： 

1、了解场景和边界，即这个东西是用在什么场合，可以解决什么问题，不能解决什么问题 

2、学习架构，在高层面上明白有什么组件，怎么交互。比如android的4种组件，chrome extension的background、browser action、content script的关系 

3、学习API，在上述2个步骤以后，就可以深入到reference里，看看都有什么API，应该怎么调用。每个平台或者框架能做什么事情，最终都是由API来体现的