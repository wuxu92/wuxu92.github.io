---
layout: default
title: about
---

<style type="text/css" media="screen">
    /*@media screen and (max-width: 720px) {
        .aboutMe {
            width: 85%;
        }
    }
    @media screen and (min-width: 720px) {
        .aboutMe {
            width: 60%;
            padding-left: 30px;
        }
    }*/
    #content {
        width: 85%;
    }

</style>


<div id="content" class="aboutMe">
<form class="page-loc" method="GET" action="/search">
	<span style="float:right"><input type="text" class="web-search" name ="q" placeholder="站内搜索" /><a href="/archive/" style="margin-left: 20px;">归档</a><a href="/atom.xml" class="page-rss" style="margin-left: 20px;">订阅</a></span>
  	wuxu's blog » 关于我
</form>
<dl class="aboutDl">
	<dd><strong>wuxu, </strong>不学无术, 划水中...</dd>
	<dd><strong>weibo: </strong><a href="http://weibo.com/u/2446209193" target="_blank">@wuxu_92</a></dd>
	<dd><strong>email: </strong><a href="mailto:wuxu92@gmail.com">wuxu92@gmail.com</a></dd>
	<dd><strong>自述: </strong>学生，计算机科学与技术，PHP, Golang, MySQL, UX, JS, CSS; Java, C, <del>Groovy</del>; <del>MongoDB</del>, Redis, Memcached; Linux, CentOS, Fedora, Nginx, Git; C#, <del>WPF</del>; <del>Python</del>, Zsh, Vim</dd>
	<dd><strong>简历: </strong><a href="http://wuxu92.github.io/resume/" title="简历" target="_blank">http://wuxu92.github.io/resume/</a></dd>

	<dt>关于博客</dt>
	<dd>所有文章非特别说明皆为原创，遵循的协议为「<a href="http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh" target="_blank">署名-非商业性使用-相同方式共享</a>」，由于文章表述或者内容可能存在诸多错误，所以部分内容会作修改，为保证转载信息与源同步，转载请注明文章出处！谢谢合作 :）</dd>

	<dt>好友链接</dt>
	<dd>
        博客基础来自 <a href="http://hahaya.github.com" target="_blank">hayaya的github</a>
   </dd>
</dl>
<div class="footer">
    <small>Powered by <a href="https://github.com/mojombo/jekyll">Jekyll</a> | Copyright 2013 - 2015 Modified by <a href="/about.html">wuxu</a> | <span class="label label-info">{{site.time | date:"%Y-%m-%d %H:%M:%S %Z"}}</span></small>
</div>
</div>
<script type="text/javascript">
$(function(){
	$('#disqus_container .comment').trigger('click');
});
</script>
