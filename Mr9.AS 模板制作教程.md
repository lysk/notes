---
title: Mr9.AS 模板制作教程 
tags: AS模板制作
grammar_cjkRuby: true
---


### 一、整体结构
从层次结构来说,一个CMS网站简单可以分为: 首页,分类页,内容页,搜索页 这四部分.

>* 首页,负责显示网站最近更新的内容,重点推荐的内容
>* 分类页,归类显示下级内容的列表
>* 内容页,显示具体内容
>* 搜索页,功能上同分类页,但是列表内容是由用户输入的查询条件动态生成

从页面结构上来说,一个页面分为变化部分和不变化部分.更具体点来说,分为 头身脚手 四个部位.

>* 头(header),不变化部分
>* 身(body),变化部分
>* 手(sidebar),有可能变化部分
>* 脚(footer),不变化部分

从文件类型上来说,一个页面上的元素,分为:HTML,图片,CSS样式,JS脚本

![页面结构](http://i.imgur.com/jTjAFea.png)

(典型的Wordpress类网站布局)

AS的标准模板文件结构如下:

> themename
    │  category.html分类页
    │  home.html首页
    │  post.html内容页
    │  search.html  搜索页
    │  _footer.html 模块:固定不动的脚
    │  _header.html 模块:固定不动的头
    │  _postlist.html   模块:显示文章列表
    │  _sidebar.html模块:显示侧栏
    └─assets
    │  jquery-1.9.1.min.jsJQUERY公共库
    │  style.cssCSS文件
    └─images模板用到的图片目录

一个最简单的模板,由至少8个文件组成. 即8个扩展名为html的文件. 其中4个页面,4个模块.
不管你的模板有多大或有多小,这8个文件必须存在.哪怕为空内容.

### 二、全局变量

一,全局数据结构,全局数据是指每一个页面都可以获取到的值,下面的代码,一般加在每个模板文件中.

    {{$group:=.group}}<!--当前网站的分组-->
    {{$site:=.site}}<!--当前网站-->
    {{$categories := .categories}}<!--当前网站的栏目列表-->
    {{$category := .category}}<!--当前所在栏目(如果有)-->
    {{$categoryid:=.categoryid}}<!--当前栏目的ID-->
    {{$post:=.post}}<!--当前所在的POST(如果有)-->
    {{$theme :=.theme}}<!--当前网站使用的模板名称-->

1) `$group`,`$site` 当前网站,及当前网站所属的分组.

![](http://i.imgur.com/VZSc00e.png)

(上图为全局变量 `$site` 和 `$group`的属性)

趁你没迷糊,先说说怎么用这个变量.

例子一:在模板文件 _header.html 中, 我们会写这样的代码:

```html
<a href="{{$site.GetUrl}}" title="{{$site.Title}}" rel="home">
  {{if $site.Logo}}{{html$site.Logo}}{{else}}{{$site.Title}}{{end}}
</a>
```
这里显示网站的LOGO,但是如果这个站没有上传图片LOGO,则以网站标题做LOGO.

例子二:
```html
{{if $site.ADTop}}
    {{html $site.ADTop}}
{{else}}
    {{html $group.ADTop}}
{{end}}
```
如果网站配置了顶部广告代码,则优先显示,否则显示分组的顶部广告代码.如果2者都没有那么就什么都不显示.
继续说变量.

2) 分类变量 `$category` 当前的栏目
当用户打开网站首页的时候,这个变量是空的. 但是当用户点击进入一个栏目的时候,这个变量就是当前所在的栏目数据.

![当前栏目](http://i.imgur.com/QfimeRT.png)

3) `$categories` 当前网站所有的栏目列表,这是一个数组对像.
比如说,我们要显示栏目列表的时候

```html
<ul id="menu-main-menu">
    <li><a title="Home" href="http://{{$site.Domain}}/">Home</a></li> <!--加一个固定的网站首页-->
    {{range $categories}} <!--循环栏目列表-->
        <li><a href="/{{.Slug}}/">{{.Name}}</a></li>
    {{end}}
 </ul>
```

