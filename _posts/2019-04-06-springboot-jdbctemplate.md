---
sidebar:
  nav: docs-zh
title: Spring Boot多数据源配置之JdbcTemplate
categories: Spring Boot
tags:
  - Spring Boot
  - JdbcTemplate
abbrlink: springboot-jdbctemplate
date: 2019-04-06 23:48:29
---
多数据源配置也算是一个常见的开发需求，Spring和SpringBoot中，对此都有相应的解决方案，不过一般来说，如果有多数据源的需求，我还是建议首选分布式数据库中间件MyCat去解决相关问题，之前有小伙伴在我的知识星球上提问，他的数据根据条件的不同，可能保存在四十多个不同的数据库中，怎么办？这种场景下使用多数据源其实就有些费事了，我给的建议是使用MyCat，然后分表策略使用sharding-by-intfile。当然如果一些简单的需求，还是可以使用多数据源的，Spring Boot中，JdbcTemplate、MyBatis以及Jpa都可以配置多数据源，本文就先和大伙聊一聊JdbcTemplate中多数据源的配置（关于JdbcTemplate的用法，如果还有小伙伴不了解，可以参考我的上篇文章[]()）。  

 <!-- more -->
 
## 创建工程

首先是创建工程，和前文一样，创建工程时，也是选择Web、Jdbc以及MySQL驱动，如下图：  

![](http://www.javaboy.org/images/sb/1-1.png)  


创建成功之后，一定接下来手动添加Druid依赖，由于这里一会需要开发者自己配置DataSoruce，所以这里必须要使用`druid-spring-boot-starter`依赖，而不是传统的那个druid依赖，因为`druid-spring-boot-starter`依赖提供了DruidDataSourceBuilder类，这个可以用来构建一个DataSource实例，而传统的Druid则没有该类。完整的依赖如下：  


```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.28</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```




## 配置数据源. 

接下来，在application.properties中配置数据源，不同于上文，这里的数据源需要配置两个，如下： 

 

```java
spring.datasource.one.url=jdbc:mysql:///test01?useUnicode=true&characterEncoding=utf-8
spring.datasource.one.username=root
spring.datasource.one.password=root
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource

spring.datasource.two.url=jdbc:mysql:///test02?useUnicode=true&characterEncoding=utf-8
spring.datasource.two.username=root
spring.datasource.two.password=root
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource
```  

这里通过one和two对数据源进行了区分，但是加了one和two之后，这里的配置就没法被SpringBoot自动加载了（因为前面的key变了），需要我们自己去加载DataSource了，此时，需要自己配置一个DataSourceConfig，用来提供两个DataSource Bean，如下：  

```java
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.one")
    DataSource dsOne() {
        return DruidDataSourceBuilder.create().build();
    }
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.two")
    DataSource dsTwo() {
        return DruidDataSourceBuilder.create().build();
    }
}
```  

这里提供了两个Bean，其中@ConfigurationProperties是Spring Boot提供的类型安全的属性绑定，以第一个Bean为例，`@ConfigurationProperties(prefix = "spring.datasource.one")`表示使用`spring.datasource.one`前缀的数据库配置去创建一个DataSource，这样配置之后，我们就有了两个不同的DataSource，接下来再用这两个不同的DataSource去创建两个不同的JdbcTemplate。  

## 配置JdbcTemplate实例

创建JdbcTemplateConfig类，用来提供两个不同的JdbcTemplate实例，如下：  


```java
@Configuration
public class JdbcTemplateConfig {

    @Bean
    JdbcTemplate jdbcTemplateOne(@Qualifier("dsOne") DataSource dsOne) {
        return new JdbcTemplate(dsOne);
    }

    @Bean
    JdbcTemplate jdbcTemplateTwo(@Qualifier("dsTwo") DataSource dsTwo) {
        return new JdbcTemplate(dsTwo);
    }
}
```  



每一个JdbcTemplate的创建都需要一个DataSource，由于Spring容器中现在存在两个DataSource，默认使用类型查找，会报错，因此加上@Qualifier注解，表示按照名称查找。这里创建了两个JdbcTemplate实例，分别对应了两个DataSource。  

接下来直接去使用这个JdbcTemplate就可以了。  

## 测试使用

关于JdbcTemplate的详细用法大伙可以参考我的上篇文章，这里我主要演示数据源的差异，在Controller中注入两个不同的JdbcTemplate，这两个JdbcTemplate分别对应了不同的数据源，如下：  



```java
@RestController
public class HelloController {
    @Autowired
    @Qualifier("jdbcTemplateOne")
    JdbcTemplate jdbcTemplateOne;
    @Resource(name = "jdbcTemplateTwo")
    JdbcTemplate jdbcTemplateTwo;

    @GetMapping("/user")
    public List<User> getAllUser() {
        List<User> list = jdbcTemplateOne.query("select * from t_user", new BeanPropertyRowMapper<>(User.class));
        return list;
    }
    @GetMapping("/user2")
    public List<User> getAllUser2() {
        List<User> list = jdbcTemplateTwo.query("select * from t_user", new BeanPropertyRowMapper<>(User.class));
        return list;
    }
}
```  


和DataSource一样，Spring容器中的JdbcTemplate也是有两个，因此不能通过byType的方式注入进来，这里给大伙提供了两种注入思路，一种是使用@Resource注解，直接通过byName的方式注入进来，另外一种就是```@Autowired```注解加上```@Qualifier```注解，两者联合起来，实际上也是byName。将JdbcTemplate注入进来之后，jdbcTemplateOne和jdbcTemplateTwo此时就代表操作不同的数据源，使用不同的JdbcTemplate操作不同的数据源，实现了多数据源配置。  

好了，这个问题就先说到这里，关于这个多数据源配置，还有一个小小的视频教程，加入我的星球免费观看：  

![](http://www.javaboy.org/images/sb/2-2.png)  

关于我的星球【Java达摩院】，大伙可以参考这篇文章[推荐一个技术圈子，Java技能提升就靠它了](https://mp.weixin.qq.com/s/hAoDC77itM7IpixxHriqUw).  

![](http://www.javaboy.org/images/sb/star.jpg)  