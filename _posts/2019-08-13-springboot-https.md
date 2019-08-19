---
title: Spring Boot 加入 Https 功能有那么难吗？
date: 2019-08-13 21:47:14
tags: [Spring Boot,Https]
categories: Spring Boot
abbrlink: springboot-https
sidebar:
  nav: docs-zh
---

https 现在已经越来越普及了，特别是做一些小程序或者公众号开发的时候，https 基本上都是刚需了。

<!--more-->

不过一个 https 证书还是挺费钱的，个人开发者可以在各个云服务提供商那里申请一个免费的证书。我印象中有效期一年，可以申请 20 个。

今天要和大家聊的是在 Spring Boot 项目中，如何开启 https 配置，为我们的接口保驾护航。

# https 简介

我们先来看看什么是 https，根据 wikipedia 上的介绍：

> 超文本传输安全协议(HyperText Transfer Protocol Secure)，缩写：HTTPS；常称为 HTTP over TLS、HTTP over SSL 或 HTTP Secure）是一种通过计算机网络进行安全通信的传输协议。HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包。HTTPS 开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。这个协议由网景公司(Netscape)在 1994 年首次提出，随后扩展到互联网上。

历史上，HTTPS 连接经常用于网络上的交易支付和企业信息系统中敏感信息的传输。在 2000 年代末至 2010 年代初，HTTPS 开始广泛使用，以确保各类型的网页真实，保护账户和保持用户通信，身份和网络浏览的私密性。

另外，还有一种安全超文本传输协议（S-HTTP），也是 HTTP 安全传输的一种实现，但是 HTTPS 的广泛应用而成为事实上的 HTTP 安全传输实现，S-HTTP并没有得到广泛支持。

# 准备工作

首先我们需要有一个 https 证书，我们可以从各个云服务厂商处申请一个免费的，不过自己做实验没有必要这么麻烦，我们可以直接借助 Java 自带的 JDK 管理工具 keytool 来生成一个免费的 https 证书。

进入到 `%JAVVA_HOME%\bin` 目录下，执行如下命令生成一个数字证书：

```
keytool -genkey -alias tomcathttps -keyalg RSA -keysize 2048  -keystore D:\javaboy.p12 -validity 365
```

命令含义如下：

- genkey 表示要创建一个新的密钥。
- alias 表示 keystore 的别名。
- keyalg 表示使用的加密算法是 RSA ，一种非对称加密算法。
- keysize 表示密钥的长度。
- keystore 表示生成的密钥存放位置。
- validity 表示密钥的有效时间，单位为天。

具体生成过程如下图：

![](http://www.javaboy.org/images/boot/30-1.png)

命令执行完成后 ，我们在 D 盘目录下会看到一个名为 javaboy.p12 的文件。如下图：

![](http://www.javaboy.org/images/boot/30-2.png)

有了这个文件之后，我们的准备工作就算是 OK 了。

# 引入 https

接下来我们需要在项目中引入 https。

将上面生成的 javaboy.p12 拷贝到 Spring Boot 项目的 resources 目录下。然后在 application.properties 中添加如下配置：

```properties
server.ssl.key-store=classpath:javaboy.p12
server.ssl.key-alias=tomcathttps
server.ssl.key-store-password=111111
```

其中：

- key-store表示密钥文件名。
- key-alias表示密钥别名。
- key-store-password就是在cmd命令执行过程中输入的密码。

配置完成后，就可以启动 Spring Boot 项目了，此时如果我们直接使用 Http 协议来访问接口，就会看到如下错误：

![](http://www.javaboy.org/images/boot/30-3.png)

改用 https 来访问 ，结果如下：

![](http://www.javaboy.org/images/boot/30-4.png)

这是因为我们自己生成的 https 证书不被浏览器认可，不过没关系，我们直接点击继续访问就可以了（实际项目中只需要更换一个被浏览器认可的 https 证书即可）。

![](http://www.javaboy.org/images/boot/30-5.png)

# 请求转发

考虑到 Spring Boot 不支持同时启动 HTTP 和 HTTPS ，为了解决这个问题，我们这里可以配置一个请求转发，当用户发起 HTTP 调用时，自动转发到 HTTPS 上。

具体配置如下：

```java
@Configuration
public class TomcatConfig {
    @Bean
    TomcatServletWebServerFactory tomcatServletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory(){
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint constraint = new SecurityConstraint();
                constraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                constraint.addCollection(collection);
                context.addConstraint(constraint);
            }
        };
        factory.addAdditionalTomcatConnectors(createTomcatConnector());
        return factory;
    }
    private Connector createTomcatConnector() {
        Connector connector = new
                Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(8081);
        connector.setSecure(false);
        connector.setRedirectPort(8080);
        return connector;
    }
}
```

在这里，我们配置了 Http 的请求端口为 8081，所有来自 8081 的请求，将被自动重定向到 8080 这个 https 的端口上。

如此之后，我们再去访问 http 请求，就会自动重定向到 https。

# 结语

Spring Boot 中加入 https 其实很方便。如果你使用了 nginx 或者 tomcat 的话，https 也可以发非常方便的配置，从各个云服务厂商处申请到 https 证书之后，官方都会有一个详细的配置教程，一般照着做，就不会错了。