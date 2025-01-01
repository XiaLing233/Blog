---
title: Nginx与Ubuntu
date: 2024-07-24
updated: 2024-07-24
"categories": Nginx
permalink: 2024/07/nginx-with-ubuntu/
---


## 一些题外话

暑假捣鼓捣鼓Nginx，花了大价钱在腾讯云买了个服务器，结果通过域名访问需要备案。其实我早就料到了这一点，只是想赌一赌，结果还是需要备案啊。。我还花了很大功夫提交了备案申请，但想想算了，太麻烦，后边还得公安备案。等我一年的服务器到期之后，迁移到国外就行了。

话说为啥要用服务器呢？想折腾折腾吧。说是想模仿一个用户登录系统，才发现，我原来欠缺了好多东西：前端、后端、数据库，都是我没接触到的。大一光顾着学数学了，一些技术一直没研究。所以先从文件存储开始，看看如何访问文件吧。这就引出了`Nginx`。

## 使用Nginx

服务器的操作系统为`Ubuntu 22.04 LTS`。我还奇怪呢，为什么我哦在VMware的虚拟机有图形化界面，服务器的只有命令行？原来服务器为了节约资源，一般不装图形界面。原来如此，为了体验纯正的Linux，还得用服务器。

### Nginx的安装

``` bash
sudo apt update
sudo apt install nginx
```

**其中：**

`sudo`意味以管理员模式运行，权限很高

### 找到配置文件，进行修改

这部分很绕，查找了不少资料。有一个比较权威的[链接](https://serverfault.com/questions/527630/difference-in-sites-available-vs-sites-enabled-vs-conf-d-directories-nginx)，但是我没看懂XD。

我当前系统的情况是这样的。安装完后，出现默认的Nginx欢迎页面。

![默认欢迎页面](https://static.xialing.icu/img/202407240024807.webp)

这个页面的配置文件，保存在`/etc/nginx/sites-available/default`中，`default`是配置文件。除此之外，有的博客会建议修改`/etc/nginx/`下的`nginx.conf`或者在`/etc/nginx/conf.d`中新建配置文件。这两种建议我都尝试过，但是没能达到预期，只能在屏幕上收到403 Forbidden或者404 Not Found。

**我的修改方法**

不敢说是正确的修改方法，但是确实起作用：

1. **只**修改default文件。把原来的所有内容都注释掉，采用`#`注释。
2. 输入以下配置：

```nginx
server {
	listen 80;
	server_name 你的IP地址或者域名（如10.80.42.245或者example.com）
	
	root /var/www;
	
	location / {
		autoindex on; # 自动索引
        autoindex_exact_size off; # 显示文件大小
        autoindex_localtime on; # 显示本地时间
	}
}
```

**说明：**

* `server`体放到最外层。如果发现文件中有`http`开头的大括号，说明找错了文件，找的可能是`nginx.conf`；
* `listen`表示监听的端口，HTTP用80即可；
* `server_name`表示服务器名，写IP地址或者域名；
* `root`表示在本地存储的根目录。比如按当前设置，当有人向服务器请求`example.com/`的文件时，`Nginx`会查找本地存储`/var/www/`下的文件，`www`后边的`/`，来自于`location`后方。换句话说，`root`的地址和`location`后面的地址是**拼接**。另一种方式是`alias /var/www/`。这里的区别是，`alias`是对地址进行**替换**，所以`www`后方的`/`不可以省略；
* `autoindex`表示是否自动开启索引。注意，如果要`autoindex`起作用，需要在当前目录下没有`index.html`等索引文件。

## 修改文件（vim）

`Ubuntu`自带的修改文件程序为`vim`，如何使用`vim`修改文件呢？

### 迁移

如何打开想要的文件？这里展示几种和路径有关的常见命令：

`ls` 	显示当前目录下的文件夹和文件；

`cd ..`	打开上一级；

`cd /path`	打开名为`path`的文件夹；

`touch filename.html`	新建一个名为`filename.html`的文件；

`sudo rm filename.html`	删除名为`filename.html`的文件。

这些操作比较简单，但对于我们当前的使用已经足够了。

### 使用`vim`打开文件

输入`sudo vim /path/to/your/file/test.html`使用`vim`编辑文件。这里，`sudo`是因为修改有些文件需要管理权限。

![vim页面概览](https://static.xialing.icu/img/202407240048900.webp)

* 输入`i`进入到插入（INSERT）模式，可以修改内容，就像一般的文本编辑器一样；

* 进入插入模式后，按`Esc`退出该模式；
* 在上一步的基础上，完成编辑，保存退出，用`:wq`。

这些操作，暂时够用了。

## 使设置生效

使修改后的Nginx设置生效，使用`sudo nginx -s reload`。再打开网页，发现大功告成：

![大功告成](https://static.xialing.icu/img/202407240052676.webp)

## 重装Nginx

有时候，走了太多弯路，想要恢复Nginx的默认设置，怎么办呢？

[参考](https://henchat.net/%E5%AE%8C%E6%95%B4%E9%87%8D%E8%A3%85nginx/)

```
apt-get remove --purge nginx nginx-full nginx-common
apt-get install nginx
```

有可能需要`sudo`，第二行可以替换为等效语句（如2.1Nginx安装的两句话）。

## 选择什么文件夹共享？

我一开始选择的是自定义的一个文件夹，也按照要求修改了权限，但是仍然显示`403 Forbidden`。修改为`\var\www`后，发现可以正常共享。不知道哪里设置出了问题。