---
title: JAVA 项目中的相对路径
date: 2013/8/9 17:57:30
categories:
- Shit Done
tags:
- java 
- 相对路径
---
File类是用来构造文件或文件夹的类,在其构造函数中要求传入一个String类型的参数,用于指示文件所在的路径.以前一直使用绝对路径作为参数,其实这里也可以使用相对路径.使用绝对路径不用说,很容易就能定位到文件,那么使用了相对路径jvm如何定位文件的呢?

按照jdk Doc上的说法”绝对路径名是完整的路径名，不需要任何其他信息就可以定位自身表示的文件。相反，相对路径名必须使用来自其他路径名的信息进行解释。默认情况下，java.io 包中的类总是根据当前用户目录来分析相对路径名。此目录由系统属性user.dir指定，通常是 Java 虚拟机的调用目录.”

相对路径顾名思义,相对于某个路径,那么究竟相对于什么路径我们必须弄明白.按照上面jdk文档上讲的这个路径是”当前用户目录”也就是”java虚拟机的调用目录”.更明白的说这个路径其实是我们在哪里调用jvm的路径.举个例子:

> 先说在dos下的情况，假设有一java源文件Example.java在d盘根目录下,该文件不含package信息.我们进入命令行窗口,然后使用”d:”命令切换到d盘根目录下,然后用javac Example.java来编译此文件,编译无错后,会在d盘根目录下自动生成Example.class文件.我们在调用java Example来运行该程序.此时我们已经启动了一个jvm,这个jvm是在d盘根目录下被启动的,所以此jvm所加载的程序中File类的相对路径也就是相对这个路径的,即d盘根目录:D:.同时” 当前用户目录”也是D:.在System.getProperty(“user.dir”);系统变量user.dir存放的也是这个值. 我们可以多做几次试验,把Example.class移动到不同路径下,同时在那些路径下,执行java Example命令启动jvm,我们会发现这个”当前用户目录”是不断变化的,它的路径始终和我们在哪启动jvm的路径是一致的.

搞清了这些,我们可以使用相对路径来创建文件,例如:

> ``` java
> File file = new File(“a.txt”);
> File.createNewFile();
> ```
> 
> 假设jvm是在D:\下启动的,那么a.txt就会生成在D:\a.txt; 此外,这个参数还可以使用一些常用的路径表示方法,例如”.”或”.\”代表当前目录,这个目录也就是jvm启动路径.所以如下代码能得到当前目录完整路径:
> 
> ``` java
> File f = new File(“.”);
> String absolutePath = f.getAbsolutePath();
> System.out.println(absolutePath);//D:\
> ```
> 
> 说说在eclipse中的情况: Eclipse中启动jvm都是在项目根路径上启动的.比如有个项目名为blog,其完整路径为:D:\work\IDE\workspace\blog.那么这个路径就是jvm的启动路径了.所以以上代码如果在eclipse里运行,则输出结果为” D:\work\IDE\workspace\blog.”

Tomcat中的情况. 如果在tomcat中运行web应用,此时,如果我们在某个类中使用如下代码:

``` java
File f = new File(“.”);
String absolutePath = f.getAbsolutePath();
System.out.println(absolutePath);
```

那么输出的将是tomcat下的bin目录.我的机器就是” D:\work\server\jakarta-tomcat-5.0.28\bin.”,由此可以看出tomcat服务器是在bin目录下启动jvm的.其实是在bin目录下的” catalina.bat”文件中启动jvm的.

总结：默认情况下，java.io 包中的类总是根据当前用户目录来分析相对路径名。此目录由系统属性user.dir指定，通常是Java虚拟机的调用目录.我们可以通过以下代码进行测试后，在进行处理；

``` java
File f = new File(“.”);
String absolutePath = f.getAbsolutePath();
System.out.println(absolutePath);//D:\
System.getProperty(“user.dir”)
```
