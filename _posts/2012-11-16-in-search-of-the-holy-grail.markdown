---
layout: post
title: 网页布局——圣杯布局
---

相关的材料：

In Search of the Holy Grail：[http://www.alistapart.com/articles/holygrail/](http://www.alistapart.com/articles/holygrail/)

三列布局，也叫做圣杯布局【Holy Grail of Layouts】是Kevin Cornell在2006年提出的一个布局模型概念，也叫做双飞翼布局，该圣杯布局（三列布局）的特点：

- 1、有一个流体中心与固定宽度侧边栏
- 2、允许中心列在源代码中最早出现
- 3、允许任何列是最高的
- 4、只需要一个额外的div标记和非常简单的CSS,以及最少的hack、补丁

例子：三列布局，一个固定宽度是200像素的左列和一个固定宽度是150像素的右列。

HTML代码：
	
	<div id="header"></div>
	
	<div id="container">
	  <div id="center" class="column"></div>
	  <div id="left" class="column"></div>
	  <div id="right" class="column"></div>
	</div>
	
	<div id="footer"></div>

CSS代码：

	body {
	  min-width: 550px;      /* 2x LC width + RC width */
	}
	#container {
	  padding-left: 200px;   /* LC width */
	  padding-right: 150px;  /* RC width */
	}
	#container .column {
	  position: relative;
	  float: left;
	}
	#center {
	  width: 100%;
	}
	#left {
	  width: 200px;          /* LC width */
	  right: 200px;          /* LC width */
	  margin-left: -100%;
	}
	#right {
	  width: 150px;          /* RC width */
	  margin-right: -150px;  /* RC width */
	}
	#footer {
	  clear: both;
	}
	/*** IE6 Fix ***/
	* html #left {
	  left: 150px;           /* RC width */
	}

最终效果图：

![](http://www.alistapart.com/d/holygrail/diagram_05.gif)	

这种技术适用于所有现代浏览器:Safari、Opera,Firefox,(需配合底部的hack)IE6

分解步骤：

1、我们先根据我们所想要的左列和右列宽度来给container设定内边距
	
	<div id="header"></div>
	<div id="container"></div>	
	<div id="footer"></div>
	
	#container {
		  padding-left: 200px;   /* LC width */
		  padding-right: 150px;  /* RC width */
	}

![](http://www.alistapart.com/d/holygrail/diagram_01.gif)

2、往container中插入列：

	<div id="header"></div>
	
	<div id="container">
	  <div id="center" class="column"></div>
	  <div id="left" class="column"></div>
	  <div id="right" class="column"></div>
	</div>
	
	<div id="footer"></div>

添加适当的宽度和浮动它们让它们在同一线上。我们也需要页脚清除浮动来保持它在浮动列下方

	#container .column {
	  float: left;
	}
	#center {
	  width: 100%; /* CC（center column）*/
	}
	#left {
	  width: 200px;  /* LC（left column ） width */
	}
	#right {
	  width: 150px;  /* RC（right column） width */
	}
	#footer {
	  clear: both;
}

center的width设置为100%,也就是container除去padding后的宽度。
![](http://www.alistapart.com/d/holygrail/diagram_02.gif)

3、将左列拖到恰当的地方

剩下唯一要做的事情就是将这些列放到container的padding位置去。首先从左列开始。

需要两步将左列放到适当的地方。

第一步、我们用100%的负外边距让它穿过整个中心列去到恰当的位子。记住，100%是指容器的内容宽度，这也正是中心列的层宽度。
	
	#left {
	  width: 200px;        /* LC width */
	  margin-left: -100%;  
	}

![](http://www.alistapart.com/d/holygrail/diagram_03.gif)

第二步、将使用相对定位，偏移量恰好是左列的宽度让左列推到旁边位置。

	#container .columns {
	  float: left;
	  position: relative;
	}
	#left {
	  width: 200px;        /* LC width */
	  margin-left: -100%;  
	  right: 200px;        /* LC width */
	}

![](http://www.alistapart.com/d/holygrail/diagram_04.gif)

4、将右侧推到适当的位置。

我们只需要将它从容器的主体脱离，进入到容器的padding。

	#right {
	  width: 150px;          /* RC width */
	  margin-right: -150px;  /* RC width */
	}

![](http://www.alistapart.com/d/holygrail/diagram_05.gif)

5、设计的防守

浏览器窗口大小缩放导致中间部分宽度小于左列宽度，布局在符合标准的浏览器中就破坏了。给body设置一个min-width可以让列保持在适当的位置。IE6中这种事情讲不会发生，因此IE6不支持min-width这个不是跟问题。

	body {
	  min-width: 550px;  /* 2x LC width + RC width */
	}

修复IE6下的bug：因为负边距使左列在IE6下(完整的浏览器窗口的宽度)过分左倾

	* html #left {
	  left: 150px;  /* RC width */
	}

`* html `因为IE6下不是以HTML作为文档的根节点，所以这种hack只有IE6识别。

6、给模块设置padding

为了让文章更加的美观，给模块设置一个padding，使得内容不会顶到border。需要注意的是，左右边距若设置了padding的值，需要保证：old width = 2 * padding + new width

	body {
	  min-width: 630px;      /* 2x (LC fullwidth +
	                            CC padding) + RC fullwidth */
	}
	#container {
	  padding-left: 200px;   /* LC fullwidth */
	  padding-right: 190px;  /* RC fullwidth + CC padding */
	}
	#container .column {
	  position: relative;
	  float: left;
	}
	#center {
	  padding: 10px 20px;    /* CC padding */
	  width: 100%;
	}
	#left {
	  width: 180px;          /* LC width */
	  padding: 0 10px;       /* LC padding */
	  right: 240px;          /* LC fullwidth + CC padding */
	  margin-left: -100%;
	}
	#right {
	  width: 130px;          /* RC width */
	  padding: 0 10px;       /* RC padding */
	  margin-right: -190px;  /* RC fullwidth + CC padding */
	}
	#footer {
	  clear: both;
	}
	
	/*** IE Fix ***/
	* html #left {
	  left: 150px;           /* RC fullwidth */
	}
7、等高列

	<div id="footer-wrapper">
	  <div id="footer"></div>
	</div>
	
	#container {
			overflow: hidden;
		}

	#container .column {
		padding-bottom: 1001em;     /* X + padding-bottom */
		margin-bottom: -1000em;     /* X */
	}

	---fix IE6 --
	* html body {
	  overflow: hidden;
	}
	* html #footer-wrapper {
	  float: left;
	  position: relative;
	  width: 100%;
	  padding-bottom: 10010px;
	  margin-bottom: -10000px;
	  background: #fff;         /* Same as body 
	                               background */
	}

例子：[http://www.alistapart.com/d/holygrail/example_3.html](http://www.alistapart.com/d/holygrail/example_3.html)