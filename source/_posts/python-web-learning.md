
---
title: Python Web 项目架构
date: 2015/10/24 20:47:30
categories:
- Shit Done
tags:
- Python
- flask
---

### 背景

作为整理aliyun的第二弹，把之前项目中用到的python项目的结构备注下来，后续如果要做东西的话除了nodejs又可以多了一项选择: )，这里要感谢@小泽同学，进行的python相关组内分享。
### 选型

关于python的web框架也是挺多的，这里我就不做各种比较了，因为也没有很多的这方面的经验，不过这次选择用的[flask框架](http://docs.jinkan.org/docs/flask/)还是挺简洁，小巧。而且扩展性很好，具有很多的插件可以用。基于flask继而确定了如下的方案：

| 名称 | 方案 |
| --- | --- |
| web 框架 | flask |
| web server | gunicorn |
| database | MongoDB |

围绕这些，还安装了Flask-Script，pymongo。其中[Flask-Script]()是Flask的一个扩展，可以往python程序中增加脚本功能，例如查看目前程序中提供了哪些路由：
`python manage.py urls`  。pymongo可以猜到是mongo的一个驱动。这些都可以通过`pip`来进行安装。
### 代码目录结构

确定好了选型了之后，就可以对代码结构进行构造了

```
├── lib    //业务代码依赖的一些库，pyton的扩展
│   └── flask_plugin   //python的扩展
│   └── mongoUtils.py    //DB操作的基类
└── blog               //业务代码
│   ├── models         //db层，根据或者结构划分文件，封装db的操作。一般继承操作DB的基类
│   ├── service        //处理业务逻辑的类，调用DB和返回数据给views
│   └── views          //根据声明的路由，调用业务处理请求。
│   └── config.py      //一些固定的配置，如db配置。
│——— manage.py           //项目的启动文件，一些Flask-Script的扩展命令写在这里。
```

整个项目结构，基本上如此，可以看出，整体上还是一个MVC的结构。其实还可以根据业务看是否把views和service中所有的业务class都从各自的一个基类中继承。这样整个看起来就很清晰了。除此之外，python为了文件之间`import`方便，制定了package的概念，可以[参考这里](http://www.cnpythoner.com/post/2.html)进行了解。一般在文件夹下面创建了`__init__.py`文件，则此文件夹就被当做为一个package，这样就可以通过   `import`的方式进行引入。真怀疑ES6的module机制也是从python抄来的 : )

接下来，会同步下reactjs的入门探索。
