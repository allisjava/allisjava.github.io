---
sidebar:
  nav: docs-zh
title: Spring Boot整合Jpa多数据源
tags:
  - Spring Boot
  - Jpa
categories: Spring Boot
abbrlink: springboot-jpa-multi
date: 2019-04-07 21:34:32
---
本文是Spring Boot整合数据持久化方案的最后一篇，主要和大伙来聊聊Spring Boot整合Jpa多数据源问题。在Spring Boot整合JbdcTemplate多数据源、Spring Boot整合MyBatis多数据源以及Spring Boot整合Jpa多数据源这三个知识点中，整合Jpa多数据源算是最复杂的一种，也是很多人在配置时最容易出错的一种。本文大伙就跟着松哥的教程，一步一步整合Jpa多数据源。  

 <!-- more -->
 
## 工程创建  

首先是创建一个Spring Boot工程，创建时添加基本的Web、Jpa以及MySQL依赖，如下：  

![](http://www.javaboy.org/images/sb/5-1.png)  

创建完成后，添加Druid依赖，这里和前文的要求一样，要使用专为Spring Boot打造的Druid，大伙可能发现了，如果整合多数据源一定要使用这个依赖，因为这个依赖中才有DruidDataSourceBuilder，最后还要记得锁定数据库依赖的版本，因为可能大部分人用的还是5.x的MySQL而不是8.x。完整依赖如下：  

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.28</version>
    <scope>runtime</scope>
</dependency>
```

如此之后，工程就创建成功了。  

## 基本配置  

在基本配置中，我们首先来配置多数据源基本信息以及DataSource，首先在application.properties中添加如下配置信息：  


```
#  数据源一
spring.datasource.one.username=root
spring.datasource.one.password=root
spring.datasource.one.url=jdbc:mysql:///test01?useUnicode=true&characterEncoding=UTF-8
spring.datasource.one.type=com.alibaba.druid.pool.DruidDataSource

#  数据源二
spring.datasource.two.username=root
spring.datasource.two.password=root
spring.datasource.two.url=jdbc:mysql:///test02?useUnicode=true&characterEncoding=UTF-8
spring.datasource.two.type=com.alibaba.druid.pool.DruidDataSource

# Jpa配置
spring.jpa.properties.database=mysql
spring.jpa.properties.show-sql=true
spring.jpa.properties.database-platform=mysql
spring.jpa.properties.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```


这里Jpa的配置和上文相比key中多了properties，多数据源的配置和前文一致，然后接下来配置两个DataSource，如下：  


```java
@Configuration
public class DataSourceConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.one")
    @Primary
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


这里的配置和前文的多数据源配置基本一致，但是注意多了一个在Spring中使用较少的注解@Primary，这个注解一定不能少，否则在项目启动时会出错，@Primary表示当某一个类存在多个实例时，优先使用哪个实例。  

好了，这样，DataSource就有了。  

## 多数据源配置  

接下来配置Jpa的基本信息，这里两个数据源，我分别在两个类中来配置，先来看第一个配置：  


```
@Configuration
@EnableJpaRepositories(basePackages = "org.sang.jpa.dao",entityManagerFactoryRef = "localContainerEntityManagerFactoryBeanOne",transactionManagerRef = "platformTransactionManagerOne")
public class JpaConfigOne {
    @Autowired
    @Qualifier(value = "dsOne")
    DataSource dsOne;

    @Autowired
    JpaProperties jpaProperties;


    @Bean
    @Primary
    LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBeanOne(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dsOne)
                .packages("org.sang.jpa.model")
                .properties(jpaProperties.getProperties())
                .persistenceUnit("pu1")
                .build();
    }

    @Bean
    PlatformTransactionManager platformTransactionManagerOne(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean factoryBeanOne = localContainerEntityManagerFactoryBeanOne(builder);
        return new JpaTransactionManager(factoryBeanOne.getObject());
    }
}
```

首先这里注入dsOne，再注入JpaProperties，JpaProperties是系统提供的一个实例，里边的数据就是我们在application.properties中配置的jpa相关的配置。然后我们提供两个Bean，分别是LocalContainerEntityManagerFactoryBean和PlatformTransactionManager事务管理器，不同于MyBatis和JdbcTemplate，在Jpa中，事务一定要配置。在提供LocalContainerEntityManagerFactoryBean的时候，需要指定packages，这里的packages指定的包就是这个数据源对应的实体类所在的位置，另外在这里配置类上通过@EnableJpaRepositories注解指定dao所在的位置，以及LocalContainerEntityManagerFactoryBean和PlatformTransactionManager分别对应的引用的名字。  


好了，这样第一个就配置好了，第二个基本和这个类似，主要有几个不同点：  

- dao的位置不同
- persistenceUnit不同
- 相关bean的名称不同

注意实体类可以共用。  

代码如下：  

```java
@Configuration
@EnableJpaRepositories(basePackages = "org.sang.jpa.dao2",entityManagerFactoryRef = "localContainerEntityManagerFactoryBeanTwo",transactionManagerRef = "platformTransactionManagerTwo")
public class JpaConfigTwo {
    @Autowired
    @Qualifier(value = "dsTwo")
    DataSource dsTwo;

    @Autowired
    JpaProperties jpaProperties;


    @Bean
    LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBeanTwo(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(dsTwo)
                .packages("org.sang.jpa.model")
                .properties(jpaProperties.getProperties())
                .persistenceUnit("pu2")
                .build();
    }

    @Bean
    PlatformTransactionManager platformTransactionManagerTwo(EntityManagerFactoryBuilder builder) {
        LocalContainerEntityManagerFactoryBean factoryBeanTwo = localContainerEntityManagerFactoryBeanTwo(builder);
        return new JpaTransactionManager(factoryBeanTwo.getObject());
    }
}
```


接下来，在对应位置分别提供相关的实体类和dao即可，数据源一的dao如下：  

```
package org.sang.jpa.dao;

public interface UserDao extends JpaRepository<User,Integer> {
    List<User> getUserByAddressEqualsAndIdLessThanEqual(String address, Integer id);

    @Query(value = "select * from t_user where id=(select max(id) from t_user)",nativeQuery = true)
    User maxIdUser();
}
```


数据源二的dao如下：  

```
package org.sang.jpa.dao2;

public interface UserDao2 extends JpaRepository<User,Integer> {
    List<User> getUserByAddressEqualsAndIdLessThanEqual(String address, Integer id);

    @Query(value = "select * from t_user where id=(select max(id) from t_user)",nativeQuery = true)
    User maxIdUser();
}
```


共同的实体类如下：  


```
package org.sang.jpa.model;

@Entity(name = "t_user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String username;
    private String address;
    //省略getter/setter
}
```


到此，所有的配置就算完成了，接下来就可以在Service中注入不同的UserDao，不同的UserDao操作不同的数据源。  

其实整合Jpa多数据源也不算难，就是有几个细节问题，这些细节问题解决，其实前面介绍的其他多数据源整个都差不多。  

好了，欢迎大家加入松哥的星球，关于我的星球【Java达摩院】，大伙可以参考这篇文章[推荐一个技术圈子，Java技能提升就靠它了](https://mp.weixin.qq.com/s/hAoDC77itM7IpixxHriqUw).  
