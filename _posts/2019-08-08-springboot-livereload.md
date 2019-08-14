---
title: Spring Boot 修改静态资源一定要重启项目才会生效吗？未必！
date: 2019-08-08 21:40:59
tags: [Spring Boot,LiveReload]
categories: Spring Boot
abbrlink: springboot-livereload
sidebar:
  nav: docs-zh
---

# 回顾热部署

Spring Boot 中的热部署相信大家都用过吧，只需要添加 `spring-boot-devtools` 依赖就可以轻松实现热部署。Spring Boot 中热部署最最关键的原理就是两个不同的 classloader：

<!--more-->

- base classloader
- restart classloader

其中 base classloader 用来加载那些不会变化的类，例如各种第三方依赖，而 restart classloader 则用来加载那些会发生变化的类，例如你自己写的代码。Spring Boot 中热部署的原理就是当代码发生变化时，base classloader 不变，而 restart classloader 则会被废弃，被另一个新的 restart classloader 代替。在整个过程中，因为只重新加载了变化的类，所以启动速度要被重启快。

但是有另外一个问题，就是静态资源文件！使用 devtools ，默认情况下当静态资源发生变化时，并不会触发项目重启。虽然我们可以通过配置解决这一问题，但是没有必要！因为静态资源文件发生变化后不需要编译，按理说保存后刷新下就可以访问到了。

那么如何才能实现静态资源变化后，不编译就能自动刷新呢？ LiveReload 可以帮助我们实现这一功能！

# LiveReload

devtools 中默认嵌入了 LiveReload 服务器，利用 LiveReload 可以实现静态文件的热部署，LiveReload 可以在资源发生变化时自动触发浏览器更新，LiveReload 支持 Chrome、Firefox 以及 Safari 。以 Chrome 为例，在 Chrome 应用商店搜索 LiveReload ，结果如下图：

![](http://www.javaboy.org/images/boot/27-1.png)
 
将第一个搜索结果添加到 Chrome 中，添加成功后，在 Chrome 右上角有一个 LiveReload 图标

![](http://www.javaboy.org/images/boot/27-2.png)

在浏览器中打开项目的页面，然后点击浏览器右上角的 LiveReload 按钮，打开 LiveReload 连接。

**注意：**

LiveReload 是和浏览器选项卡绑定在一起的，在哪个选项卡中打开了 LiveReload，就在哪个选项卡中访问页面，这样才有效果。

打开 LiveReload 之后，我们启动一个加了 devtools 依赖的 Spring Boot 项目：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

此时随便在 resources/static 目录下添加一个静态 html 页面，然后启动 Spring Boot 项目，在**打开了 LiveReload 的选项卡中访问 html 页面**。

访问成功后，我们再去手动修改 html 页面代码，修改成功后，回到浏览器，不用做任何操作，就会发现浏览器自动刷新了，页面已经更新了。

整个过程中，我的 Spring Boot 项目并没有重启。

如果开发者安装并且启动了 LiveReload 插件，同时也添加了 devtools 依赖，但是却并不想当静态页面发生变化时浏览器自动刷新，那么可以在 application.properties 中添加如下代码进行配置：

```
spring.devtools.livereload.enabled=false
```

# 最佳实践

建议开发者使用 LiveReload 策略而不是项目重启策略来实现静态资源的动态加载，因为项目重启所耗费时间一般来说要超过使用LiveReload 所耗费的时间。

Firefox 也可以安装 LiveReload 插件，装好之后和 Chrome 用法基本一致，这里不再赘述。