4) `$post` 这个变量,只有内容页的时候才会有 ,也就是在 post.html里才有用 ,也就是当前用户正在看的是什么文章.

![当前文章](http://i.imgur.com/UlALbWV.png)

5) `$theme`和`$categoryid`
这个简单了,就是当前模板名称和当前栏目编号.比如说 “default”
这个变量的目的是为了方便 引用CSS或图片
比如说:
```html
<link rel='stylesheet' href='/themes/{{$theme}}/assets/style.css' type='text/css' media='all' />
```

###三、全局函数

```php
func GET_POSTS(siteid int64, categoryid interface{}, limit int, sortstr string) []*Post{
//指定条件获取文章列表
}
```
调用的例子如下:
```php
{{$posts := GET_POSTS $site.Id  $categoryid 10 "-views"}} 
<!--定义一个变量$post,并赋值为 当前站点,当前分类,10篇文章 ,按浏览次数倒排序 -->
<!--用人话说就是: 取10个当前站当前栏目被看的最多的文章-->
    {{range $posts}} <!--循环显示这10个文章-->
        <li>
            <a href="{{.GetUrl}}">{{.Title}}"></a>
        </li>
    {{end}} <!--结束循环-->
```
上面这个函数是可以自己组合来获取 最POPUP的,最新的指定数量的全站或栏目的文章 列表.
```php
func GET_CATEGORIES(siteid int64, limit int) []*Category {
    //获取指定个数的分类列表. 有时候你的网站定义了100个栏目,但是在页面的最顶部只显示有限个数的栏目. 然后你会在其它地方比如说页面边上或底部显示所有的栏目.嗯,这是个大站.
    //进一步解释,全局变量 $categories是这个站的所有栏目.可能有100个.  你可以用这个函数来取前n个.
}
```

'safehtml' 显示安全的html
'html' 显示一段html
'eqq' 等于
'neqq' 不等于

具体模板表达式及教程参考 : http://www.cnblogs.com/Pynix/p/4154630.html

###四、模块详解

