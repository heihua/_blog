---
layout: post
title: 高性能Javascript(1、加载和运行)
---

### Script Positioning

雅虎第一条军规：将脚本放在页面底部（**put scripts at the bottom**）。此方法可以保证页面在脚本允许之前完成解析。

### Grouping Scripts


每个HTTP请求都会产生额外的性能，因此下载一个仅仅100KB的文件会比下载4个25KB的文件快。总之，减少你页面相关的外部文件的数量是很有用的。

使用combo，减少script的标签数量

例如：
	`<script type="text/javascript"
	src="http://yui.yahooapis.com/combo?2.7.0/build/yahoo/yahoo-min.js&2.7.0/build/event/event-min.js"></script>`

### Nonblocking Scripts 非阻塞脚本

在页面完成加载之后，再加载Javascript源文件。从技术角度角度讲，这意味着在window的load事件触发之后开始下载代码。
    
方法一：Deferred Scripts（延迟脚本）
	
	使用script标签的扩展属性——defer属性（HTML4定义），defer表明该脚本不打算修改DOM，因此代码被下载，但是不会执行，直到DOM全部加载完，在window的onload被调用之前被执行。defer无论在内联脚本还是在外部脚本文件，都是这样。这个属性只被IE4+和Firefox4.5+支持，其他浏览器上，defer属性被忽略。如果浏览器支持的话，它仍是一种很有用的解决方案。
	例如：`<script type="text/javascript" src="file1.js" defer></script>`
方法二：Dynamic Script Elements（动态脚本元素）

	使用Javascript动态的创建`<script>`元素。使用Javascript指定他的type、src等属性。将它插入页面中。
	
	当这个元素被添加到页面中的时候，才开始下载文件。此技术重点在于：无论您在何处开始，文件的下载和运行都不会阻塞其他页面处理过程。

	function loadScript(url, callback){
		var script = document.createElement ("script")
		script.type = "text/javascript";
		if (script.readyState){ //IE
			script.onreadystatechange = function(){
				if (script.readyState == "loaded" || script.readyState == "complete"){
						script.onreadystatechange = null;
						callback();
				}
			};
		} else { //Others
			script.onload = function(){
				callback();
			};
		}
		script.src = url;
		document.getElementsByTagName_r("head")[0].appendChild(script);
	}

	动态脚本加载时非阻塞Javascript下载中最常用的模式，因为他可以跨浏览器，而且简单易用。
方法三：XMLHttpRequest Script Injection XHR脚本注入
	
	var xhr = new XMLHttpRequest();
		xhr.open("get", "file1.js", true);
		xhr.onreadystatechange = function(){
		if (xhr.readyState == 4){
			if (xhr.status >= 200 && xhr.status < 300 || 	xhr.status == 304){
				var script = document.createElement ("script");
				script.type = "text/javascript";
				script.text = xhr.responseText;
				document.body.appendChild(script);
			}
		}
	};
	xhr.send(null);

优点：1、可以下载但不立即执行Javascript代码。script.text = xhr.responseText;因此代码的返回在< script>之外，它下载后不会立即执行，直到你把它包在< script>中，并插入到页面之后才会执行。2、同样的代码在所有模式的浏览器下都不会发生异常。
限制：Javascript和页面必须放置在同一个域内。不能从CND上下载。请求中使用的是相对路径。


### Recommended Nonblocking Pattern

第一步：下载页面初始化时必要的javascript文件，以及动态加载Javascript必须的代码。这部分的代码尽量小。
第二步：调用以下载的动态加载Javascript的代码来加载剩余的Javascript。

### The YUI 3 approach

核心设计理念：用很小的初始代码，下载其余的功能代码。

### The LazyLoad library

 LazyLoad可以下载多个Javascript文件，并保证它们在所有浏览器上都能按照正确的顺序执行。

	LazyLoad.js(["first-file.js", "the-rest.js"], function(){
		Application.init();
	});
	
### The LABjs library

独特之处在于它能够管理依赖关系，wait函数允许你指定哪些文件应该等待其他文件。




