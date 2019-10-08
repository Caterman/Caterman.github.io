---
layout: post
title: 使用这个jekyll博客模板，编辑和显示mathjax数学公式的一些问题
category: 技术杂谈
tags: [mathjax ,pjax,javascript,jquery]
description: 如果你遇到搭建的博客上数学公式显示的问题，看看这个吧
---
## 问题1：只会显示公式编码并没有显示公式
在`config.yml`文件中，有 一个属性是`mathjax: enabled`，将其设置为`enabled`即可，实际上设置为`enabled`后再，这个jekyll框架下，在`_includes/scripts.html`文件下面有一句：
{% raw %}
```javascript
{% if site.mathjax == 'enabled' %}
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" id='mathjax'></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {
    inlineMath: [['$','$'], ['\\(','\\)']],
    processEscapes: true
  }
});
</script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
      }
    });
</script>
{% endif %}
```
{% endraw %}  
也就是根据配置文件的这个属性值选择是否加载`mathjax.js`文件。

## 问题2：`mathjax:enabled`以后显示的公式，只能显示单行的公式
比如单行的公式很容易显示例如：
```javascript
单行公式代码：$$e ^ {i\pi} + 1 = 0 $$
```
<center>$$e ^ {i\pi} + 1 = 0$$
</center>

```javascript
多行公式代码：
$$
    \begin{matrix}
    1 & x & x^2 \\
    1 & y & y^2 \\
    1 & z & z^2 \\
    \end{matrix}
$$
```
就会显示的很奇怪，刚开始还以为是公式代码的问题，最后发现是jekyll生成的html静态网页的原因，因为在`_layouts\default.html`文件中，开头的布局显示为
```javascript
---
layout: compress
---
```
所以，所有生成的html文件都是压缩的一行文本，当然的确删除了不必要的空格和回车，很省流量，加快响应速度。但是，jekyll对markdown build的时候，对于这样的多行的公式，实际上，为了保险起见，外面嵌套了一层{% raw %}< ![CDATA[公式代码]]{% endraw %}。理论上正确的的形式如下。
{% raw %}
```javascript
%<![CDATA[
  \begin{matrix}
  1 & x & x^2 \\
  1 & y & y^2 \\
  1 & z & z^2 \\
  \end{matrix}
%]]
```
{% endraw %}  
%的目的是注释掉标签内部这一行后面的符号，不让在前端显示，如果全部挤在一行，那么公式显然也就被注释掉了，也就让人产生不能写多行公式的错觉。
具体解决的方法也自然就是不使用`layout: compress`的布局，随便改改吧，或许可以写成`uncompressed`，反正不压缩就对了。
## 问题3：点开左侧文章不能显示公式，刷新以后就能显示公式了
这个问题困扰了好久，当然，先给出正确的解决方法。
javascript，jquery，ajax什么的早就丢的干干净净，还能从代码角度解决也是很佩服自己，当然，要是对这些很熟悉的人一眼就能发现问题所在啦。  
**原因：** 当你点击左侧目录中的文章的时候，这个框架做了这样一件事情，它为了实现伪动态页面加载效果，向服务器发送的`GET`请求中包含了一个参数`_pjax=#main`，通过pjax技术，服务器端就保持左端的目录不变，更改右边显示的文章内容，即不会使得页面刷新，也意味着，所有的脚步文件都不会再请求加载一遍，而且pjax返回的容器内部（把文章装进容器）元素不会被原先的js解析执行，所以就不显示公式了。  
**解决：** 虽然鼓捣了很久，试过各种各样的乱七八糟的方法，解决却很简单。找到pjax代码的位置，在`pjax:end`的回调函数末尾添加代码
```javascript
var math = document.getElementsByClassName("文章所在的容器类名")[0];
MathJax.Hub.Queue(["Typeset",MathJax.Hub,math]);
```
这几句代码的含义看上去应该就是，直接调用mathjax里面的方法专门对容器内部解析一次，思路都能想到，谁会知道这个方法呢。尝试在mathjax.js源码寻找过，无奈被劝退了，官方文档当然纯粹是没有心思去翻，当然，这是不对的，其实当你没有直接google或者百度到答案的时候，就应该尝试去官方文档去看了。   
这里要感谢一位不知名的博主，是他解决了这个问题，我只是拿来主义，[**单击这里**](https://www.jianshu.com/p/8bec0ab9b467)  
**试错过程：** 当然作为前端的技术外行，本来就只能猜测，早就猜到了问题所在，苦于不会描述问题，不会解决问题。核心问题是，js不解析pjax返回容器的内容，所以包括在:  
文章内部插入`mathjax.js`  
在`pjax:end`的回调函数尾部重新加载`mathjax.js`  
对于每一个思路都想了多方法，当然是徒劳，好了，关于博客本身的问题解决了，以后只要解决写文章的事情了，哈哈哈，没办法，就是喜欢这个博客简洁的样子，不然早他喵的换别的框架了。
