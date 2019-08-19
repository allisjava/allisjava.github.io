---
title: Spring Boot 中的同一个 Bug，竟然把我坑了两次！
date: 2019-08-14 21:52:22
tags: Spring Boot
categories: Spring Boot
abbrlink: springboot-bug
sidebar:
  nav: docs-zh
---

真是郁闷，不过这事又一次提醒我解决问题还是要根治，不能囫囵吞枣，否则相同的问题可能会以不同的形式出现，每次都得花时间去搞。刨根问底，一步到位，再遇到类似问题就可以分分钟解决了。

<!--more-->

如果大家没看过松哥之前写的 Spring Boot 整合 Spring Session，可以先回顾下：

- [Spring Boot 一个依赖搞定 session 共享，没有比这更简单的方案了！](https://mp.weixin.qq.com/s/xs67SzSkMLz6-HgZVxTDFw)

# 第一次踩坑

事情是这样的，大概在今年 6 月初的时候，我在项目中使用到了 Session 共享，当时采用的方案就是 Redis+Spring Session。本来这是一个很简单的问题，我在以前的项目中也用过多次这种方案，早已轻车熟路，但是那次有点不对劲，项目启动时候报了如下错误：

![](http://www.javaboy.org/images/boot/31-1.png)

一模一样的代码，但是运行就是会出错，我感觉莫名其妙。因为在 Spring Boot 中整合 Spring Session 是一个非常简单的操作，就几行 Redis 的配置而已，我在确认了代码没问题之后，很快想到了可能是版本问题，因为当时 Spring Boot2.1.5 刚刚发布，我喜欢用最新版。于是我尝试将 Spring Boot 的版本切换到 2.1.4 ，切换回去之后，果然就 OK了，再次启动项目又不会报错了。于是基本确定这是 Spring Boot 的版本升级带来的问题。

但是当时我并没有深究，我以为就是官方出于安全考虑，让你在使用 Redis 时强制加上 Spring Security（因为根据错误提示，很容想到加上 Spring Security 依赖），加上 Spring Security 依赖之后，果然就没有问题了，我也没有多想，这件事就这样过了。​

# 第二次踩坑

前两天我在给星球上的小伙伴录制 Spring Boot 视频的时候，采用了  Spring Boot 最新版 2.1.7，也是 Spring Session，但是在创建项目的时候，忘记添加 Spring Security 依赖了（第一次踩坑之后，我每次用 Spring Session 都会自觉的加上 Spring Security 依赖），运行的时候竟然没报错！我就郁闷了。

于是我去试了 Spring Boot2.1.4、Spring Boot2.1.6 发现都没有问题，在使用 Spring Session 的时候都不需要添加 Spring Security 依赖，只有 Spring Boot2.1.5 才有这个问题。于是我大概明白了，这可能是一个 Bug，而不是版本升级的新功能。

这一次，那我就打算追究一下问题的根源。

# 源头

要追究问题的源头，我们当然得从 Spring Session 的自动化配置类开始。

在 Spring Boot2.1.5 的 `org.springframework.boot.autoconfigure.session.SessionAutoConfiguration` 类中，我看到如下源码：

```java
@Bean
@Conditional(DefaultCookieSerializerCondition.class)
public DefaultCookieSerializer cookieSerializer(ServerProperties serverProperties,
		ObjectProvider<SpringSessionRememberMeServices> springSessionRememberMeServices) {
	//.....
	map.from(cookie::getMaxAge).to((maxAge) -> cookieSerializer
			.setCookieMaxAge((int) maxAge.getSeconds()));
	springSessionRememberMeServices.ifAvailable((
			rememberMeServices) -> cookieSerializer.setRememberMeRequestAttribute(
					SpringSessionRememberMeServices.REMEMBER_ME_LOGIN_ATTR));
	return cookieSerializer;
}
```

从这一段源码中我们可以看到，这里使用到了 SpringSessionRememberMeServices ，而这个类中则用到 Spring Security 中相关的类。因此，如果不引入 Spring Security 就会报错。

我们再来看看 Spring Boot2.1.6 中 `org.springframework.boot.autoconfigure.session.SessionAutoConfiguration` 类的源码，如下：

```java
@Bean
@Conditional(DefaultCookieSerializerCondition.class)
public DefaultCookieSerializer cookieSerializer(ServerProperties serverProperties) {
	//...
	map.from(cookie::getMaxAge).to((maxAge) -> cookieSerializer.setCookieMaxAge((int) maxAge.getSeconds()));
	if (ClassUtils.isPresent(REMEMBER_ME_SERVICES_CLASS, getClass().getClassLoader())) {
		new RememberMeServicesCookieSerializerCustomizer().apply(cookieSerializer);
	}
	return cookieSerializer;
}
```

可以看到，在 Spring Boot2.1.6 中，这个问题已经得到修复。这里就没有 2.1.5 那么冲动了，上来了先用 `ClassUtils.isPresent` 方法判断了下 REMEMBER_ME_SERVICES_CLASS(`org.springframework.security.web.authentication.RememberMeServices`) 是否存在，存在的话，才有后面的操作。

至此，这个问题就总算弄懂了。

# 结语

大家平时遇到问题，如果项目不是很赶的话，可以留意多想想，多追究一下原因，说不定你会有很多意外的收获。我这次就是一个活生生的例子，一开始没多想，后来又发现不对劲，前前后后一折腾，反而又多浪费了一些时间。