`_header.html`
```html
{{$group:=.group}}
{{$site:=.site}}
{{$categories := .categories}}
{{$category := .category}}
{{$categoryid:= .categoryid}}
{{$post:=.post}}
{{$theme :=.theme}}
<!DOCTYPE html>
<head>
    <meta charset="UTF-8" />
    {{if $post}}<!-- 如果有$post变量,优先显示$post的标题 -->
        <title>{{$post.Title}}</title>
        <meta http-equiv="keywords" content="{{$post.Keyword}}" />
    {{else}}
        <title>{{$site.Title}} {{$category.Name}}</title><!-- 否则显示站点和分类标题 -->
    {{end}}

    <link rel='stylesheet' href='/themes/{{$theme}}/assets/style.css' type='text/css' media='all' /><!-- 调用当前模板下的CSS -->
    <script type='text/javascript' src='/themes/{{$theme}}/assets/jquery-1.9.1.min.js'></script><!-- 调用当前模板下的JS -->

    {{html $group.TXTHeader}}<!-- 显示组和站的头部附加文本 -->
    {{html $site.TXTHeader}}
</head>


<body>
    {{html $group.TXTBodyT}}<!-- 显示组和站的页面顶部代码 -->
    {{html $site.TXTBodyT}}
    <div class="main-wrap" >
        <header>
            <div class="title">
                <a href="{{$site.GetUrl}}" title="{{$site.Title}}" rel="home">
                {{if $site.Logo}}{{html $site.Logo}}{{else}}{{$site.Title}}{{end}}<!-- 如果有LOGO代码就显示,没有就显示站点标题 -->
                </a>
            </div>
            <div class="right">
                <div class="adwrap-widget">
                    {{if $site.ADTop}}<!-- 优先显示站点的顶部广告代码,没有就显示组的顶部广告代码 -->
                    {{html $site.ADTop}}
                    {{else}}
                    {{html $group.ADTop}}
                    {{end}}
                </div>
            </div>
        </header>
        <nav class="navigation">
            <div class="menu-main-menu-container">
                <ul id="menu-main-menu" class="menu">
                    <li class="menu-item menu-item-type-post_type menu-item-object-page"><a title="Home" href="http://{{$site.Domain}}/">Home</a></li><!-- 在导航中显示首页连接 -->
                    {{range $categories}}<!-- 循环并显示站点栏目导航 -->
                    <li class="menu-item menu-item-type-custom menu-item-object-custom "><a href="/{{.Slug}}/">{{.Name}}</a></li>
                    {{end}}
                </ul>
            </div>
        </nav>
    </div>
    {{if .location}}<!-- 如果有当前位置的数据则显示用户当前所在位置,首页没有所以不显示这一块. -->
    <div class="wrap">
        <div class="breadcrumbs">
            <span class="location">You are at:</span><span itemscope="">
            <a itemprop="url" href="{{$site.GetUrl}}">
            <span itemprop="title">Home</span>
            </a>
            </span>
            <span class="delim">»</span>
            <span class="current">{{.location}}</span>
        </div>
    </div>
    {{end}}
```
`_footer.html`
```html
{{$group:=.group}}
{{$site:=.site}}
{{$categories := .categories}}
{{$category := .category}}
{{$categoryid:= .categoryid}}
{{$post:=.post}}
{{$theme :=.theme}}

{{html $site.ADBottom}}{{html $group.ADBottom}} <!-- 显示站点和分组的底部广告代码 -->
<footer class="main-footer">
    <div class="textwidget">Copyright © 2015 <a href="http://{{$site.Domain}}">{{$site.Title}}</a>.</div>
</footer>
<img src="/stat_icon.gif" alt="stats"/> <!-- AS统计代码调用,不加这个无法统计用户访问 -->
{{html $group.TXTBodyB}} <!-- 显示分组页面底部代码 -->
{{html $site.TXTBodyB}} <!-- 显示站点页面底部代码 -->
</body>
</html>
```
`_sidebar.html`
```html
<!-- 侧栏代码开始 -->
{{$group:=.group}}
{{$site:=.site}}
{{$categories := .categories}}
{{$category := .category}}
{{$categoryid:= .categoryid}}
{{$post:=.post}}
{{$theme :=.theme}}

<form method="get" action="/search/"><!-- 搜索框 -->
    <input type="text" name="q" id="q" style="width:200px" placeholder="" value="{{.keyword}}" /><input type="submit" value="Search" />
</form>
{{html $site.ADSidebar}} <!-- 显示组和站的侧栏广告代码,一般300*250 -->
{{html $group.ADSidebar}}


<!-- 显示最后被回复的10篇文章列表 开始-->
<ul class="posts-list">
    {{$posts := GET_POSTS $site.Id  $categoryid 10 "-lastreplydt"}} 
    {{range $posts}}
    <li>
        <a href="{{.GetUrl}}">
        <img src="/images/{{.Id}}/110x96" class="attachment-post-thumbnail wp-post-image" title="{{.Title}}">    
        </a>
        <div class="content">
            <time datetime="{{.PublishDT.YMDHS}}">{{.PublishDT.MDHS}} </time>
            <span class="comments"><a href="{{.GetUrl}}#comments"><i class="fa fa-comments-o"></i>{{if .Comments}} {{.Comments}}{{end}}</a></span>
            <a href="{{.GetUrl}}" title="{{.Title}}">{{.Title}}</a>
        </div>
    </li>
    {{end}}
</ul>
<!-- 显示最后被回复的10篇文章列表 结束-->
<!-- 侧栏代码结束 -->
```
`_postlist.html`
```html
{{range .}}<!-- 循环数据,这个数据现在还不知道哪来的 -->
<article >
    <span class="cat-title"><a href="{{.Category.GetUrl}}">{{.Category.Name}}</a></span> <!-- 文章栏目名称 -->
    <a href="{{.GetUrl}}" itemprop="url">
        <!-- 文章连接 -->
        <img src="/images/{{.Id}}/350x180" class="attachment-main-block wp-post-image no-display appear" alt="{{.Title}}" title="{{.Title}}" itemprop="image" height="185" width="351">        <!-- 文章图片 -->        
    </a>
    <time datetime="{{.PublishDT.YMDHS}}" itemprop="datePublished">{{.PublishDT.MDHS}}</time> <!-- 文章发布时间 -->
    <span class="comments"><a href="{{.GetUrl}}#comments">{{.Comments}}</a></span><!-- 文章评论数 -->
    <a href="{{.GetUrl}}" title="{{.Title}}" itemprop="name headline url">{{.Title}}</a><!-- 文章标题连接 -->
    <div class="excerpt">
        <p>{{html .Summary}}</p>
        <!-- 文章抽取的重点简介 -->
        <div class="read-more"><a href="{{.GetUrl}}" title="Read More">Read More</a></div>
        <!-- 读取更多连接 -->
    </div>
</article>
{{end}}<!-- 结束文章循环 -->
```
###五、页面详解

