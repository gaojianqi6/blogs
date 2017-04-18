---
title: IE下Placeholder的兼容处理
date: 2016-07-08 15:49:53
tags: [javascript, 前端] 
categories: [技术]
---
## placeholder
placeholder是html5新增的一个属性，当input或者textarea设置了这个属性之后，placeholder的值会作为灰字提示显示在该输入框上，当文本框获取焦点之后或者用户输入一个字符之后，则提示文字消失。这个属性在IE 10中对他的支持才好一点，而用户还有好多使用IE7、8、9的，所以还是需要在旧的浏览器中对他进行一下处理。
## 解决办法
网上对placeholder的解决办法大多数是对placeholder进行判断，在不支持placeholder属性的浏览器中，检测blur和focus事件，直接把placeholder的值赋在该元素的value上，如：


```javascript
function placeholder() {
	if (/MSIE\s(\d+)/.test(navigator.userAgent) && navigator.userAgent.match(/MSIE\s(\d+)/)[1] < 10) {
		$('[placeholder]').each(function() {
			var pla = $(this).attr('placeholder');
			$(this).focus(function() {
				if ($(this).val() == pla) {
					$(this).val('');
				}
			}).blur(function() {
				if ($(this).val() == '') {
					$(this).val(pla);
				}
			});
			$(this).trigger('blur'); //此处利用blur事件,为了业务需求。防止页面上查询后，文本框需要保留搜索的关键被placeholder代替
		});
	}
}

$(document).ready(function() {
	placeholder();
});
```

或者一些公用库对他也有很好的实现，如[Placeholders.js](https://jamesallardice.github.io/Placeholders.js/)。

但是，如果在项目中用到了验证表单的库，像[jquery.validation](https://jqueryvalidation.org/)这样的，也是对输入框的值进行了验证，那么在使用Placeholder的时候会有一些问题，比如发现值是错的对他改变了颜色之类的。

## 最终的解决办法
所以后来就决定在输入框后面加一个span元素，当页面dom加载完毕之后，对所有符合条件的input、textarea进行处理，默认把他们的placeholder赋值在后面的span元素上面，点击的时候隐藏提示，离开的时候判断输入框是否有值来对他进行显示。
代码中采用了[Placeholders.js](https://jamesallardice.github.io/Placeholders.js/)的一些思路，使用createTextNode来添加css，这样只用引进一个js文件就有默认的placeholder样式。

检查的时候会对display为none、type等于hidden、class为hidePassword和placehloder-check为false的直接跳过，不进行处理。

使用的时候需要提前引用jQuery。
### 写的时候遇到的问题
jQuery在父元素display为none时，计算width，会加上padding-left 和padding-right,而不去管box-sizing 是否为border-box，发现这个问题之后在代码中对当前元素进行了判断，如果是hidden的话，则使用width()方法获取宽度，不是的话，则全使用css("width")获取宽度。
判断方法:

```javascript
 .is(":visible")  .is(":hidden"))
```

项目地址：[https://github.com/gaojianqi6/placeholder.jquery.js](https://github.com/gaojianqi6/placeholder.jquery.js)
