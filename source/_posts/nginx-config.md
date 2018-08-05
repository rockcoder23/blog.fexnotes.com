---
title: JAVA 对象得生命周期
date: 2015/10/7 20:47:30
categories:
- Shit Done
tags:
- Nginx 
- nginx 配置
---


### 一、背景

放假七天，没什么事情，就想结合自己的工作经验，整理下我的aliyun服务器。那么就从nginx开始。
### 二、 选型

目前[nginx提供的版本](http://nginx.org/en/download.html)有很多, 面对这么多版本，翻了翻changelog，发现最新版的刚好添加了http2的支持，而且对于我来的个人站点来说，nginx的版本并没有太大的要求，基本上都能满足，所以就决定试试最新版，也为之后可以试试http2的特性做准备。但是大家在生产环境还是尽量选择stable版本 : )
### 三、安装
#### 1. 之前的方式

我的aliyun当初配置的是ubuntu系统，之前安装什么软件都是采用`apt-get install`的方式，这种方式对于使用来说很方便，但是有个缺点对于软件安装的目录很分散，导致你要配置一个东西需要找各种目录, 还有就是有些软件要删除的时候使用`apt-get remove`经常删除不完整，导致经常看到这样一个问题[`What is the correct way to completely remove an application`](http://askubuntu.com/questions/187888/what-is-the-correct-way-to-completely-remove-an-application) ，特别是时间久了，自己也会忘记了删除的正确方法。
#### 2. 目前的方式

所以这次，打算学着sys的做法，建立一个独立的用户`/home/sys` 用来管理和安装这些我们要经常维护的软件。这样安装好了之后，只要把`/home/sys/nginx/sbin/nginx`建立一个软链到/usr/sbin中就可以了(编译安装过程自行建立)。 要删除的时候也可以直接删除nginx文件夹和软链就可以了。但是这种方式就得我们自己编译和安装nginx了，不过这个也很简单。
#### 3. 操作

``` sh
useradd sys -m  #创建/home/sys和sys账号

cd /home/sys

mkdir nginx   #创建/home/sys/nginx目录待会安装使用。

wget  http://nginx.org/download/nginx-1.9.5.tar.gz  #下载nginx源码包

tar -zxvf  nginx-1.9.5.tar.gz ###解压

cd  nginx-1.9.5

./configure --prefix=/home/sys/nginx  #检测依赖和配置安装目录, 会再当前目录中生成makefile文件

make && make install   # 执行编译和安装。
```

就这样把nginx独立编译安装到你指定的目录中，当然肯定会在configure步骤中遇到一些依赖问题，这个各位就自行解决啦，因为这个过程和问题google一大堆。
### 四、配置

nginx安装好了之后，进入nginx目录，我们要重点关注的就是conf文件夹了, 那是我们需要重点配置和维护的地方。可以看到里面每份文件都有两个版本，如：nginx.conf  和 nginx.conf.default，你git diff下发现其实里面内容是一模一样的。这个估计是nginx考虑到经常备份的问题，所以就自己先提供一份备份，这样就可以直接修改_.conf文件，因为搞坏了还可以直接用_.conf.default恢复。

在nginx.conf中我们主要关注是的`http  server location`这种块级的指令，而这次我主要要做的是把nginx的配置合理的解耦, 这个是咱们配置的重点。主要分为三部分，http指令的配置， server指令的配置， 还有upstream的配置。分别对应的如下的文件或文件夹： nginx.conf,   nginx/vhost/_.conf,   nginx/upstream/_.conf;  关于这些名词可参考[这里](http://nginx.org/en/docs/beginners_guide.html#conf_structure)
#### 1. nginx.conf

先看第一部分，nginx.conf 中咱们把默认http下server配置全部移除，留下http部分的配置。这里我们主要针对的是落在这台nginx的所有请求进行配置。截取主要的配置文件内容如下：

```
user       www www;  ## Default: nobody 以什么用户启动nginx
worker_processes  5;  ## Default: 1  启动后fork多少个work子进程，可以改动后看看效果，根据cpu核数配置，或者直接写 auto；
error_log  logs/error.log;
pid        logs/nginx.pid;
worker_rlimit_nofile 1024;  ###每个work进程最大并发数量

events {
  worker_connections  1024;  ## Default: 1024  ####最大连接数，受worker_rlimit_nofile限制
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '  #配置日志的等级main及格式
                     '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  error '$remote_addr - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;                                      #默认开始gzip压缩
    include upstream/*.conf;             #通过include指令引入需要代理到的upstream配置。
    include vhost/fex/*.conf;             #通过include指令引入server部分(virtul host)的配置。
}
```
#### 2. vhost/*.conf

这个目录下里面的文件主要对server进行配置，可以直接连接对某个域名站点进行配置。是对某一个域名下的请求进行配置的，所以可以以域名为单位建立配置文件，如：vhost/fexnotes.com.conf。简单截取文件内容下：

```
server {
    listen       80;      #监听80端口
    server_name  fexnotes.com *.fexnotes.com;  #服务的域名
    access_log   logs/fexnotes.access.log main;  #使用main等级配置访问日志。
    error_log    logs/fexnotes.error.log error;          #使用error等级配置错误日志。

    set $the_fexnotes_upstream   "to_hexo_all";   #使用set指令配置upstream为：'to_hexo_all'
    location / {  #将所有的请求代理到the_fexnotes_upstream中。
      proxy_pass      http://$the_fexnotes_upstream;    
    }
  }
```
#### 3. upstream/*.conf

upstream中主要用来从server指令中配置反向代理到的机器列表。同理可以使用域名为单位建立配置文件，如： fexnotes.com.upstream.conf.简单截取内容如下：

```
upstream to_hexo_all {
    server x.x.x.x:port max_fails=3 fail_timeout=10s;  #ip端口可以自行配置。 这样请求落最后落到了这台机器上。
}
```

经过如上的配置，这样就把原本在nginx.conf一个文件中配置所有的内容，解耦成三部分，这样维护起来就方便多了。以后需要增加域名，只要在vhost和upstream中新建域名相应文件和配置就可以了。
### 五、启动

大功告成，启动命令如下：

``` sh
 cd /home/sys/sbin/nginx  
./nginx -t   #检测是否配置有问题
./nginx -r reload  #没有文件就可以直接reload生效了
```

现在[点击这里](http://www.fexnotes.com)访问下试试吧. : )
### 参考：
1. http://nginx.org/en/docs/beginners_guide.html#conf_structure
2. https://www.nginx.com/resources/admin-guide/?_ga=1.72470931.143333815.1440149452
3. http://nginx.org/en/docs/
