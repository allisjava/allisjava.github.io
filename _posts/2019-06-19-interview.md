---
sidebar:
  nav: docs-zh
title: 松哥整理了 15 道 Spring Boot 高频面试题，看完当面霸
date: 2019-06-19 09:32:30
tags: 面试
categories: 面试
abbrlink: interview
---

什么是面霸？就是在面试中，神挡杀神佛挡杀佛，见招拆招，面到面试官自惭形秽自叹不如！松哥希望本文能成为你面霸路上的垫脚石！

<!--more-->

做 Java 开发，没有人敢小觑 Spring Boot 的重要性，现在出去面试，无论多小的公司 or 项目，都要跟你扯一扯 Spring Boot，扯一扯微服务，不会？没用过？ Sorry ，我们不合适！

今天松哥就给大家整理了 15 道高频 Spring Boot 面试题，希望能够帮助到刚刚走出校门的小伙伴以及准备寻找新的工作机会的小伙伴。

- 1.什么是 Spring Boot ?

传统的 SSM/SSH 框架组合配置繁琐臃肿，不同项目有很多重复、模板化的配置，严重降低了 Java 工程师的开发效率，而 Spring Boot 可以轻松创建基于 Spring 的、可以独立运行的、生产级的应用程序。通过对 Spring 家族和一些第三方库提供一系列自动化配置的 Starter，来使得开发快速搭建一个基于 Spring 的应用程序。

Spring Boot 让日益臃肿的 Java 代码又重回简洁。在配合 Spring Cloud 使用时，还可以发挥更大的威力。

- 2.Spring Boot 有哪些特点 ?

Spring Boot 主要有如下特点：

1. 为 Spring 开发提供一个更快、更广泛的入门体验。
2. 开箱即用，远离繁琐的配置。
3. 提供了一系列大型项目通用的非业务性功能，例如：内嵌服务器、安全管理、运行数据监控、运行状况检查和外部化配置等。
4. 绝对没有代码生成，也不需要XML配置。

- 3.Spring Boot 中的 starter 到底是什么 ?

首先，这个 Starter 并非什么新的技术点，基本上还是基于 Spring 已有功能来实现的。首先它提供了一个自动化配置类，一般命名为 `XXXAutoConfiguration` ，在这个配置类中通过条件注解来决定一个配置是否生效（条件注解就是 Spring 中原本就有的），然后它还会提供一系列的默认配置，也允许开发者根据实际情况自定义相关配置，然后通过类型安全的属性注入将这些配置属性注入进来，新注入的属性会代替掉默认属性。正因为如此，很多第三方框架，我们只需要引入依赖就可以直接使用了。

