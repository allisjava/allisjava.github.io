---
title: Spring Boot 整合 Freemarker，50 多行配置是怎么省略掉的？
date: 2019-07-05 12:14:59
tags: [SpringBoot,Freemarker]
categories: Spring Boot
abbrlink: springboot-freemarker
---

Spring Boot2 系列教程接近完工，最近进入修修补补阶段。Freemarker 整合貌似还没和大家聊过，因此今天把这个补充上。

<!--more-->

已经完工的 Spring Boot2 教程，大家可以参考这里：

1. [干货\|最新版 Spring Boot2.1.5 教程+案例合集](https://mp.weixin.qq.com/s/YXBFFtWvSwR6dVLbaGDxcQ)

# Freemarker 简介

这是一个相当老牌的开源的免费的模版引擎。通过 Freemarker 模版，我们可以将数据渲染成 HTML 网页、电子邮件、配置文件以及源代码等。Freemarker 不是面向最终用户的，而是一个 Java 类库，我们可以将之作为一个普通的组件嵌入到我们的产品中。

来看一张来自 Freemarker 官网的图片：

![](http://www.javaboy.org/images/boot/22-1.png)

可以看到，Freemarker 可以将模版和数据渲染成 HTML 。

Freemarker 模版后缀为 `.ftl`(FreeMarker Template Language)。FTL 是一种简单的、专用的语言，它不是像 Java 那样成熟的编程语言。在模板中，你可以专注于如何展现数据， 而在模板之外可以专注于要展示什么数据。

好了，这是一个简单的介绍，接下来我们来看看 Freemarker 和 Spring Boot 的一个整合操作。

# 实践

在 SSM 中整合 Freemarker ，所有的配置文件加起来，前前后后大约在 50 行左右，Spring Boot 中要几行配置呢？ 0 行！

## 1.创建工程

首先创建一个 Spring Boot 工程，引入 Freemarker 依赖，如下图：

![](http://www.javaboy.org/images/boot/22-2.png)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

工程创建完成后，在 `org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration` 类中，可以看到关于 Freemarker 的自动化配置：

```java
@Configuration
@ConditionalOnClass({ freemarker.template.Configuration.class, FreeMarkerConfigurationFactory.class })
@EnableConfigurationProperties(FreeMarkerProperties.class)
@Import({ FreeMarkerServletWebConfiguration.class, FreeMarkerReactiveWebConfiguration.class,
                FreeMarkerNonWebConfiguration.class })
public class FreeMarkerAutoConfiguration {
}
```

从这里可以看出，当 `classpath` 下存在 `freemarker.template.Configuration` 以及 `FreeMarkerConfigurationFactory` 时，配置才会生效，也就是说当我们引入了 `Freemarker` 之后，配置就会生效。但是这里的自动化配置只做了模板位置检查，其他配置则是在导入的 `FreeMarkerServletWebConfiguration` 配置中完成的。那么我们再来看看 `FreeMarkerServletWebConfiguration` 类，部分源码如下：

```java
@Configuration
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass({ Servlet.class, FreeMarkerConfigurer.class })
@AutoConfigureAfter(WebMvcAutoConfiguration.class)
class FreeMarkerServletWebConfiguration extends AbstractFreeMarkerConfiguration {
        protected FreeMarkerServletWebConfiguration(FreeMarkerProperties properties) {
                super(properties);
        }
        @Bean
        @ConditionalOnMissingBean(FreeMarkerConfig.class)
        public FreeMarkerConfigurer freeMarkerConfigurer() {
                FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
                applyProperties(configurer);
                return configurer;
        }
        @Bean
        @ConditionalOnMissingBean(name = "freeMarkerViewResolver")
        @ConditionalOnProperty(name = "spring.freemarker.enabled", matchIfMissing = true)
        public FreeMarkerViewResolver freeMarkerViewResolver() {
                FreeMarkerViewResolver resolver = new FreeMarkerViewResolver();
                getProperties().applyToMvcViewResolver(resolver);
                return resolver;
        }
}
```

我们来简单看下这段源码：

1. @ConditionalOnWebApplication 表示当前配置在 web 环境下才会生效。
2. ConditionalOnClass 表示当前配置在存在 Servlet 和 FreeMarkerConfigurer 时才会生效。
3. @AutoConfigureAfter 表示当前自动化配置在 WebMvcAutoConfiguration 之后完成。
4. 代码中，主要提供了 FreeMarkerConfigurer 和 FreeMarkerViewResolver。
5. FreeMarkerConfigurer 是 Freemarker 的一些基本配置，例如 templateLoaderPath、defaultEncoding 等
6. FreeMarkerViewResolver 则是视图解析器的基本配置，包含了viewClass、suffix、allowRequestOverride、allowSessionOverride 等属性。

另外还有一点，在这个类的构造方法中，注入了 FreeMarkerProperties：

```java
@ConfigurationProperties(prefix = "spring.freemarker")
public class FreeMarkerProperties extends AbstractTemplateViewResolverProperties {
        public static final String DEFAULT_TEMPLATE_LOADER_PATH = "classpath:/templates/";
        public static final String DEFAULT_PREFIX = "";
        public static final String DEFAULT_SUFFIX = ".ftl";
        /**
         * Well-known FreeMarker keys which are passed to FreeMarker's Configuration.
         */
        private Map<String, String> settings = new HashMap<>();
}
```

FreeMarkerProperties 中则配置了 Freemarker 的基本信息，例如模板位置在 `classpath:/templates/` ，再例如模板后缀为 `.ftl`，那么这些配置我们以后都可以在 application.properties 中进行修改。

如果我们在 SSM 的 XML 文件中自己配置 Freemarker ，也不过就是配置这些东西。现在，这些配置由  FreeMarkerServletWebConfiguration​  帮我们完成了。

## 2.创建类

首先我们来创建一个 User 类，如下：

```java
public class User {
    private Long id;
    private String username;
    private String address;
    //省略 getter/setter
}
```

再来创建 `UserController`：

```java
@Controller
public class UserController {
    @GetMapping("/index")
    public String index(Model model) {
        List<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setId((long) i);
            user.setUsername("javaboy>>>>" + i);
            user.setAddress("www.javaboy.org>>>>" + i);
            users.add(user);
        }
        model.addAttribute("users", users);
        return "index";
    }
}
```

最后在 freemarker 中渲染数据：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table border="1">
    <tr>
        <td>用户编号</td>
        <td>用户名称</td>
        <td>用户地址</td>
    </tr>
    <#list users as user>
        <tr>
            <td>${user.id}</td>
            <td>${user.username}</td>
            <td>${user.address}</td>
        </tr>
    </#list>
</table>
</body>
</html>
```

运行效果如下：

![](http://www.javaboy.org/images/boot/22-3.png)

# 其他配置

如果我们要修改模版文件位置等，可以在 application.properties 中进行配置：

```properties
spring.freemarker.allow-request-override=false
spring.freemarker.allow-session-override=false
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.check-template-location=true
spring.freemarker.content-type=text/html
spring.freemarker.expose-request-attributes=false
spring.freemarker.expose-session-attributes=false
spring.freemarker.suffix=.ftl
spring.freemarker.template-loader-path=classpath:/templates/
```

配置文件按照顺序依次解释如下：

1. HttpServletRequest的属性是否可以覆盖controller中model的同名项
2. HttpSession的属性是否可以覆盖controller中model的同名项
3. 是否开启缓存
4. 模板文件编码
5. 是否检查模板位置
6. Content-Type的值
7. 是否将HttpServletRequest中的属性添加到Model中
8. 是否将HttpSession中的属性添加到Model中
9. 模板文件后缀
10. 模板文件位置

好了，整合完成之后，Freemarker 的更多用法，就和在 SSM 中使用 Freemarker 一样了，这里我就不再赘述。

# 结语

本文和大家简单聊一聊 Spring Boot 整合 Freemarker，算是对 Spring Boot2 教程的一个补充（后面还会有一些补充），有问题欢迎留言讨论。

本项目案例，我已经上传到 GitHub 上，欢迎大家 star：[https://github.com/lenve/javaboy-code-samples](https://github.com/lenve/javaboy-code-samples)