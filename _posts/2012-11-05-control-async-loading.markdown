---
layout: post
title: (翻译)ControlJS第一部分——async loading
---

* 作者：Steve Souder
* 原文：[http://www.stevesouders.com/blog/2010/12/15/controljs-part-1/](http://www.stevesouders.com/blog/2010/12/15/controljs-part-1/)

这个关于ControlJs（一个让脚本加载更快的Javascript模块）三篇文章中的第一篇。这三篇文章描述了ControlJS如何用于[异步加载]、[延迟执行]、[]

关于第一部分的异步加载，这个的关键在于尽快将页面作为html绘制出来，然后再用javascript进行优化，或者说用js进一步渲染。我们看到过很多网页，展示给用户一片空白，只是因为在等待几百KB的js文件下载、解析、执行，而这些js是用来绘制页面中展示给用户看的DOM元素。

其实上述的情况，很容易理解–但是真正实现改进恐怕就困难的多。好几百KB的js必须要拆散重组。那些通过js来创建网页DOM的逻辑代码必须全部合成到后台的服务器端去生成html。即使使用最新的服务器端JS来实现，也是一项很大的重构工程。

我又回到了Opera的Delayed Script Execution选项。一旦这个选项生效，js会被搁置起来(move aside)，让页面首先进行渲染。而这个选项确实很棒，而且没有网站因为开启了这个选项而导致页面报错的。我也一直持续在和其他浏览器的供应商联系，让他们也实现这个功能，我迫切的希望开发人员能够尽快的使用上这个选项。

近些年来，一些网页加速器（例如Aptimize, Strangeloop, FastSoft, CloudFlare, Torbit,和最近的 mod_pagespeed）犹如雨后春笋般脱颖而出。他们修改了页面中的HTML标签，用来改善网页的性能。从这些模式中，我摸索出了一个方法，比起延迟加载页面渲染的JS，我们可以更容易的改变HTML标记来实现。

于是ControlJS诞生了！

### Controlling download and execution

ControlJS的目标是让开发员更好的控制JS的加载。关键是要意识到“加载”氛围两个步骤：下载[download](即获取内容)和执行[execution](包含解析).为了更好的性能，这两个步骤有必要分离控制。

在对网页性能敏感的程序员眼中，如何控制脚本下载是一个很广泛流行的话题。当脚本使用普通的方式(也就是`<script src=”"` …的形式)，脚本会阻塞其他资源的下载（在新版本浏览器中稍微好一些），同时也会阻塞页面的渲染（所有浏览器都有这种情况）。使用异步脚本加载(asynchronous script loading)的技术在很大程序上缓解了这个问题。我在Even Faster Web Sites中提及过几种异步加载的技术。LABjs 和 HeadJS是提供异步加载的js模块。虽然它们的异步技术解决了脚本下载阶段的阻塞问题，但是并没有解决脚本执行时发生的情况。

在脚本执行阶段，页面渲染和新资源下载会被阻塞。在如今网页中的脚本越来越多，在脚本执行阶段的阻塞问题就更为严重，尤其是在移动设备上。事实上，Gmail移动设备组考虑到脚本执行带来的问题，它们完善了new async technique that downloads JavaScript wrapped in comments。这项技术可以使得脚本执行和下载分离。脚本会下载到客户端（在浏览器的缓存中），但是因为它是脚本注释，所以在执行阶段不会阻塞（因为不会执行）。当用户使用了相关功能的时候，需要某些脚本时，会将注释符号删除，然后eval执行代码。

Stoyan Stefanov, 一名优秀的性能优化前驱者，最近发布了一篇博文preloading JavaScript without execution.他用IMAGE或者OBJECT(这取决于浏览器）来下载脚本。假设这些脚本可以缓存在客户端中，随后可以使用SCRIPT标签插入脚本。这个也是在ControlJS中使用的技术。

### ControlJS: how to use it

使用ControlJS涉及到三个步骤的修改

修改点#1：添加control.js

我想关于脚本加载模块的使用，最讽刺的地方就在于，加载这些异步脚本加载种子文件是会阻塞页面的。所以从一开始我就必须确认ControlJS是要能被异步加载的。

	var cjsscript = document.createElement('script');
	cjsscript.src = "control.js";
	var cjssib = document.getElementsByTagName('script')[0];
	cjssib.parentNode.insertBefore(cjsscript, cjssib);

修改点#2：修改外链的脚本
下一步就是将所有外链脚本的标签改为ControlJS的形式。原本的脚本引用如下

	<script type="text/javascript" src="main.js"><script>

SCRIPT元素的TYPE要改为”text/cjs”,SRC改为“data-cjssrc”，如下：

	<script type="text/cjs" data-cjssrc="main.js"><script>

修改点#3 修改内联的脚本
大多数的页面都会有内联的脚本。这些脚本有一些依赖性：内联脚本中的变量一般依赖于外链的脚本，反之亦然。很重要的一点是：内敛脚本和外链脚本的执行顺序是需要保证的。因此，内联脚本必须改变TYPE属性的值

	<script type="text/javascript">
	var name = getName();
	<script>

修改“text/javascript”为”text/cjs”,如下：

	<script type="text/cjs">
	var name = getName();
	<script>

这样就ok了！后续的一系列工作就交给ControlJS担心吧！

###ControlJS: how it works


现在脚本不会阻塞页面了，因为TYPE属性已经改为浏览器无法识别的值了。这也就让ControlJS可以用一种高性能的方法更好的控制脚本加载。让我们大概看看ControlJS是如何运作的呢？当然，你也可以查看control.js 的代码来更详细的了解。

我们都希望能够尽快的开始下载脚本。因为我们使用IMAGE或者OBJECT来下载脚本，所以它们不会在下载阶段阻塞页面。而且他们并不是作为SCRIPT下载的，所以它们也不会执行。ControlJS首先找出所有SCRIPT的，并且type为”text/cjs”的标签。如果脚本有一个DATA-CJSSRC，那么IMAGE(IE和Opera浏览器）或者OBJECT(其他浏览器）就会根据它们的URL动态的创建出来。（你可以查看Stoyan’s post去了解更详细的内容）。

一般来说，ControlJS会在window load的时间之后进入执行阶段（当然也可以立刻执行代码或者某个DOM元素加载完毕了）。ControlJS会第二次遍历所有的脚本，确保它们都正确的出现在页面中。如果这个脚本是一个内联脚本，那么它的代码会被执行。如果这个脚本是一个外链脚本，由IMAGE或者OBJECT动态下载下来的，那么它会作为一个SCRIPT标签插入到页面中，那么它的代码也会随之解析并且执行。如果IMAGE或者OBJECT并没有下载结束，那么会在一个短暂的时间间隔之后(a short timeout)，重新进入刚才的遍历流程。

后续的文章中我还会讨论关于document.write的功能，以及跳过执行阶段。这篇中，我们首先来看一个简单的异步加载的例子。

### Async example

为了能更好展示异步加载的情况，我创建了一个文件，文件顶部包含三个脚本：

main.js – 耗费4秒下载
一段内联脚本会使用main.js的一个变量
page.js – 耗费2秒下载，使用内联脚本的一个变量
我让page.js的下载时间比main.js的下载时间短，是为了确认这些脚本可以以正确的顺序执行(尽管page.js下载更快）。我同事也包含了内联的脚本，因为这也是很多页面的实际情况（例如Google Analytics)，但是很多脚本加载器并不支持内联脚本的执行顺序。

Async withOUT ControlJS是一个基本范例。它使用了最普通的方式加载脚本。可以看一下如下图所示的IE8下的HTTP瀑布图(使用HttpWatch).IE8比IE6和IE7要好 – 能够使main.js和page.js并行加载。但是所有的IE都会因为脚本加载而阻塞图片的下载。所以images1-4都延迟了。在所有浏览器中，页面渲染都会被阻塞。在图示中也可以看出，main.js阻塞渲染长达4秒钟（绿色的竖线表示页面开始渲染时间）

![](http://stevesouders.com/images/controljs-async-baseline.png)

Async WITH ControlJS 演示了ControlJS是如何解决由于脚本引发的阻塞问题。不像上面的例子，脚本和图片会并行加载。同时页面渲染也是立刻开始的。如果你同时在浏览器中演示这两个页面，你会发现ControlJS的页面会快很多。当然，你也发现在下面的图示中多了3个请求，一个是Control.js – 这个是用来异步加载脚本用的代码。另外两个请求时因为main.js和page.js下载了两次。第一次是使用IMAGE或者OBJCET下载，第二次是使用SCRIPT标签加入页面用来执行代码的。因为main.js和page.js已经存在在浏览器缓存中了，所以他们不需要额外的下载时间。只是短暂的缓存读取时间（也就是那两根很细的蓝色线）。

![](http://stevesouders.com/images/controljs-async.png)


### The ControlJS project

ControlJS是在Apache License下面的开源项目。你可以在ControlJS Google Code project中找到control.js的代码。关于它的讨论可以在ControlJS Google Group.关于上面提到的例子放在我的站点 ControlJS Home Page.我只在大多数的浏览器中测试了，如果要投入到大规模的产品环境中，请进行更详尽的测试。

###Only part1

这只是ControlJS的第一部分。目前看来似乎已经解决了所有问题了。如果我们只是需要异步加载，那么其他的一些技术其实也已经实现了。不过我的目标是不能让脚本阻塞页面的渲染。我们就必须面临一个问题，如果处理document.write.另一个方面就是类似于Gmail的移动组实现的- 下载脚本同时在执行阶段不会阻塞页面渲染。我们会在下面的两篇博文中详细分析。