1),分类页,分类页的局很简单.代码也很简单.从它入手.
![](http://i.imgur.com/WJsKnl2.png)
因为已经有了几个模块了.现在只要拼这些模块就行了.

`category.html`
```html
{{$group:=.group}}
{{$site:=.site}}
{{$categories := .categories}}
{{$category := .category}}
{{$categoryid:= .categoryid}}
{{$post:=.post}}
{{$theme :=.theme}}

{{template "_header.html" .}} <!-- 调用头模块显示,并把所有的参数传过去. -->

<div class="main wrap cf">
    <div class="row">
        <div class="col-8 main-content">
            <div class="page type-page status-publish page-content">
                <header class="post-header">
                    <h1 class="main-heading">
                        {{.category.Name}}    <!-- 显示当前分类名称 -->            
                    </h1>
                </header>


                    {{template "_postlist.html" .posts}} <!-- 调用postlist显示,并只把  .post 参数传过去 -->

                <div class="main-pagination">{{.pager}}</div><!-- 显示分页(如果需要分页的话) -->
            </div>
        </div>

{{template "_sidebar.html" .}}<!-- 显示侧栏 -->

    </div>
    <!-- .row -->
</div>
{{template "_footer.html" .}}<!-- 显示页脚 -->
```
这里有个模板嵌套的方法,就是使用 `{{template "模块名称" 参数}}`
用点号.表示所有参数传递给下一级
用具体的名称,就是只将这个参数传给下一级.
在调用` _postlist.html` 模块的时候,就只传递了 `.posts` 过去, 也就是这个模块只要显示文章列表. 如果你要干更多事,就得把所有的参数传递过去.那时候 `_postlist.html`里面则需要使用具体的名称去获取变量.

2) 首页和搜索页,简单来说是和分类页一样的,只是需要不同的样式去表现文章列表.

3)内容页
内容页2部分功能,1是需要显示具体的文章内容,2是可能需要(如果打开)评论功能.

