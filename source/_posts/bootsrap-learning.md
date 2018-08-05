---
title: bootstrap架构 
date: 2015/10/17 22:01:25
categories:
- Shit Done
tags:
- CSS
- Bootstrap 
---

## bootstrap架构

### 背景

最新负责的业务正在进行UI改版，因为之前是基于bootstrap进行的UI构造，所以把之前整理的一份bootstrap目录架构记录于此，便于翻阅。
### 目录结构

其实bootstrap整个目录结构还是挺清晰的，整个bootstrap的大致的结构如下：

```
├── dist     //build的产出文件
├── docs     //bs文档
├── fonts    //bs使用到的文件
├── grunt    //bs的打包脚本
├── js       //bs的js插件
├── less     //bs的less源码
└── test-infra   //bs构建文档的工具
```

我们主要关注的是less目录，而less中的目录结构如下

```
├── mixins               //各个组件使用的函数,以组件为单位划分文件
├── mixins.less          //mixins目录中文件的入口文件
├── boostrap.less        //整个bootstrap的入口文件
├──[components].less     //bootstrap的组件less文件                
├──theme.less            //bootstrap的主题文件
```
### 目录间关系

boostrap中有两个大的入口文件： theme.less 和 bootstrap.less。bootstrap.less引入了整个项目中最原始的组件的样式，而theme.less其实就是用来对bootstrap进行覆盖和重置样式的，以便定制出需要的样式。 其实文字用来描述这个东西有所费力，所以less目录中的文件关系我使用脑图进行了整理，便于后续查阅，截图如下：

并且附上原始的[脑图链接](http://naotu.baidu.com/file/ebc96f510255a9c1e32c980ef4b3cf45?token=cee6c0740a71ecee)

btw：最新的UI改版方案，已经确定，并且是按着预期设想的那样进行的，之后要是经过实践可行的话，会继续另起一篇blog分享 ：）
