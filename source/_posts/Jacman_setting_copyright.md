---
title: Jacman-master主题设置 - 自动添加版权声明
date: 2016-05-13 19:21
tags: [Hexo,Jacman]
show_copyright : true
---
虽然这里不会有太多人来看，文章大多数也是自己的个人看法，鉴于自己的阅历和技术水平，难免会有纰漏。
但是每一篇原创文章自己都有花心思在上面，所以我会在每篇原创的文章最后添加`<<<<<原创 装载请注明出处>>>>>`的信息(虽然到现在也没发现被别人转载过什么T_T)

鉴于每次POST文章都要手动添加那一行，这个操作感觉有点反人类，所以决定添加自动在文章底部添加版权信息的功能。
这里要感谢培豪的文章 - [Hexo - 文章版权追加](http://www.cnblogs.com/peihao/p/5365733.html)，里面为我提供了漂亮的css模板和处理的方法。
但是可能是因为使用的主题有差(我用的是Jacman-master主题)，所以还要做部分的修改。

## 创建版权信息的内容
在`/Jacman-master/layout/_partial/post/`下添加要show出来的版权信息，这里我们创建`auto_copyright.ejs`，内容如下
```html
<div class="article-footer-copyright">
  <center>本文由<b><a href="<%= config.root %>about/" target="_blank" title="<%= config.author %>"><%= config.author %></a></b>原创,采用<b>BY</b>-<b>NC</b>-<b>SA</b>国际许可协议进行许可</center>

  <center>转载请注明作者及出处,本文作者为<b><a href="<%= config.root %>about/" target="_blank" title="<%= config.author %>"><%= config.author %></a></b></center>

  <center>本文链接为<b><a href="<%- config.root %><%- item.path %>" target="_blank" title="<%= item.title %>"><%- config.url %>/<%- item.path %></a></b>.</center>
</div>
```
鉴于主题的不同，里面的参数部分会有差异。
显示的内容可根据自己需求自行更改。

## 创建版权信息对应的样式表
在`/Jacman-master/source/css/_partial`下面，创建 `copyright.styl`。css只是看的懂，写嘛就算了，只好`拿来主义`了。这里完全借鉴了培豪的样式表，如下：
```css
.article-footer-copyright {
  border-top: 1px solid #d3d3d3;
  margin: 10px auto;
  padding-left: 2em;
  width: 80%;
}
.article-footer-copyright span,
.copyright abbr {
  color: #3d3d3d;
}
div.copyright {
  font-weight: bold;
  color: #fcb297;
  padding: 0.3em 0.5em;
  margin: 0 0 1em 0;
  border-bottom: none;
  background-color: #74a474;
  -moz-border-radius: 1em;
  -webkit-border-radius: 1em;
  -webkit-border-radius: 1em;
  border-radius: 1em;
  -moz-box-shadow: inset 0px 1px 0px 0px #eee;
  -webkit-box-shadow: inset 0px 1px 0px 0px #eee;
  -webkit-box-shadow: inset 0px 1px 0px 0px #eee;
  box-shadow: inset 0px 1px 0px 0px #eee;
  background: -webkit-gradient(linear, left top, left bottom, color-stop(0.05, #aad2f0), color-stop(1, #8bc1ed));
  background: -webkit-gradient(linear, left top, left bottom, color-stop(0.05, #aad2f0), color-stop(1, #8bc1ed));
  background: -webkit-gradient(linear, left top, left bottom, color-stop(0.05, #aad2f0), color-stop(1, #8bc1ed));
  background: -webkit-gradient(linear, left top, left bottom, color-stop(0.05, #aad2f0), color-stop(1, #8bc1ed));
  background: -moz--webkit-linear-gradient(center top, #aad2f0 5%, #8bc1ed 100%);
  background: -moz--moz-linear-gradient(center top, #aad2f0 5%, #8bc1ed 100%);
  background: -moz--ms-linear-gradient(center top, #aad2f0 5%, #8bc1ed 100%);
  background: -moz-linear-gradient(center top, #aad2f0 5%, #8bc1ed 100%);
/* filter:progid:DXImageTransform.Microsoft.gradient(startColorstr='#aad2f0', endColorstr='#8bc1ed'); */
  background-color: #74a474;
  border: 1px solid #dcdcdc;
  text-shadow: 1px 1px 0px #eee;
}
div.article-footer-copyright {
  margin-top: 2em;
  padding: 0.8em;
  border: 1px solid #d3d3d3;
  background-color: #ffffcc;
}
.article-footer-copyright p {
  line-height: 140%;
  margin: 10px;
  font-size: 100%;
}
```
需要注意的是样式表需要导入，不然前面设置的版权信息没有办法引用css中的article-footer-copyright类。
导入须在`/Jacman-master/source/css/style.style`里面添加：
```
@import '_partial/copyright'
```

## 布局页面版权信息
前面两步将要插入的版权信息写好，接下来就是要放进我们的页面中了。看个人口味，我是将信息放在文章最后，也就是在`item.content`的后面导入`auto_copyright`。当然放哪里完全是个人想法，你就是插在文章中间也没人说你什么。
```html
<div id="main" class="<%= item.layout %>" itemscope itemprop="blogPost">
	......<省略>......
		<%- item.content %>  
	</div>
		<%- partial('auto_copyright') %>
		<%- partial('footer') %>   	       
	</article>
	<%- partial('pagination') %>
	<%- partial('comment') %>
</div>  
```
可是运行之后你会发现，这个信息在每一个页面都有显示，这就不好了，况且我还有一些转载的文章，总不能厚颜无耻的说是自己原创吧。
所以需要给显示设定一个开关，该显示的时候显示，不该显示的时候就不要显示。
这里我需要在文章中设置一个 `show_copyright`的 flag，为 true 的时候显示，如：
```
title: Jacman-master主题设置 - 自动添加版权声明
date: 2016-05-13 19:21
tags: [Hexo,Jacman]
show_copyright : true
```
在 article.ejs 中我们对这个 flag 和是否是主页做判断，flag 为 true 且不是主页的情况下再显示 `copyright`信息。完整代码如下：
```html
<div id="main" class="<%= item.layout %>" itemscope itemprop="blogPost">
  <% if (page.layout=='photo' && item.photos && item.photos.length){ %>
    <%- partial('gallery') %>
  <% } %>

	<article itemprop="articleBody"> 
		<%- partial('header') %>
	<div class="article-content">
		<% if( table&&(item.toc !== false) && theme.toc.article){ %>
		<div id="toc" class="toc-article">
			<strong class="toc-title"><%= __('contents') %></strong>
		<% if(item.list_number == false) {%>
		<%- toc(item.content,{list_number:false}) %>
		<% }else{ %>
			<%- toc(item.content) %>
		<% } %>
		</div>
		<% } %>
		<%- item.content %>  
	</div>
		<% if (!index && item.show_copyright == true) { %>
			<%- partial('auto_copyright') %>
		<% } %>
		<%- partial('footer') %>   	       
	</article>
	<%- partial('pagination') %>
	<%- partial('comment') %>
</div>  
```
这样在我们原创的文章前面设置 `show_copyright` 为 `true` 时就可以显示 `copyright` 信息啦！！具体效果往下翻翻你就看到啦！

That's all, Thank you.
