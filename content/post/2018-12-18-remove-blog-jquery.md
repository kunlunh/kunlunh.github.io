---
author: HKL
categories:
- 默认分类
cid: 63
date: "2018-12-18T04:47:00Z"
slug: remove-blog-jquery
status: publish
tags:
- Coding
- Blog
title: 移除Blog对jQuery的依赖
updated: 2019/01/29 16:20:26
---


1.由于博客其实动态功能并不多，而且很多都是多年前完全不懂前端的情况下写的，所以有些功能没有考虑好，现在再看了一下前端代码部分，发现很多DOM操作已经完全没有必要去用jQuery了，以后再加新功能也不会用到jQuery的特性，所以计划改写jQuery部分为原生javascript。
   
2.逐步改写

(1)部分插件的改用

主要是博客使用了Bootstrap的框架，所以官方的Bootstrap部分功能是依赖jQuery的，这个直接替换成[Bootstrap.native](https://thednp.github.io/bootstrap.native/v4.html "Bootstrap.native")了

另外一个就是timeago的实现替换了jquery.timeago为使用原生js的[timeago](https://timeago.org "timeago")

(2)DOM操作部分

原来博客主要通过jQuery做了许多DOM操作，例如Query Selector，这部分参考了[You-Dont-Need-jQuery](https://github.com/nefe/You-Dont-Need-jQuery)进行改写

(3)特殊部分

有一些jQuery特有的方法通过事件绑定重新实现了功能

<!--more-->

3.The code

(1)timeago binding

`former`
```javascript
timeago().render($('time.timeago'),'zh_CN');
```
`now`
```javascript
timeago().render(document.getElementsByClassName('timeago'),'zh_CN');
```

(2)click toggle

`former`
```javascript
$(document).ready(function() {
     $(".page").children().first().find(".page-item-body").css( "display", "block" );
     $(".page").children().first().find(".pib-indicator")
         .find("i").toggleClass("iconfont icon-shrink").toggleClass("iconfont icon-expand");
 
     $(".page-item .pib-indicator").click(function() {
         var preview = $(this);
         preview.find("i").toggleClass("iconfont icon-expand").toggleClass("iconfont icon-shrink");
         $(this).closest(".page-item").find(".page-item-body").toggle("fast", function() {});
     });
 });
```
`now`
```javascript
var superToggle = function(element, class0, class1) {
  element.classList.toggle(class0);
  element.classList.toggle(class1);
}
document.addEventListener("DOMContentLoaded", function(event) { 
	var pib = document.getElementsByClassName('pib-indicator');
    document.getElementsByClassName('page-item-body')[0].style.display = 'block';
	superToggle(pib[0].querySelectorAll('i')[0],'icon-shrink', 'icon-expand');
	var count = 0;
	for (count ; count < pib.length; count++) {
		pib[count].addEventListener("click", function() {
			superToggle(this.querySelectorAll('i')[0],'icon-expand', 'icon-shrink');
			var el = this.closest('.page-item').querySelectorAll('.page-item-body')[0];
			if (el.style.display === "none" || el.style.display == "") {
				el.style.display = "block";
			  } else {
				el.style.display = "none";
			  }	
		});
	}
	//closest只支持非IE浏览器
});
```
`former`
```javascript
$(document).ready(function() {
    $('.menu-btn').click(function() {
        $('.row-offcanvas').toggleClass('active');
    });
    $('.close-btn').click(function() {
        $('.row-offcanvas').toggleClass('active');
    });
});
```
`now`
```javascript
document.addEventListener("DOMContentLoaded", function(event) { 
    document.getElementsByClassName('menu-btn')[0].addEventListener("click", function() {
		document.getElementsByClassName('row-offcanvas')[0].classList.toggle('active');
	});
    document.getElementsByClassName('close-btn')[0].addEventListener("click", function() {
		document.getElementsByClassName('row-offcanvas')[0].classList.toggle('active');
	});
});
```

(3)search engine

`former`
```javascript
$("#tet").click(function(){
  var html = '';
  $.ajax({
	  url: '../index.xml',
      dataType: 'xml',
	  success: function (data) {
		var count = 0;
		html = '<table class="table table-bordered"><thead><tr><th scope="col" class="mb-2">#</th><th scope="col" class="mb-2">标题</th><th scope="col" class="mb-5">链接</th></tr></thead><tbody>';
		var keyword = $('#keyword').val().toLowerCase();
		if (keyword == ""){
			html = '<h2>No Input!</h2>'
		}else{
		   $(data).find('item').each(function () {
           var $this = $(this);
		   var judge = $this.find('title').text().toLowerCase();
           var key = $this.find('title').text();
		   if(judge.indexOf(keyword)!=-1){
             $this.find('title').each(function () {
				html += '<tr><th scope="row">' + count + '</th><td>' + key + '</td><td><a href="' + $this.find('link').text() + '">' + $this.find('link').text() + '</a></td></tr>';
			   });	
			  count = count + 1;
		     }		   
			});
			html += '</tbody></table>';
			if(count == 0){
				html = '<h2>Not Found!</h2>'
			}        
	    }
		$('#search').html('查询结果');	
	    $('#result').html(html);
		},      
		error: function () {
		$('#search').html('查询结果');	
        $('#result').html('连接失败');
     }

  });
});
```
`now`
```javascript
document.addEventListener("DOMContentLoaded", function(event) { 
	document.getElementById('tet').addEventListener("click", function() {
			loadXMLDoc('/index.xml');
	});
});

function loadXMLDoc(url)
{
	
var xmlhttp;
var html = '';
var x,xx,i;
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
  xmlhttp.onreadystatechange=function()
  {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
		console.log('xml');
	var count = 0;
	html = '<table class="table table-bordered"><thead><tr><th scope="col" class="mb-2">#</th><th scope="col" class="mb-2">标题</th><th scope="col" class="mb-5">链接</th></tr></thead><tbody>';
	//var keyword = $('#keyword').val().toLowerCase();
	var keyword = document.getElementById('keyword').value.toLowerCase();
	if (keyword == ""){
			html = '<h2>No Input!</h2>';
	}else{
		x=xmlhttp.responseXML.documentElement.getElementsByTagName("item");
		for (i=0;i<x.length;i++)
		  {
	      //var judge = x[i].getElementsByTagName("title").toLowerCase();
		  //var key = x[i].getElementsByTagName("title");
		  xx=x[i].getElementsByTagName("title");
		  {
			try
			  {
			  var judge = xx[0].firstChild.nodeValue.toLowerCase();
			  var key = xx[0].firstChild.nodeValue;
			  var posturl = x[i].getElementsByTagName("link")[0].firstChild.nodeValue;
			  }
			catch (er)
			  {
				console.log(er);  
			  continue;
			  }
			}
		  if(judge.indexOf(keyword)!=-1){
			  html += '<tr><th scope="row">' + count + '</th><td>' + key + '</td><td><a href="' + posturl + '">' + posturl + '</a></td></tr>';
			  count = count + 1;
		     }
		  }
		html += '</tbody></table>';
		if(count == 0){
			html = '<h2>Not Found!</h2>'
		}
		
		}
		document.getElementById('result').innerHTML=html;
	}else{
		document.getElementById('result').innerHTML='No connection';
	}
  }
xmlhttp.open("GET",url,true);
xmlhttp.send();
}
```

(4)window对象的绑定事件

`former`
```javascript
$(window).bind('scroll', (function() {
    clearTimeout(timer);
    timer = setTimeout(getPercent, 5);
    isTop();
}));
```
`now`
```javascript
window.addEventListener("scroll",function(){
	clearTimeout(timer);
	timer = setTimeout(getPercent, 5);
	isTop();
});
```

(5)fadein and fadeout

`former`
```javascript
function isTop() {
  if(scrollPossition() > 30)
    $(".goto-top").fadeIn("slow");
  else
    $(".goto-top").fadeOut("slow");
}
```
`now`
```javascript
function isTop() {
  if(scrollPossition() > 30) {
    document.getElementsByClassName('goto-top')[0].style.transition = 'opacity 3s';
	document.getElementsByClassName('goto-top')[0].style.opacity = '1';
  }else{
    document.getElementsByClassName('goto-top')[0].style.transition = 'opacity 3s';
	document.getElementsByClassName('goto-top')[0].style.opacity = '0';
  }
}
```

(6)$(window).scrollTop() and $(document)

`former`
```javascript
function scrollPossition() {
    return ($(window).scrollTop() / ($(document).height() - $(window).height())) * 100;
}
```
`now`
```javascript
function scrollPossition() {
	var window_scrollTop =   (document.documentElement && document.documentElement.scrollTop) || document.body.scrollTop;
    const body = document.body;
	const html = document.documentElement;
	var document_height = Math.max(
	  body.offsetHeight,
	  body.scrollHeight,
	  html.clientHeight,
	  html.offsetHeight,
	  html.scrollHeight
	);
	var window_height = window.innerHeight;
	return ( window_scrollTop / ( document_height - window_height )) * 100;
}
```

(7)CSS style

`former`
```javascript
function getPercent() {
    $(".statusbar").css("width", String(scrollPossition()) + "%");
    if (scrollPossition() > 97) {
        $(".statusbar").css("background-color", "#1ca265");
    } else {
        $(".statusbar").css("background-color", "#eb5e29");
    }
    return;
}
```
`now`
```javascript
function getPercent() {
	document.getElementsByClassName('statusbar')[0].style.width = String(scrollPossition()) + "%";
    if (scrollPossition() > 97) {
        document.getElementsByClassName('statusbar')[0].style.background = "#1ca265" ;	
    } else {
        document.getElementsByClassName('statusbar')[0].style.background = "#eb5e29" ;
    }
    return;
}
```


4.总结

目前的原生javascript已经足够优秀（ IE除外 :) ）,很多原来必须使用到jQuery的场合也能找到相应的替代方案，由于这次改写只是用在自己博客上，很多地方应该可以更加严谨地用代码，这个就留在以后在解决，这次改写仅仅解决能用的问题啦。（It just works.）