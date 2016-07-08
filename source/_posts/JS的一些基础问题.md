---
title: JS的一些基础问题
date: 2016-05-26 14:35:37
tags: [javascript]
categories: [技术]
---

### 1、如果给定一个字符串，怎么返回成一个JS的Dom对象。 比如下面的这个字符串：

```javascript
<div>测试<span>cd</span></div>
```

这个要用到浏览器的innerHTML来实现了，首先使用createElement来创建一个div元素，然后使用innerHTML把字符串设置为它的子元素，然后再返回他的子元素就可以了。实现例子：

```javascript
function parseDom(str){
	var objParent = document.createElement("div");
	objParent.innerHTML = str;
	return objParent.childNodes;
}

parseDom("<div>测试<span>cd</span></div>");
```
那这里面的childNodes返回的是一个数组，如果确定传入的是一个元素，则使用下标取第一个就行，parseDom('&lt;div&gt;&lt;/div&gt;')[0]

### 2、给一个父元素，怎么遍历下面的所有子元素。
```html
<div id="ID">
    <div id="1">内容1</div>

    <div id="2">
        <div id="2_1">内容2.1</div>
        <div id="2_1">内容2.1</div>    
    </div>
    <div id="3">内容3</div>
</div>
```


使用childNodes获取元素的子节点，但是在FF和Chrome下面换行符也会当成一个元素储存在元素列表里，所以根据元素的nodeName来判断，如果是的话，则删除当前元素。
```javascript
function getChildNodes(obj){
	var nodes = obj.childNodes,
		objNode,
		i = 0;
	for(; i < nodes.length; i++){
		objNode = nodes[i];
		//当元素里面有节点类型是文本并且文本类型节点的节点值是空的。就把他删除。
		//nodeNames可以得到一个节点的节点类型，nodeValue表示得到这个节点里的值
		// /\s/是非空字符在JS里的正则表达式,如：
		// /\s/.test(" ") //true
		// /\s/.test("sdf") //false
		if(objNode.nodeName == "#text" && /\s/.test(objNode.nodeValue)){
			obj.removeChild(objNode);
		}
	}
	return obj.childNodes;
}

getChildNodes(document.getElementById("ID"));
```
### 3、给一个冒泡排序。
[【轻松学排序算法】眼睛直观感受几种常用排序算法](http://www.cnblogs.com/wangfupeng1988/archive/2011/12/26/2302216.html)
[js实现数组冒泡排序、快速排序原理](http://www.52codes.net/article/2331.html)
[Javascript算法系列之快速排序（Quicksort）](http://www.cnblogs.com/aaronjs/p/3539538.html)
### 4、retina屏幕如何实现真正1px的边框线
解决办法:
1、判断浏览器是否支持0.5PX的线，支持给html元素添加一个class
```css
.hairlines div {   border-width: 0.5px; }
```
```javascript
if (window.devicePixelRatio && devicePixelRatio >= 2) {
  var testElem = document.createElement('div');
  testElem.style.border = '.5px solid transparent';
  document.body.appendChild(testElem);
  if (testElem.offsetHeight == 1)
  {
    document.querySelector('html').classList.add('hairlines');
  }
  document.body.removeChild(testElem);
}
// 脚本应该放在 <body> 内， 如果在 <head> 里面运行，需要包装 $(document).ready(function() {   })
```
不过这个兼容不了安卓设备，而且ios 8以下不支持，所以还是得安卓支持才能用。。
2、使用图片完成border
一个6*6的图片
![切图](http://o7bfyflfi.bkt.clouddn.com/image/blogs/border-bg.png)

```css
.border{     border-width: 1px;     border-image: url(border.gif) 2 repeat; }
```
缺点是每次改变颜色都要重新切图。
3、用多背景渐变实现的，设置 1px 的渐变背景，50% 有颜色，50% 透明
``` css
.border {     background:     linear-gradient(180deg, black, black 50%, transparent 50%) top    left  / 100% 1px no-repeat,     linear-gradient(90deg,  black, black 50%, transparent 50%) top    right / 1px 100% no-repeat,     linear-gradient(0,      black, black 50%, transparent 50%) bottom right / 100% 1px no-repeat,     linear-gradient(-90deg, black, black 50%, transparent 50%) bottom left  / 1px 100% no-repeat; }
```
圆角没法实现，而且代码比较长
4、通过 viewport + rem 实现的
当 devicePixelRatio = 2 时，输出 viewport
```html
<meta name="viewport" content="initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no">
```
当devicePixelRatio = 3时，输出 viewport
```html
<meta name="viewport" content="initial-scale=0.3333333333333333, maximum-scale=0.3333333333333333, minimum-scale=0.3333333333333333, user-scalable=no">
```
同时通过设置对应 viewport 的 rem 基准值，这种方式就可以像以前一样写 1px 了。
5、伪类 + transform
原理是把原先元素的 border 去掉，然后利用:before或者:after重做 border ，并 transform 的 scale 缩小一半，原先的元素相对定位，新做的 border 绝对定位
* 单条 border
```css
.hairlines li{     position: relative;     border:none; }
.hairlines li:after{     content: '';     position: absolute;     left: 0;     background: #000;     width: 100%;     height: 1px;     -webkit-transform: scaleY(0.5);             transform: scaleY(0.5);     -webkit-transform-origin: 0 0; 

```

* 四条 border
```css
.hairlines li{     position: relative;     margin-bottom: 20px;     border:none; }
.hairlines li:after{     content: '';     position: absolute;     top: 0;     left: 0;     border: 1px solid #000;     -webkit-box-sizing: border-box;     box-sizing: border-box;     width: 200%;     height: 200%;     -webkit-transform: scale(0.5);     transform: scale(0.5);     -webkit-transform-origin: left top;     transform-origin: left top; }
```
样式使用的时候，也要结合 JS 代码，判断是否 Retina 屏
```javascript
if(window.devicePixelRatio && devicePixelRatio >= 2){
    document.querySelector('ul').className = 'hairlines';
}
```
可以支持圆角，唯一的一点缺陷是td用不了。

### 5、点击延迟300ms
#### 问题由来 

这要追溯至 2007 年初。苹果公司在发布首款 iPhone 前夕，遇到一个问题：当时的网站都是为大屏幕设备所设计的。于是苹果的工程师们做了一些约定，应对 iPhone 这种小屏幕浏览桌面端站点的问题。 

这当中最出名的，当属双击缩放(double tap to zoom)，这也是会有上述 300 毫秒延迟的主要原因。双击缩放，顾名思义，即用手指在屏幕上快速点击两次，iOS 自带的 Safari 浏览器会将网页缩放至原始比例。

#### 解决办法
1、[FastClick](https://github.com/ftlabs/fastclick),监测touchend事件，通过Dom自定义事件触发一个模拟click事件，并且把浏览器300MS之后真正触发的click事件关掉；   
2、touch-action 这个现在只在chrome和IE 10+支持，就是说在css设置touch-action: none后，那么对应的元素在被点击之后，浏览器不会启动缩放操作；

### 6、事件穿透 pointer-event
CSS pointer-events
pointer-events：auto | none | visiblepainted | visiblefill | visiblestroke | visible | painted | fill | stroke | all
当设置为none的时候，元素永远不会成为鼠标事件的target，当然如果需要禁止点透的节点是父子关系，那么这个方法是行不通的。

1、同样 [FastClick](https://github.com/ftlabs/fastclick)可以解决；
2、用touchend代替tap事件并阻止掉touchend的默认行为preventDefault()
### 7、viewport作用。
[手机浏览器的viewport（视觉窗口）](http://www.wufangbo.com/mobile-browser-viewport/)
### 8、闭包的用途和原理。
闭包的场景：
* 函数内部可以访问到外部的变量或者对象
* 使用闭包可以在JavaScript中模拟块级作用域
* 闭包可以用于在对象中创建私有变量, 不污染外部环境
* 避免垃圾回收

最简形式, 还有jquery的ajax调用时候也是形成了一个闭包。：
```javascript
function a(i)
{
  return function(){
    return i+1
  }
}
```
这篇 [javascript中闭包的工作原理](http://www.frontopen.com/1702.html) 对闭包的工作原理讲解的到位。
### 9、面向对象方法的扩充。
一般使用prototype来扩充，具体的实现可以模仿jQ的构建方法，下面这一篇讲的比较好，逻辑清晰易懂。
[jQuery 2.0.3 源码分析core - 整体架构](http://www.cnblogs.com/aaronjs/p/3278578.html)