---
title: 干货最新版 Spring Boot2.1.5 教程+案例合集
date: 2019-06-14 08:41:16
tags: Spring Boot
categories: Spring Boot
abbrlink: springboot-resources
---

最近发了一系列的 Spring Boot 教程，但是发的时候没有顺序，有小伙伴反映不知道该从哪篇文章开始看起，刚好最近工作告一个小小段落，松哥就把这些资料按照学习顺序重新整理了一遍，给大家做一个索引，大家照着索引就可以由浅入深的学习了。

<!--more-->

松哥刚开始写这个系列的时候最新版是 Spring Boot2.1.4 ，后来写着写着版本升级了变成 Spring Boot2.1.5 了，于是我又用 Spring Boot2.1.5 接着写，因此索引中的教程主要是这两个版本的教程。

可能有人觉得小版本的变化差异不大，事实上也确实如此，不过变化不大不意味着没有变化，给大家随便举两个例子：

- 在整合 Redis 时，Spring Boot2.1.4 不用引入 Spring Security，而 Spring Boot2.1.5 则需要引入 Spring Security。
- 再比如 Spring Security 中的角色继承，在 Spring Boot2.0.8 之前和之后的写法完全不同，这些差异松哥也给大家细细剖析了。

这一系列教程不是终点，而是一个起点，松哥后期还会不断完善这个教程，也会持续更新 Spring Boot 最新版本的教程，希望能帮到大家。教程索引如下：

1. [创建一个 Spring Boot 项目，你会几种方法？](https://mp.weixin.qq.com/s/FMVut8slVZJdxxLf3Y6jLw)
2. [这一次，我连 web.xml 都不要了，纯 Java 搭建 SSM 环境](https://mp.weixin.qq.com/s/NC_0oaeBzRjCB34U_ZWxIQ)
3. [你真的理解 Spring Boot 项目中的 parent 吗？](https://mp.weixin.qq.com/s/2w6B4fMdbTK_mGjnaMG4BQ)
4. [一文读懂 Spring Boot 配置文件 application.properties ！](https://mp.weixin.qq.com/s/cUhzpo8zkQq09d8S4WkAsw)
5. [Spring Boot中的yaml配置简介](https://mp.weixin.qq.com/s/dbSBzFICIDPLkj5Tuv2-yA)
6. [Spring Boot 中的静态资源到底要放在哪里？](https://mp.weixin.qq.com/s/rjscYivhLwg-2ECqps1J-A)
7. [极简 Spring Boot 整合 Thymeleaf 页面模板](https://mp.weixin.qq.com/s/7tgiuFceyZPHBZcLnPmkfw)
8. [Spring Boot 中关于自定义异常处理的套路！](https://mp.weixin.qq.com/s/w26MvCWQ1RO4CUJrfXi5AA)
9. [Spring Boot中通过CORS解决跨域问题](https://mp.weixin.qq.com/s/ASEJwiswLu1UCRE-e2twYQ)
10. [SpringMVC 中 @ControllerAdvice 注解的三种使用场景！](https://mp.weixin.qq.com/s/P-iQ0MH1GLJuO5dNHXEgVw)
11. [Spring Boot中，Redis缓存还能这么用！](https://mp.weixin.qq.com/s/UpTewC66iJyzq0osm_0cfw)
12. [Spring Boot 操作 Redis，三种方案全解析！](https://mp.weixin.qq.com/s/cgDtmjPWTdh44bSlLC0Qsw)
13. [Spring Boot 一个依赖搞定 session 共享，没有比这更简单的方案了！](https://mp.weixin.qq.com/s/xs67SzSkMLz6-HgZVxTDFw)
14. [另一种缓存，Spring Boot 整合 Ehcache](https://mp.weixin.qq.com/s/i9a3VOf_GMN_UBQ-8tKi3A)
15. [徒手撸一个 Spring Boot 中的 Starter ，解密自动化配置黑魔法！](https://mp.weixin.qq.com/s/tKr_shLQnvcQADr4mvcU3A)
16. [Spring Boot 定义系统启动任务，你会几种方式？](https://mp.weixin.qq.com/s/3HFAoAl1OjZ_YnLbQLDF3g)
17. [干货|一文读懂 Spring Data Jpa！](https://mp.weixin.qq.com/s/Fg5ssXuvabZwEfRMKfpY9Q)
18. [Spring Boot数据持久化之JdbcTemplate](https://mp.weixin.qq.com/s/X4-e1cf3uZafg8XtMJeo_Q)
19. [Spring Boot多数据源配置之JdbcTemplate](https://mp.weixin.qq.com/s/7po83-CAoryo1eglumW42Q)
20. [最简单的SpringBoot整合MyBatis教程](https://mp.weixin.qq.com/s/HOnX2XRDWrQ9oOKLo1ueKw)
21. [极简Spring Boot整合MyBatis多数据源](https://mp.weixin.qq.com/s/9YXwk2-4zIq60WFuy6nXdw)
22. [Spring Boot 中 10 行代码构建 RESTful 风格应用](https://mp.weixin.qq.com/s/7uO87SOu93XH2Y3iWxWicg)
23. [Spring Boot 整合 Shiro ，两种方式全总结！](https://mp.weixin.qq.com/s/JU_-gn-yZ4VJJXTZvo7nZQ)
24. [干货|一个案例学会Spring Security 中使用 JWT!](https://mp.weixin.qq.com/s/riyFQSrkQBQBCyomE__fLA)
25. [Spring Security 中的角色继承问题](https://mp.weixin.qq.com/s/7D0qJiEIzNuz8VAVvZsXCA)
26. [Spring Security 登录添加验证码](https://mp.weixin.qq.com/s/oDow2miLIst-R4NNzc_i4g)
27. [SpringSecurity登录使用JSON格式数据](https://mp.weixin.qq.com/s/X1t-VCxzxIcQKOAu-pJrdw)
28. [Spring Boot 中实现定时任务的两种方式!](https://mp.weixin.qq.com/s/_20RYBkjKrB4tdpXI3hBOA)
29. [SpringBoot整合Swagger2，再也不用维护接口文档了！](https://mp.weixin.qq.com/s/iTsTqEeqT9K84S091ycdog)
30. [整理了八个开源的 Spring Boot 学习资源](https://mp.weixin.qq.com/s/L8z4MOfP37fooNpYw-1mQg)


另外，还有一件重要的事，就是松哥把微信公众号中文章的案例，都整理到 GitHub 上了，**每个案例都对应了一篇解读的文章**，方便大家学习。松哥以前写博客没养成好习惯，有的案例丢失了，现在在慢慢整理补上。

GitHub 仓库地址：[https://github.com/lenve/javaboy-code-samples](https://github.com/lenve/javaboy-code-samples)，欢迎大家 star。已有的案例如下图：

![](http://www.javaboy.org/images/boot/19-1.png)

好了，这就是松哥说的干货，大家撸起袖子加油学吧！