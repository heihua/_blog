---
layout: post
title: ControlJS简介
---


* 作者： Steve Souder
* 发布时间：2010年10月

ControlJS：一个让脚本加更快的Javascript模块（a javascript module for making scripts load faster）

### ControlJS相关文章

* [ControlJS part 1: async loading](http://www.stevesouders.com/blog/2010/12/15/controljs-part-1/)
中文翻译：[http://ued.ctrip.com/blog/?p=2964](http://ued.ctrip.com/blog/?p=2964)
* [ControlJS part 2: delayed execution](http://www.stevesouders.com/blog/2010/12/15/controljs-part-2/)
中文翻译：[http://ued.ctrip.com/blog/?p=3017](http://ued.ctrip.com/blog/?p=3017)
* [ControlJS part 3: overriding document.write](http://www.stevesouders.com/blog/2010/12/15/controljs-part-3/)
中文翻译：[http://ued.ctrip.com/blog/?p=3017](http://ued.ctrip.com/blog/?p=3111)


### ControlJS例子

* [没有使用ControlJS](http://stevesouders.com/controljs/examples/async-baseline.php?t=1352557824)
* [使用了ControlJS](http://stevesouders.com/controljs/examples/async.php?t=1352557824)
* [没有使用ControlJS的菜单](http://stevesouders.com/controljs/examples/async.php?t=1352557824)
* [使用了ControlJS的菜单](http://stevesouders.com/controljs/examples/menu.php?t=1352557824)
* [没有使用ControlJS的document.write](http://stevesouders.com/controljs/examples/docwrite-baseline.php?t=1352557824)
* [使用了ControlJS的document.write](http://stevesouders.com/controljs/examples/docwrite-baseline.php?t=1352557824)

### ControlJS的特点：

* 1、异步加载HTML中的Javascript
* 2、可以处理内联script和外联script
* 3、延迟执行script直到页面全部渲染完
* 4、允许script加载但却不执行
* 5、结合了HTML标记的属性修改-不需要修改代码
* 6、能够解决一些document.write引发的问题
* 7、control.js它本身也是异步加载的。

