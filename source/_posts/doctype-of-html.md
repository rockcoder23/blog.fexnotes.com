---
title: HTML 的 DOCTYPE 标签的作用
date: 2014/6/13 19:37:30
categories:
- Shit Done
tags:
- HTML 
- DOCTYPE
---


在写html页面的时候我们一般都会在首行添加`〈!DOCTYPE〉`的标签，有些编辑器会自动帮我们添加，今天讲讲这个`〈!DOCTYPE〉`标签的作用。
##### 一、HTML与XHTML

在`W3C`组织还么颁发html标准之前，开发网页的时候大家都没有加这个标签，那个时候.html页面的开发也比较混乱，直到1999年的时候，HTML 4.01成为了推荐标准，那时候起大多数人都使用这个版本。又大概在2000年的时候，W3C组织又颁发了基于HTML4.01的XML版本，并命名为XHTML1.0。

两者其实主要的差别是XHTML遵循了XML的编码标准，这意味着XHTML文档中增加了标签的属性必须包含在`""`中，所有的标签必须闭合等比较严格的规定。这个也是有经验的开发人员经常提醒新手写.html页面的要注意的地方。好了，问题来了：
- 前面我们提到在还没使用HTML 4.01作为推荐标准时，html文档是没有`〈!DOCTYPE〉`标签的，那么浏览器在面对一个.html文档时，如果有这个标签会怎么解析，没有这个文档会怎样解释。
- 标准推出后有了两种版本（标准）来写.html文档，那么浏览器在解析html文档的时候应该使用哪种标准呢？

这个时候`〈!DOCTYPE〉`标签就派上用场了。

<!--more-->
##### 二、`〈!DOCTYPE〉`约定浏览器使用什么版本的html

   `<!DOCTYPE>` 声明html文档引用哪个[DTD](http://www.w3school.com.cn/dtd/)， 和哪个html版本来解析html文档。学过xml的应该对这个比较了解了.而目前的`<!DOCTYPE>`也存在着三种风格： Strict (严格)、 Transitional（过渡）、 Frameset（框架集）。

  这三种风格下有这不同的要求和规定，下面看下HTML 4.01版本的声明：
- HTML 4.01 Strict：

``` html
  <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

   该 DTD 包含所有 HTML 元素和属性，但不包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。这个声明在我们开发中比较少用，因为它比较严格控制了html页面上不应该出现样式控制的属性。
- HTML 4.01 Transitional：

``` html
   <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" 
"http://www.w3.org/TR/html4/loose.dtd">
```

该 DTD 包含所有 HTML元素和属性，包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。这个是目前比较常用的声明。
- HTML 4.01 Frameset：

``` html
   <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" 
"http://www.w3.org/TR/html4/frameset.dtd">
```

该 DTD 等同于 HTML 4.01 Transitional，但允许框架集内容。

同理，XHTML 1.0也有类似的三种风格的`<!DOCTYPE>`声明。具体的含义参见[W3C关于此标签](http://www.w3school.com.cn/tags/tag_doctype.asp) 的说明。

   这个是第二个问题的答案。
##### 三、`<!DOCTYPE>`约定浏览器使用哪种模式渲染页面

  第二点是说怎么解析html文档，这个是说渲染页面，对的，渲染不仅仅是dom节点还包括css样式和javascript事件等等。

  在标准未出来前html文档中没有使用`<!DOCTYPE>`标签，所以在现在有了标准后我们强烈建议要添加这个标签，不然开发的页面就跟十几年前一样了，那么浏览器为了兼容旧时代的网页而开启混杂模式。现代浏览器一般在有两种方式渲染页面。一种是混杂模式（主要是使用了上一代的盒模型，为了兼容旧时代的网页），一种是标准模式（目前主流的）。有关浏览器渲染模式可以查看[这里](https://hsivonen.fi/doctype/)。当你写的html页面因为没有声明`<!DOCTYPE>`标签，就会被混杂模式渲染，那么你css样式和javascript代码的执行都有可能会收到影响，这个问题就大了。

现在直到了把，`<!DOCTYPE>`会告知浏览器应使用什么模式渲染页面。

但是，我们到目前主流的网站，如： 豆瓣，新浪， 淘宝等看看页面发现他们的`<!DOCTYPE>`标签居然是这样声明的：

``` html
<!DOCTYPE html>
```

没有html版本也没有dtd链接等，这样也能正常运行？答案是：这个是第五代html版本的`<!DOCTYPE>`声明方式，也就是HTML 5的声明方式。够简洁把，而且还能够向后兼容。这也是一种很好方式，因为现在主流的浏览器都已经支持HTML 5了。
##### 四：总结

`<!DOCTYPE>`作用主要有两个：
-  约定浏览器使用什么版本的html解析页面。
- `约定浏览器使用哪种模式渲染页面。

这是一个很重要的标签。
