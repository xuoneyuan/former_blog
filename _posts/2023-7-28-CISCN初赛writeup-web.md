---
layout:     post
title:      2023CISCN初赛writeup-web
subtitle:   全国大学生信息安全竞赛
date:       2023-7-24
author:     xuoneyuan
header-img: 
catalog: true
tags:
    - CTF
---
## 前言
本来最近有些懈怠了，但是想到我的博客竟然真的有人看，还是振作精神更新一些新东西
## unzip
### 靶场
ctfshow
### 靶场环境
打开靶场，发现它要求我们进行文件上传，可见是一道文件挂马getshell题目的变形

![1]({{site.baseurl}}/img-post/c-1.png)

随便上传一张图片马，发现还不能上传，返回报错代码

![1]({{site.baseurl}}/img-post/c-2.png)

代码解释：
~~~
<?php
// 禁用错误报告，不显示PHP的错误信息
error_reporting(0);

// 将当前文件的源代码进行高亮显示
highlight_file(__FILE__);

// 创建一个文件信息资源对象
$finfo = finfo_open(FILEINFO_MIME_TYPE);

// 判断上传的文件是否为ZIP压缩文件
if (finfo_file($finfo, $_FILES["file"]["tmp_name"]) === 'application/zip') {
    // 如果上传的文件是ZIP压缩文件，则执行解压缩操作
    // 将文件上传至/tmp目录，并使用unzip命令解压缩
    exec('cd /tmp && unzip -o ' . $_FILES["file"]["tmp_name"]);
};
~~~
说明我们只能上传zip文件，并且上传之后文件会解压到/tmp目录\
由此得出结论我们只要上传一个带有马的压缩包再getshell
### 软连接
软链接也叫符号链接。在符号连接中，文件实际上是一个文本文件，其中包含的有另一文件的位置信息。相当于创建一个快捷方式，记录原文件的位置，原文件删除，则该文件无法访问，有主次之分。就是可以将某个目录连接到另一个目录或者文件下，那么我们以后对这个目录的任何操作，都会作用到另一个目录或者文件下。\
创建命令 ：  ln -s 原始文件路径 软链接后的路径

### 解题思路
在 Linux 系统中，如果使用 Apache 服务器，默认的 Web 根目录是/var/www/html\
上传解压后的文件被解压到/tmp目录下，并且不能直接访问到/tmp目录，但可以直接访问网站根目录/var/www/html,我们可以利用软连接实现目录跳转\
首先我们先上传一个带有软链接的压缩包(1.zip)，然后让其指向网站的根目录（/var/www/html)，这样我们就可以通过访问该文件直接访问网站的目录了，然后我们再上传一个带有木马的压缩包(2.zip)，他的目录为1.zip的目录下

### 具体操作
1. 创建软连接
2. 压缩
![1]({{site.baseurl}}/img-post/c-6.png)
3. 删除link文件夹
4. 在当前目录下创建一个子目录
5. 进入子目录创建shell.php
![1]({{site.baseurl}}/img-post/c-7.png)
6. 返回上一级文件目录压缩子目录文件夹
![1]({{site.baseurl}}/img-post/c-8.png)
7. 上传1.zip和2.zip
![1]({{site.baseurl}}/img-post/c-3.png)
8. 访问/shell.php,查看目录内容
![1]({{site.baseurl}}/img-post/c-4.png)
9. 获取flag
![1]({{site.baseurl}}/img-post/c-5.png)