```html
{{$group:=.group}}
{{$site:=.site}}
{{$categories := .categories}}
{{$category := .category}}
{{$categoryid:= .categoryid}}
{{$post:=.post}}
{{$theme :=.theme}}

{{template "_header.html" .}} <!-- 模块显示页对 -->

{{$post.Title}}        <!-- 显示文章标题 -->
{{html .post.Body}} <!-- 显示文章正文 -->
{{if $site.EnableSourcelink}} <!-- 如果允许显示源连接,则显示 -->
<a href="{{$post.SourceUrl}}" target="_blank" rel="nofollow" class="alignright">Source</a>
{{end}}

{{html $site.ADContent}} <!-- 显示站和组的内容广告 -->
{{html $group.ADContent}}

{{html $site.ADBottom}}{{html $group.ADBottom}} <!-- 显示站和组的底部广告 -->

{{if .site.EnableComments}} <!-- 如果网站允许评论 -->
<div class="comments">
    <!-- 评论部分开始 -->
    <div id="comments">
        {{if .comments}} <!-- 如果有评论,开始显示 -->
        <h3 class="section-head">Comments</h3>
        <ol class="comments-list">
            {{range .comments}} <!-- 循环显示这个评论 -->
            <li class="comment even thread-even depth-1">
                <article class="comment">
                    <a id="comment{{.Id}}"></a>
                    <div class="comment-avatar">
                        <img alt="" src="http://1.gravatar.com/avatar/{{.EmailHash}}?s=40&amp;d=wavatar&amp;r=G" class="avatar avatar-40 photo no-display appear" height="40" width="40">                    
                    </div>
                    <div class="comment-meta">                    
                        <span class="comment-author">{{.Name}}</span> on 
                        <time pubdate="" datetime="{{.PublishDT.YMDHS}}">{{.PublishDT.MDHS}}</time>
                    </div>
                    <!-- .comment-meta -->
                    <div class="comment-content">
                        <p>{{.Body}}</p>
                    </div>
                </article>
                <!-- #comment-N -->
            </li>
            {{end}}
        </ol>
        {{end}}<!-- 结束评论列表显示 -->

        <div id="respond" class="comment-respond"><!-- 评论编辑发布开始 -->
            <h3 id="reply-title" class="comment-reply-title"><span class="section-head">Leave A Reply</span></h3>
            <form action="{{$post.GetUrl}}/comment" method="post" id="commentform" class="comment-form" novalidate=""> <!-- 评论表单,发布地址为 当前文章的URL加/comment -->
                <p>
                    <input name="author" id="author" size="40" aria-required="true" placeholder="Your Name" value="" type="text"><!-- 评论用户名 -->
                    <input name="email" id="email" size="40" aria-required="true" placeholder="Your Email" value="" type="text"><!-- 评论用户邮件 -->
                </p>
                <p>
                    <input name="captchacode" size="10" aria-required="true" placeholder="Captha" type="text"> <!-- 验工业化码框 -->
                    <a href="javascript:void(0)" onclick="$('#captchaimage').attr('src','/captcha/{{.captchaid}}.png?reload='+(new Date()).getTime());">
                    <img id="captchaimage" src="/captcha/{{.captchaid}}.png" alt="Captcha image" style="vertical-align:middle"> <!-- 验证码图片,点击更换 -->
                    </a>
                </p>
                <p>
                    <textarea name="comment" id="comment" cols="45" rows="8" aria-required="true" placeholder="Your Comment"></textarea><!-- 评论表单内容输入 -->
                </p>
                <p class="form-submit">
                    <span id="errmsg"></span>
                    <input name="submit" id="comment-submit" class="submit" value="Post Comment" type="submit">
                    <input name="token" value="{{.commenttoken}}" type="hidden"><!-- 隐藏参数,不可修改,必须 -->
                    <input type="hidden" name="captchaid" value="{{.captchaid}}"><br><!-- 隐藏参数,不可修改,必须 -->
                </p>
            </form>
        </div>
        <!-- 评论编辑发布结束-->
    </div>
    <!-- #comments -->
</div>
{{end}}   <!-- 评论部分结束 -->

{{template "_sidebar.html" .}} <!-- 模块显示侧栏 -->

<script type="text/javascript">
    $(document).ready(function(){
        $('#commentform').on('submit', function(e){
            e.preventDefault();

            $.ajax({type: 'post',url: $('#commentform').attr('action'),data:$('#commentform').serialize(),
                   success: function (res) {if (res.result){window.location.reload(true);}else{$('#errmsg').text(res.msg);}}
                });
        });
    });
</script>

{{template "_footer.html" .}}<!-- 模块显示页脚 -->
```