当然，开发者也可以自定义 Starter，自定义 Starter 可以参考：[徒手撸一个 Spring Boot 中的 Starter ，解密自动化配置黑魔法！](https://mp.weixin.qq.com/s/tKr_shLQnvcQADr4mvcU3A)。

- 4.spring-boot-starter-parent 有什么用 ?

我们都知道，新创建一个 Spring Boot 项目，默认都是有 parent 的，这个 parent 就是 spring-boot-starter-parent ，spring-boot-starter-parent 主要有如下作用：

1. 定义了 Java 编译版本为 1.8 。
2. 使用 UTF-8 格式编码。
3. 继承自 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
4. 执行打包操作的配置。
5. 自动化的资源过滤。
6. 自动化的插件配置。
7. 针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml。

关于这个问题，读者可以参考：[你真的理解 Spring Boot 项目中的 parent 吗？](https://mp.weixin.qq.com/s/2w6B4fMdbTK_mGjnaMG4BQ)

- 5.YAML 配置的优势在哪里 ?

YAML 现在可以算是非常流行的一种配置文件格式了，无论是前端还是后端，都可以见到 YAML 配置。那么 YAML 配置和传统的 properties 配置相比到底有哪些优势呢？

1. 配置有序，在一些特殊的场景下，配置有序很关键
2. 支持数组，数组中的元素可以是基本数据类型也可以是对象
3. 简洁

相比 properties 配置文件，YAML 还有一个缺点，就是不支持 @PropertySource 注解导入自定义的 YAML 配置。

关于 YAML 配置，要是大家还不熟悉，可以参考: [Spring Boot 中的 yaml 配置简介](https://mp.weixin.qq.com/s/dbSBzFICIDPLkj5Tuv2-yA)

- 6.Spring Boot 中如何解决跨域问题 ?

跨域可以在前端通过 JSONP 来解决，但是 JSONP 只可以发送 GET 请求，无法发送其他类型的请求，在 RESTful 风格的应用中，就显得非常鸡肋，因此我们推荐在后端通过 （CORS，Cross-origin resource sharing） 来解决跨域问题。这种解决方案并非 Spring Boot 特有的，在传统的 SSM 框架中，就可以通过 CORS 来解决跨域问题，只不过之前我们是在 XML 文件中配置 CORS ，现在则是通过 @CrossOrigin 注解来解决跨域问题。关于 CORS ，小伙伴们可以参考：[Spring Boot 中通过 CORS 解决跨域问题](https://mp.weixin.qq.com/s/ASEJwiswLu1UCRE-e2twYQ)

- 7.比较一下 Spring Security 和 Shiro 各自的优缺点 ?

由于 Spring Boot 官方提供了大量的非常方便的开箱即用的 Starter ，包括 Spring Security 的 Starter ，使得在 Spring Boot 中使用 Spring Security 变得更加容易，甚至只需要添加一个依赖就可以保护所有的接口，所以，如果是 Spring Boot 项目，一般选择 Spring Security 。当然这只是一个建议的组合，单纯从技术上来说，无论怎么组合，都是没有问题的。Shiro 和 Spring Security 相比，主要有如下一些特点：

1. Spring Security 是一个重量级的安全管理框架；Shiro 则是一个轻量级的安全管理框架
2. Spring Security 概念复杂，配置繁琐；Shiro 概念简单、配置简单
3. Spring Security 功能强大；Shiro 功能简单

- 8.微服务中如何实现 session 共享 ?

在微服务中，一个完整的项目被拆分成多个不相同的独立的服务，各个服务独立部署在不同的服务器上，各自的 session 被从物理空间上隔离开了，但是经常，我们需要在不同微服务之间共享 session ，常见的方案就是 Spring Session + Redis 来实现 session 共享。将所有微服务的 session 统一保存在 Redis 上，当各个微服务对 session 有相关的读写操作时，都去操作 Redis 上的 session 。这样就实现了 session 共享，Spring Session 基于 Spring 中的代理过滤器实现，使得 session 的同步操作对开发人员而言是透明的，非常简便。 session 共享大家可以参考：[Spring Boot 一个依赖搞定 session 共享，没有比这更简单的方案了！](https://mp.weixin.qq.com/s/xs67SzSkMLz6-HgZVxTDFw)

- 9.Spring Boot 如何实现热部署 ?

Spring Boot 实现热部署其实很容易，引入 devtools 依赖即可，这样当编译文件发生变化时，Spring Boot 就会自动重启。在 Eclipse 中，用户按下保存按键，就会自动编译进而重启 Spring Boot，IDEA 中由于是自动保存的，自动保存时并未编译，所以需要开发者按下 Ctrl+F9 进行编译，编译完成后，项目就自动重启了。

如果仅仅只是页面模板发生变化，Java 类并未发生变化，此时可以不用重启 Spring Boot，使用 LiveReload 插件就可以轻松实现热部署。

- 10.Spring Boot 中如何实现定时任务 ?

定时任务也是一个常见的需求，Spring Boot 中对于定时任务的支持主要还是来自 Spring 框架。

在 Spring Boot 中使用定时任务主要有两种不同的方式，一个就是使用 Spring 中的 @Scheduled 注解，另一个则是使用第三方框架 Quartz。

使用 Spring 中的 @Scheduled 的方式主要通过 @Scheduled 注解来实现。

使用 Quartz ，则按照 Quartz 的方式，定义 Job 和 Trigger 即可。

关于定时任务这一块，大家可以参考：[Spring Boot 中实现定时任务的两种方式!](https://mp.weixin.qq.com/s/_20RYBkjKrB4tdpXI3hBOA)

- 11.前后端分离，如何维护接口文档 ?

前后端分离开发日益流行，大部分情况下，我们都是通过 Spring Boot 做前后端分离开发，前后端分离一定会有接口文档，不然会前后端会深深陷入到扯皮中。一个比较笨的方法就是使用 word 或者 md 来维护接口文档，但是效率太低，接口一变，所有人手上的文档都得变。在 Spring Boot 中，这个问题常见的解决方案是 Swagger ，使用 Swagger 我们可以快速生成一个接口文档网站，接口一旦发生变化，文档就会自动更新，所有开发工程师访问这一个在线网站就可以获取到最新的接口文档，非常方便。关于 Swagger 的用法，大家可以参考：[SpringBoot整合Swagger2，再也不用维护接口文档了！](https://mp.weixin.qq.com/s/iTsTqEeqT9K84S091ycdog)

- 12.什么是 Spring Data ?

Spring Data 是 Spring 的一个子项目。用于简化数据库访问，支持NoSQL 和 关系数据存储。其主要目标是使数据库的访问变得方便快捷。Spring Data 具有如下特点：

1. SpringData 项目支持 NoSQL 存储：
2. MongoDB （文档数据库）
3. Neo4j（图形数据库）
4. Redis（键/值存储）
5. Hbase（列族数据库）

SpringData 项目所支持的关系数据存储技术：

1. JDBC
2. JPA

Spring Data Jpa 致力于减少数据访问层 (DAO) 的开发量. 开发者唯一要做的，就是声明持久层的接口，其他都交给 Spring Data JPA 来帮你完成！Spring Data JPA 通过规范方法的名字，根据符合规范的名字来确定方法需要实现什么样的逻辑。

- 13.Spring Boot 是否可以使用 XML 配置 ?

Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 @ImportResource 注解可以引入一个 XML 配置。

- 14.Spring Boot 打成的 jar 和普通的 jar 有什么区别 ?

Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 `java -jar xxx.jar` 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。

Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 `\BOOT-INF\classes` 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

- 15.bootstrap.properties 和 application.properties 有何区别 ?

单纯做 Spring Boot 开发，可能不太容易遇到 bootstrap.properties 配置文件，但是在结合 Spring Cloud 时，这个配置就会经常遇到了，特别是在需要加载一些远程配置文件的时侯。

bootstrap.properties 在 application.properties 之前加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。bootstrap.properties 被 Spring ApplicationContext 的父类加载，这个类先于加载 application.properties 的 ApplicatonContext 启动。

当然，前面叙述中的 properties 也可以修改为 yaml 。

好了，本文就说到这里，欢迎小伙伴留言说说你曾经遇到过的 Spring Boot 面试题！