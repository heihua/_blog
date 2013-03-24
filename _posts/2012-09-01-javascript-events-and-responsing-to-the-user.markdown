---
layout: post
title: Javascript Events And Responsing To The User 译文
---

原文：[http://coding.smashingmagazine.com/2012/08/17/javascript-events-responding-user/](http://coding.smashingmagazine.com/2012/08/17/javascript-events-responding-user/)

每当有人问我Javascript和DOM中，什么东西最强大时，我会立马想到事件(Events)。因为事件在浏览器中是非常有用的。而且通过事件降低功能的耦合性是一个非常厉害的想法，这也是为什么Node.js能够成为一个这么热的话题的原因。

今天，让我们回到基本的事件并开始使用它们，而不是对什么都应用点击操作或使用 < a href="javascript:void(0)" >阻止Web请求或使用onclick="foo()"这样的内联操作来搞乱我们的HTML页面（在2005的[why these are bad ideas](http://www.onlinetools.org/articles/unobtrusivejavascript/chapter4.html)一文中我已经详细做了解释）。

注释：这篇文章使用原生的JavaScript而不使用任何类库。这里我们将讨论的许多问题在jQuery、YUI、Dojo里是更容易实现的，但是理解基础知识是很重要的，因为你将发现在某些场合你将不能使用类库但仍需要你交付一个令人惊奇的解决方案。

**免责声明**：这里我们将使用的语法是addEventListener()，正如在”DOM Level 3 Events”规范中定义的，它可以在现在使用的所有浏览器中使用，除了IE9版本以下的IE浏览器。 这里我们将展示的很多东西都可以通过jQuery实现，尽管如此，他们也支持早期的浏览器。仔细想想，在DOMContentLoaded事件中一个简单addEventListener()是一个很好的方式确保你的脚本代码不会在早期的浏览器上运行。这是一个好事。如果我们想让浏览器得到发展，就需要停止给早期的浏览器复杂和牵强的代码。 如果你以正确的方式实现你的方案，IE6将不需要任何JavaScript代码去展示一个可行的，即使简单的方案。想象一下你的产品是一部自动扶梯：如果你的JavaScript不能运行的话，那么你的网站仍然可以被用作普通楼梯。

在我们正式学习事件细节和怎么使用它们之前，看看一些使用Scroll Event实现的demo，它们通过很灵巧的方式实现，确有非常美好的效果:

- 在针对设计师的调查中，Wealthfront工程使用了 s[crolling and shifting content along the Z axis.](https://www.wealthfront.com/designerwanted) 这曾经是[Beercamp 2011 website](http://2011.beercamp.com/)一个大的组成部分. Wealthfront的[blogged in detail](http://eng.wealthfront.com/2012/03/scrolling-z-axis-with-css-3d-transforms.html)解释了它是怎么实现
- [Stroll.js](http://lab.hakim.se/scroll-effects/)采用了一种比较相似的方式，展示了当用户滚动列表时，过渡是多么地友好。
- [jQuery Scroll Path 8](http://joelb.me/scrollpath/)是一个这样的插件，当用户滚动页面时它能够使内容按照一个路径移动。

所有的这些都是基于事件处理以及解读浏览器给我们提供了什么。现在，让我们看下重复执行的基础知识。

### 基础：什么是一个事件？

	var log = document.getElementById('log'),
	    i = '', 
	    out = [];
	for (i in window) {
	    if ( /^on/.test(i)) { 
		    out[out.length] = i; 
	    }
	}
	log.innerHTML = out.join(', ');

将这个例子在firefox中运行，我得到了这个信息：

onmouseenter, onmouseleave, onafterprint, onbeforeprint, onbeforeunload, onhashchange, onmessage, onoffline, ononline, onpopstate, onpagehide, onpageshow, onresize, onunload, ondevicemotion, ondeviceorientation, onabort, onblur, oncanplay, oncanplaythrough, onchange, onclick, oncontextmenu, ondblclick, ondrag, ondragend, ondragenter, ondragleave, ondragover, ondragstart, ondrop, ondurationchange, onemptied, onended, onerror, onfocus, oninput, oninvalid, onkeydown, onkeypress, onkeyup, onload, onloadeddata, onloadedmetadata, onloadstart, onmousedown, onmousemove, onmouseout, onmouseover, onmouseup, onmozfullscreenchange, onmozfullscreenerror, onmozpointerlockchange, onmozpointerlockerror, onpause, onplay, onplaying, onprogress, onratechange, onreset, onscroll, onseeked, onseeking, onselect, onshow, onstalled, onsubmit, onsuspend, ontimeupdate, onvolumechange, onwaiting, oncopy, oncut, onpaste, onbeforescriptexecute, onafterscriptexecute

以上信息意味着可以做很多好玩的操作，实现这些操作的方式就是使用函数addEventListener()：

	element.addEventListener(event, handler, useCapture);
例如：
	var a = document.querySelector('a'); // 获得文档中的第一个a标签
	a.addEventListener('click', ajaxloader, false);

element是我们使用处理程序的元素；好比：“嗨, 链接! 如果要你发生什么事情，请保证告诉我！”。 ajaxloader()函数是一个事件监听器；好比，”嗨! 你只要站在原地，谨慎小心的听着，以防链接发生事件”。 把useCapture设置成false意味着我们愿意在冒泡阶段捕获事件，而不是捕获阶段。这是一个长期和艰巨的课题（[long and arduous topic](http://www.w3.org/TR/DOM-Level-3-Events/#event-flow)）,已经在[Event capture explained](http://dev.opera.com/articles/view/event-capture-explained/)中得到了很好的解释。 把useCapture设置成false，在99.7434%（粗略精确）的情况下是不会有问题的。这个参数实际上是可选的，除了Opera浏览器。

现在，事件处理程序从事件中获得一个对象作为一个参数，它具有很多我们可以处理的丰富的属性。如果你试过了我的例子[try out my example](http://thewebrocks.com/demos/smashing-events/eventproperties.html)，你将知道下面的代码是做什么的：

	var log = document.getElementById('log'),
	i = 0,
	out = '';
	
	document.addEventListener('click', logeventinfo, false);
	document.addEventListener('keypress', logeventinfo, false);
	
	function logeventinfo (ev) {
		log.innerHTML = '';
		out = '<ul>';
		for (var i in ev) {
			if (typeof ev[i] === 'function' || i === i.toUpperCase()) {
				continue;
			}
		out += '< li>'+i+': '+ev[i]+'< /li>';
		}
		log.innerHTML += out + '</ul>';
	}

你可以给一个事件分配几个事件处理程序，或给多个事件分配同一个事件处理程序（就像在demo中展示的那样）

ev就是我从事件那取回来的，（再一次，将我的案例运行于Firefox中）它包含了很多有趣的信息：

	originalTarget: [object HTMLHtmlElement]
	type: click
	target: [object HTMLHtmlElement]
	currentTarget: [object HTMLDocument]
	eventPhase: 3
	bubbles: true
	cancelable: true
	timeStamp: 574553210
	defaultPrevented: false
	which: 1
	rangeParent: [object Text]
	rangeOffset: 23
	pageX: 182
	pageY: 111
	isChar: false
	screenX: 1016
	screenY: 572
	clientX: 182
	clientY: 111
	ctrlKey: false
	shiftKey: false
	altKey: false
	metaKey: false
	button: 0
	relatedTarget: null
	mozPressure: 0
	mozInputSource: 1
	view: [object Window]
	detail: 1
	layerX: 182
	layerY: 111
	cancelBubble: false
	explicitOriginalTarget: [object HTMLHtmlElement]
	isTrusted: true
	originalTarget: [object HTMLHeadingElement]
	type: click
	target: [object HTMLHeadingElement]
	currentTarget: [object HTMLDocument]
	eventPhase: 3
	bubbles: true
	cancelable: true
	timeStamp: 574554192
	defaultPrevented: false
	which: 1
	rangeParent: [object Text]
	rangeOffset: 0
	pageX: 1
	pageY: 18
	isChar: false
	screenX: 835
	screenY: 479
	clientX: 1
	clientY: 18
	ctrlKey: false
	shiftKey: false
	altKey: false
	metaKey: false
	button: 0
	relatedTarget: null
	mozPressure: 0
	mozInputSource: 1
	view: [object Window]
	detail: 1
	layerX: 1
	layerY: 18
	cancelBubble: false
	explicitOriginalTarget: [object Text]
	isTrusted: true

它也随着事件的不同而不同。试着点击demo和按下键盘，你将看到你获得了不同的结果。你也可以参考[full list of standard event properties](https://developer.mozilla.org/en/DOM/event#Properties)

### 最后一个基础知识：阻止执行并且获得目标

对于浏览器中的事件，有两个事情是更加重要的：一是我们必须阻止浏览器执行事件的默认操作，另一个是我们必须找出哪一个元素是事件针对的。前者可以通过函数ev.preventDefault()实现，后者是通过ev.target保存。

如果你想知道一个链接已经被点击，但是你不想让浏览器响应点击事件，因为你有一个更好的想法替代这个事件处理。你可以通过订阅链接的点击事件实现，并且你可以通过调用preventDefault()函数阻止浏览器的响应。下面是HTML代码：
	
	<a class="prevent" href="http://smashingmagazine.com">Smashing, my dear!</a>
	<a class="normal" href="http://smashingmagazine.com">Smashing, my dear!</a>

JavaScript代码：

	var normal = document.querySelector('.normal'),
	prevent = document.querySelector('.prevent');	 
	prevent.addEventListener('click', function(ev) {
		  alert('fabulous, really!');
		  ev.preventDefault();
	}, false);
	 
	normal.addEventListener('click', function(ev) {
	  alert('fabulous, really!');
	}, false);

注释：document.querySelector()是从DOM中获取元素的一个标准方式,就像在jQuery中使用$()方法一样。你可以查阅[W3C’s specification，](http://www.w3.org/TR/selectors-api/)也可以在Mozilla开发者网络（MDN）获得一些解释性的代码片段（[explanatory code snippets](https://developer.mozilla.org/En/Code_snippets/QuerySelector)）。

如果你现在点击链接，你将获得一个alert弹出框。当你点击弹出框的OK按钮时，没有更多的响应；浏览器不会转到http://smashingmagazine.com。不使用preventDefault(), 浏览器将显示警告并且转向链接，[Try it out](http://thewebrocks.com/demos/smashing-events/preventdefault.html).。

访问被点击、被悬停或键盘按下的元素的标准方式是在程序中使用this关键字。这是简洁和友好的方式，但是它实际上是受限制的，因为addEventListener()可以给我们更好的东西：事件目标。它还是会令人困惑的，因为this关键字可能已经已经被绑定了，所以像说明书中说的使用ev.currentTarget是一个更安全的方式。

### 事件委托：很流行，使用它！

使用事件对象的target属性，你可以知道事件发生在哪个元素上。

事件发生后向下遍历整个文档结构树到达交互的元素，然后再返回到主窗口。这意味着如果你对某个元素添加一个事件处理程序，那么你将很容易得到这个元素的所有子元素。这时，你所需要做的只是测试这个事件target,然后做出相应地响应。看看我的列表实例:

	<ul id="resources">
	    <li><a href="http://developer.mozilla.org">MDN</a></li> 
	    <li><a href="http://html5doctor.com">HTML5 Doctor</a></li>
	    <li><a href="http://html5rocks.com">HTML5 Rocks</a></li>
	    <li><a href="http://beta.theexpressiveweb.com/">Expressive Web</a></li>
	    <li><a href="http://creativeJS.com/">CreativeJS</a></li>
	</ul>

将鼠标悬浮在demo的一个列表上，你将看到一个事件处理程序将足以得到链接、列表项和列表本身。你所需要做的就是比较事件目标的tagName是否和你想获得是一样。

	var resources = document.querySelector('#resources'),
    log = document.querySelector('#log'); 
	resources.addEventListener('mouseover', showtarget, false);
	 
	function showtarget(ev) {
	  var target = ev.target;
	  if (target.tagName === 'A') {
	    log.innerHTML = 'A link, with the href:' + target.href;
	  }
	  if (target.tagName === 'LI') {
	    log.innerHTML = 'A list item';
	  }
	  if (target.tagName === 'UL') {
	    log.innerHTML = 'The list itself';
	  }
	}

这意味着你可以保存很多的事件处理程序——他们每一个对于浏览器来说代价都是很大的。为了取代对每一个链接都应用某个事件处理程序——就像使用jQuery中的$(‘a’).click(…)那样响应（尽管jQuery中的on用法是不错的），你可以对列表本身分配一个单独的事件处理程序，检查被点中的元素。

这样做最大的好处是使你的HTML是独立的，这样如果你在后期需要添加更多的链接，将没有必要给他们分配新的事件处理程序；事件处理程序将自动获知有一个新的链接需要去处理。

### 事件检测，流畅的CSS过渡

如果你还记得之前文章中的属性列表，那么我们就有很多事可以做了。在过去，我们使用事件实现简单的悬停效果，现在它已经被CSS的:hover和:focus选择器实现的效果所代替了。但是，某些事CSS确无法做到；比如，找到鼠标的位置。通过事件监听器，这就非常简单了。首先，我们定义一个元素的位置，比如一个球。HTML代码
	
	<div class="plot"></div>

CSS代码：

	.plot {
		position:absolute;
		background:rgb(175,50,50);
		width: 20px;
		height: 20px;
		border-radius: 20px;
		display: block;
		top:0;
		left:0;
	}

然后我们给文档分配一个点击处理程序并且定位球在pageX和 pageY位置上。需要注意，为了将让球的中心在鼠标上，我们必须将target得到的宽度减去球宽度的一半：

	var plot = document.querySelector('.plot'),
	offset = plot.offsetWidth / 2;
	document.addEventListener('click', function(ev) {
		plot.style.left = (ev.pageX - offset) + 'px';
		plot.style.top = (ev.pageY - offset) + 'px';
	}, false);

点击屏幕上的任何位置将立刻将球移到那里，然而，那并不是平滑的。如果你启动实例中的复选框（enable the checkbox in the [demo](http://thewebrocks.com/demos/smashing-events/mouseposition.html)），你将看到球是平滑移动的，这个我们使用库办到。不过如今浏览器能够做得越来越好了，我们需要做的只是在CCS中添加一个transition属性，然后浏览器就可以平滑将那个球从一个位置移到另一个位置。为了实现这个效果，我们定义一个名为smooth的css类。当文档中的复选框被点击选中时，把这个css类加到plot上。CSS代码：
	
	.smooth {
	  -webkit-transition: 0.5s;
	     -moz-transition: 0.5s;
	      -ms-transition: 0.5s;
	       -o-transition: 0.5s;
	          transition: 0.5s;
	}

JavaScript代码：

	var cb = document.querySelector('input[type=checkbox]');
	cb.addEventListener('click', function(ev) {
	  plot.classList.toggle('smooth');
	}, false);

CSS和JavaScript事件之间的交互总是非常强大的，但是在新浏览器中他们的交互甚至更好。正如你已经猜到的，CSS的transitions和animations已经有了他们自己的事件

### 一个键按下要多久？

正如你之前在可用的事件列表中看到的那样，浏览器也给我们对键盘输入进行响应的机会并且告诉我们什么时候用户已经按下了一个键。遗憾的是，浏览器中的键入处理是很难处理好的，正如Jan Wolter在[JavaScript Madness: Keyboard Events](http://unixpapa.com/js/key.html)一文中解释的那样。不管怎样，作为一个简单的实例，让我们看下我们怎么测出一个用户按下一个按钮需要多少毫秒。看看[“Calculating Keypress time”](http://thewebrocks.com/demos/smashing-events/keytime.html)这个实例。按下键盘上任意一个键，你将看到当键按时，输出框区域在增长。一旦你释放按键时，你就会看到你按下按键持续的毫秒数。代码非常简单的：
	
	var resources = document.querySelector('#resources'),
	    log = document.querySelector('#log'),
	    time = 0;
	 
	document.addEventListener('keydown', keydown, false);
	document.addEventListener('keyup', keyup, false);
	 
	function keydown(ev) {
	  if (time === 0) { 
	    time = ev.timeStamp; 
	    log.classList.add('animate');
	  }
	}
	function keyup(ev) {
	  if (time !== 0) {
	    log.innerHTML = ev.timeStamp - time;
	    time = 0;
	   log.classList.remove('animate');
	  }
	}

我们定义一个我们想要的元素并且将time设置为0。然后我们给文档应用两个事件处理程序，一个对keydown事件，另一个对keyup事件。

在keydown处理程序中，我们检查time是否为0，如果是，我们就设置time为事件的timeStamp值。我们分配一个CCS类给输出元素，此元素将开始一个CSS动画（看CSS是怎么实现的）。

keyup处理程序检查time是否仍然是0（如当键盘被按下时，keydown被连续触发），如果不是，处理程序将计算在时间戳上的区别，同时把time设回到0并且移除CSS类从而停止动画。

### 使用CSS transitions(和动画)

CSS transitions会触发一个单独的叫做Transitioned的事件，监听JavaScript中的transitionend。这个事件对象有两个属性：一个是propertyName，它包含已经被过渡的属性，另一个是elapsedTime，它告诉你过渡花了多长时间？

看看运行中的实例（[the demo](http://thewebrocks.com/demos/smashing-events/transitionevent.html)）。代码非常简单,下面是CSS代码
	
	.plot {
	  background:rgb(175,50,50);
	  width: 20px;
	  height: 20px;
	  border-radius: 20px;
	  display: block;
	  -webkit-transition: 0.5s;
	    -moz-transition: 0.5s;
	      -ms-transition: 0.5s;
	       -o-transition: 0.5s;
	          transition: 0.5s;
	}
	.plot:hover {
	  width: 50px;
	  height: 50px;
	  border-radius: 100px;
	  background: blue;
	}

JavaScript代码：

	plot.addEventListener('transitionend', function(ev) {
	  log.innerHTML += ev.propertyName + ':' + ev.elapsedTime + 's ';
	}, false);

然而，这个代码现在只能在Firefox上很好的工作，因为Chrome、Safari和Opera浏览器都有带前缀的事件代替。正如David Calhoun的要点一文（David Calhoun’s gist）所展示的那样，你需要检测浏览器支持什么并且像它支持的那样定义事件的名称。

CSS动画事件（[CSS animation events](https://developer.mozilla.org/en/CSS/CSS_animations)）以同样的方式工作，但是你有三个事件而不是一个：animationstart、animationend和animationiteration。MDN有一个关于它的实例（[MDN has a demo](https://developer.mozilla.org/samples/cssref/animations/animevents.html)）

### 速度，距离和角度

检测事件的发生是一件事，如果你想根据这些事件做一些漂亮的令人着迷的事，你还需要进一步并且加入一些数学进去。所以，让我们试试使用一些鼠标处理程序去计算当用户在屏幕上拖动元素时鼠标移动的角度、距离和速度。[首先看看实例](http://thewebrocks.com/demos/smashing-events/speeddistanceangle.html)。

	var plot = document.querySelector('.plot'),
	    log = document.querySelector('output'),
	   offset = plot.offsetWidth / 2,
	   pressed = false,
	   start = 0, x = 0, y = 0, end = 0, ex = 0, ey = 0, mx = 0, my = 0, 
	   duration = 0, dist = 0, angle = 0;
	
	document.addEventListener('mousedown', onmousedown, false);
	document.addEventListener('mouseup', onmouseup, false);
	document.addEventListener('mousemove', onmousemove, false);
	 
	function onmousedown(ev) {
	 if (start === 0 && x === 0 && y === 0) {
	    start = ev.timeStamp;
	    x = ev.clientX;
	    y = ev.clientY;
	    moveplot(x, y);
	    pressed = true;
	  }
	}
	function onmouseup(ev) {
	  end = ev.timeStamp;
	  duration = end - start;
	  ex = ev.clientX;
	  ey = ev.clientY;
	  mx = ex - x;
	  my = ey - y;
	  dist = Math.sqrt(mx * mx + my * my);
	  start = x = y = 0;
	  pressed = false;
	  angle = Math.atan2( my, mx ) * 180 / Math.PI;
	  log.innerHTML = '' + (dist>>0) +' pixels in '+
	              duration +' ms ( ' +
	                 twofloat(dist/duration) +' pixels/ms)'+
	                  ' at ' + twofloat(angle) +
	                  ' degrees';
	}
	function onmousemove (ev) {
	  if (pressed) {
	    moveplot(ev.pageX, ev.pageY);
	  }
	}
	function twofloat(val) {
	  return Math.round((val*100))/100;
	}
	function moveplot(x, y) {
	  plot.style.left = (x - offset) + 'px';
	  plot.style.top = (y - offset) + 'px';
	}

当然，我承认：很多是在这里发生的，但是它并不像看上去那么难。对于onmousedown和onmouseup两者来说，我们通过clientX及clientY读到鼠标的位置并且读到事件的timeStamp。鼠标事件有时间戳，这能告诉你事件在什么时候发生。当鼠标移动的时候，我们所要检查的是鼠标按钮是否被按下了（通过一个mousedown处理程序中的布尔设置）并且用鼠标移动plot区块。

剩余的是几何问题——优秀古老的毕达哥拉斯定理（勾股定理Pythagoras），将更加准确。我们通过检测mousedown和mouseup两者的时间差中所走过的像素数获得移动的速度。

我们获得鼠标所走过的像素数作为x轴和y轴上运动起止点之间像素差的平方和的平方根。并且我们通过计算三角形的反正切获得了角度。所有的这些都被包含在”[A Quick Look Into the Math of Animations With JavaScript](http://coding.smashingmagazine.com/2011/10/04/quick-look-math-animations-javascript/)”一文中；或者你可以练习下面的JSFiddle实例：

### 媒体事件

视频和音频两者都可以触发许多我们可以开发的事件。最有趣的是时间事件，它可以告诉你你的视频或音频已经播放了多久。有一个小巧的实例，你可以在MDN中的[MGM-inspired dinosaur animation](http://hacks.mozilla.org/2012/03/making-the-dino-roar-syncing-audio-and-css-transitions/)看到；我记录的6分钟的[视频](file:///E:/studynote/studynote/www.youtube.com/watch?v=aFIJ_ZpV-8Q)中解释它是怎么实现的。

如果你想看所有运行中的事件的实例，JPlayer团队有一个非常棒的实例页面可以展示诸多的媒体事件（[demo page showing media events](http://www.jplayer.org/HTML5.Media.Event.Inspector/)）。

### 输入框选项

传统上，浏览器给我们提供了鼠标和键盘交互。如今，这是不够的，因为我们使用的硬件设备可以给我们提供更多。比如说，设备方向感应（[Device orientation](https://developer.mozilla.org/en/DOM/DeviceOrientationEvent)）允许你对手机或平板的倾斜作出响应；触摸事件(touch events)是移动设备和平板上的大发明；游戏手柄API([the Gamepad API)](http://hacks.mozilla.org/2011/12/paving-the-way-for-open-games-on-the-web-with-the-gamepad-and-mouse-lock-apis/)允许我们读取浏览器中的游戏控制信息；邮寄消息（[postMessage](https://developer.mozilla.org/en/DOM/window.postMessage)）允许我们通过跨域和浏览器窗口发送消息；页面可视性（[pageVisibility](https://developer.mozilla.org/en/DOM/Using_the_Page_Visibility_API)）允许我们对用户做出响应可以切换到另一个标签页。我们甚至能检测什么时候窗口的历史对象已经被控制了（[when the history object of the window has been manipulated](https://developer.mozilla.org/en/DOM/window.onpopstate)）。检查窗口对象中的事件列表去发现某些更多的好东西可能不能，但是随着我们不断得深入挖掘是会实现的。

不管在浏览器支持下以后会出现什么，你应该相信事件将被鼓励和发展并且你将能够监听他们。这种方法有效并且确实流行。

### 开始行动吧

就是这样。事件并不难；在大多数情况下，你仅仅需要订阅他们并且检查返回的事件对象是什么，同时看怎么处理它。当然，大多的浏览器攻击有时还是需要的，但是我对于其中之一发现了难以置信可以和用户交互和看他们正在做什么的方式。如果你想在这方面有所创新，就停止思考我们现在的用例并且深入研究这些细节——对于交互界面来说，距离、角度、速度和输入意味着什么。如果你思考它，那么最大程度地玩风怒小鸟意味着检测触摸事件的开始和结束并且检测电量以及鸟应该起飞的方向。所以，是什么阻碍了你创造极具交互性和一流的事物？