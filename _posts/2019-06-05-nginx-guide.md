---
title: Nginx 极简入门教程！
date: 2019-06-05 08:35:27
tags: Nginx
categories: Nginx
abbrlink: nginx-guide
sidebar:
  nav: docs-zh
---

上篇文章和大家聊了 Spring Session 实现 Session 共享的问题，有的小伙伴看了后表示对 Nginx 还是很懵，因此有了这篇文章，算是一个 Nginx 扫盲入门吧！

<!--more-->

# 基本介绍

`Nginx` 是一个高性能的 `HTTP` 和反向代理 `web` 服务器，同时也提供了 `IMAP/POP3/SMTP` 服务。 

`Nginx` 是由伊戈尔·赛索耶夫为俄罗斯访问量第二的 `Rambler.ru` 站点开发的，第一个公开版本 `0.1.0` 发布于 `2004` 年 `10` 月 `4` 日。 

`Nginx` 特点是占有内存少，并发能力强。

事实上 `nginx` 的并发能力确实在同类型的网页服务器中表现较好，一般来说，如果我们在项目中引入了 `Nginx` ，我们的项目架构可能是这样：

![](http://www.javaboy.org/images/boot/15-1.png)

在这样的架构中 ， `Nginx` 所代表的角色叫做负载均衡服务器或者反向代理服务器，所有请求首先到达 `Nginx` 上，再由 `Nginx` 根据提前配置好的转发规则，将客户端发来的请求转发到某一个 `Tomcat` 上去。

那么这里涉及到两个概念：

- 负载均衡服务器

就是进行请求转发，降低某一个服务器的压力。负载均衡策略很多，也有很多层，对于一些大型网站基本上从 `DNS` 就开始负载均衡，负载均衡有硬件和软件之分，各自代表分别是 `F5` 和 `Nginx` （目前 `Nginx` 已经被 `F5` 收购），早些年，也可以使用 `Apache` 来做负载均衡，但是效率不如 `Nginx` ，所以现在主流方案是 `Nginx` 。

- 反向代理服务器：

另一个概念是反向代理服务器，得先说正向代理，看下面一张图：

![](http://www.javaboy.org/images/boot/15-2.png)

在这个过程中，Google 并不知道真正访问它的客户端是谁，它只知道这个中间服务器在访问它。因此，这里的代理，实际上是中间服务器代理了客户端，这种代理叫做正向代理。

那么什么是反向代理呢？看下面一张图：

![](http://www.javaboy.org/images/boot/15-3.png)

在这个过程中，10086 这个号码相当于是一个代理，真正提供服务的，是话务员，但是对于客户来说，他不关心到底是哪一个话务员提供的服务，他只需要记得 10086 这个号码就行了。

所有的请求打到 10086 上，再由 10086 将请求转发给某一个话务员去处理。因此，在这里，10086 就相当于是一个代理，只不过它代理的是话务员而不是客户端，这种代理称之为反向代理。

# Nginx 的优势

在 Java 开发中，Nginx 有着非常广泛的使用，随便举几点：

1. 使用 Nginx 做静态资源服务器：Java 中的资源可以分为动态和静态，动态需要经过 Tomcat 解析之后，才能返回给浏览器，例如 JSP 页面、Freemarker 页面、控制器返回的 JSON 数据等，都算作动态资源，动态资源经过了 Tomcat 处理，速度必然降低。对于静态资源，例如图片、HTML、JS、CSS 等资源，这种资源可以不必经过 Tomcat 解析，当客户端请求这些资源时，之间将资源返回给客户端就行了。此时，可以使用 Nginx 搭建静态资源服务器，将静态资源直接返回给客户端。
2. 使用 Nginx 做负载均衡服务器，无论是使用 Dubbo 还是 Spirng Cloud ，除了使用各自自带的负载均衡策略之外，也都可以使用 Nginx 做负载均衡服务器。
3. 支持高并发、内存消耗少、成本低廉、配置简单、运行稳定等。

# Nginx 安装：

由于基本上都是在 Linux 上使用 Nginx，因此松哥这里主要向大家展示 CentOS 7 安装 Nginx：

1. 首先下载 Nginx

```
wget http://nginx.org/download/nginx-1.17.0.tar.gz
```

然后解压下载的目录，进入解压目录中，在编译安装之前，需要安装两个依赖：

```
yum -y install pcre-devel
yum -y install openssl openssl-devel
```

然后开始编译安装：

```
./configure
make
make install
```

装好之后，默认安装位置在 ：

```
/usr/local/nginx/sbin/nginx
```

进入到该目录的 `sbin` 目录下，执行 `nginx` 即可启动 `Nginx` ：

![](http://www.javaboy.org/images/boot/15-4.png)

Nginx 启动成功之后，在浏览器中直接访问 Nginx 地址：

![](http://www.javaboy.org/images/boot/15-5.png)

看到如上页面，表示 Nginx 已经安装成功了。

如果修改了 Nginx 配置，则可以通过如下命令重新加载 Nginx 配置文件：

```
./nginx -s reload
```

# 总结

本文算是一个简单的 Nginx 扫盲文，希望大家看完后对 Nginx 有一个基本的认知。本文先说到这里，有问题欢迎留言讨论。