---
sidebar:
  nav: docs-zh
title: Spring Boot数据持久化之JdbcTemplate
tags:
  - Spring Boot
  - JdbcTemplate
categories: 'Spring Boot'
abbrlink: jdbctemplate
date: 2019-04-06 22:28:52
---

在Java领域，数据持久化有几个常见的方案，有Spring自带的JdbcTemplate、有MyBatis，还有JPA，在这些方案中，最简单的就是Spring自带的JdbcTemplate了，这个东西虽然没有MyBatis那么方便，但是比起最开始的Jdbc已经强了很多了，它没有MyBatis功能那么强大，当然也意味着它的使用比较简单，事实上，JdbcTemplate算是最简单的数据持久化方案了，本文就和大伙来说说这个东西的使用。  

 <!-- more -->
 
## 基本配置

JdbcTemplate基本用法实际上很简单，开发者在创建一个SpringBoot项目时，除了选择基本的Web依赖，再记得选上Jdbc依赖，以及数据库驱动依赖即可，如下：  

![](http://www.javaboy.org/images/sb/1-1.png)  

项目创建成功之后，记得添加Druid数据库连接池依赖（注意这里可以添加专门为Spring Boot打造的druid-spring-boot-starter，而不是我们一般在SSM中添加的Druid），所有添加的依赖如下：  

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
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
    <version>5.1.27</version>
    <scope>runtime</scope>
</dependency>
```


项目创建完后，接下来只需要在application.properties中提供数据的基本配置即可，如下：  


```
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.username=root
spring.datasource.password=123
spring.datasource.url=jdbc:mysql:///test01?useUnicode=true&characterEncoding=UTF-8
```

如此之后，所有的配置就算完成了，接下来就可以直接使用JdbcTemplate了？咋这么方便呢？其实这就是SpringBoot的自动化配置带来的好处，我们先说用法，一会来说原理。  

## 基本用法

首先我们来创建一个User Bean，如下：  

```java
public class User {
    private Long id;
    private String username;
    private String address;
    //省略getter/setter
}
```  

然后来创建一个UserService类，在UserService类中注入JdbcTemplate，如下：  

```java
@Service
public class UserService {
    @Autowired
    JdbcTemplate jdbcTemplate;
}
```  

好了，如此之后，准备工作就算完成了。  

### 增

JdbcTemplate中，除了查询有几个API之外，增删改统一都使用update来操作，自己来传入SQL即可。例如添加数据，方法如下：  

```java
public int addUser(User user) {
    return jdbcTemplate.update("insert into user (username,address) values (?,?);", user.getUsername(), user.getAddress());
}
```  

update方法的返回值就是SQL执行受影响的行数。  

这里只需要传入SQL即可，如果你的需求比较复杂，例如在数据插入的过程中希望实现主键回填，那么可以使用PreparedStatementCreator，如下：  

```java
public int addUser2(User user) {
    KeyHolder keyHolder = new GeneratedKeyHolder();
    int update = jdbcTemplate.update(new PreparedStatementCreator() {
        @Override
        public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
            PreparedStatement ps = connection.prepareStatement("insert into user (username,address) values (?,?);", Statement.RETURN_GENERATED_KEYS);
            ps.setString(1, user.getUsername());
            ps.setString(2, user.getAddress());
            return ps;
        }
    }, keyHolder);
    user.setId(keyHolder.getKey().longValue());
    System.out.println(user);
    return update;
}
```  

实际上这里就相当于完全使用了JDBC中的解决方案了，首先在构建PreparedStatement时传入Statement.RETURN_GENERATED_KEYS，然后传入KeyHolder，最终从KeyHolder中获取刚刚插入数据的id保存到user对象的id属性中去。  

你能想到的JDBC的用法，在这里都能实现，Spring提供的JdbcTemplate虽然不如MyBatis，但是比起Jdbc还是要方便很多的。  

### 删

删除也是使用update API，传入你的SQL即可：  

```java
public int deleteUserById(Long id) {
    return jdbcTemplate.update("delete from user where id=?", id);
}
```  

当然你也可以使用PreparedStatementCreator。  

### 改

```java
public int updateUserById(User user) {
    return jdbcTemplate.update("update user set username=?,address=? where id=?", user.getUsername(), user.getAddress(),user.getId());
}
```  

当然你也可以使用PreparedStatementCreator。  

### 查

查询的话，稍微有点变化，这里主要向大伙介绍query方法，例如查询所有用户：  

```java
public List<User> getAllUsers() {
    return jdbcTemplate.query("select * from user", new RowMapper<User>() {
        @Override
        public User mapRow(ResultSet resultSet, int i) throws SQLException {
            String username = resultSet.getString("username");
            String address = resultSet.getString("address");
            long id = resultSet.getLong("id");
            User user = new User();
            user.setAddress(address);
            user.setUsername(username);
            user.setId(id);
            return user;
        }
    });
}
```  

查询的时候需要提供一个RowMapper，就是需要自己手动映射，将数据库中的字段和对象的属性一一对应起来，这样。。。。嗯看起来有点麻烦，实际上，如果数据库中的字段和对象属性的名字一模一样的话，有另外一个简单的方案，如下：  

```java
public List<User> getAllUsers2() {
    return jdbcTemplate.query("select * from user", new BeanPropertyRowMapper<>(User.class));
}
```  

至于查询时候传参也是使用占位符，这个和前文的一致，这里不再赘述。  

###  其他

除了这些基本用法之外，JdbcTemplate也支持其他用法，例如调用存储过程等，这些都比较容易，而且和Jdbc本身都比较相似，这里也就不做介绍了，有兴趣可以留言讨论。  

## 原理分析

那么在SpringBoot中，配置完数据库基本信息之后，就有了一个JdbcTemplate了，这个东西是从哪里来的呢？源码在```org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration```类中，该类源码如下：  

```java
@Configuration
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties(JdbcProperties.class)
public class JdbcTemplateAutoConfiguration {

	@Configuration
	static class JdbcTemplateConfiguration {

		private final DataSource dataSource;

		private final JdbcProperties properties;

		JdbcTemplateConfiguration(DataSource dataSource, JdbcProperties properties) {
			this.dataSource = dataSource;
			this.properties = properties;
		}

		@Bean
		@Primary
		@ConditionalOnMissingBean(JdbcOperations.class)
		public JdbcTemplate jdbcTemplate() {
			JdbcTemplate jdbcTemplate = new JdbcTemplate(this.dataSource);
			JdbcProperties.Template template = this.properties.getTemplate();
			jdbcTemplate.setFetchSize(template.getFetchSize());
			jdbcTemplate.setMaxRows(template.getMaxRows());
			if (template.getQueryTimeout() != null) {
				jdbcTemplate
						.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
			}
			return jdbcTemplate;
		}

	}

	@Configuration
	@Import(JdbcTemplateConfiguration.class)
	static class NamedParameterJdbcTemplateConfiguration {

		@Bean
		@Primary
		@ConditionalOnSingleCandidate(JdbcTemplate.class)
		@ConditionalOnMissingBean(NamedParameterJdbcOperations.class)
		public NamedParameterJdbcTemplate namedParameterJdbcTemplate(
				JdbcTemplate jdbcTemplate) {
			return new NamedParameterJdbcTemplate(jdbcTemplate);
		}

	}

}
```  

从这个类中，大致可以看出，当当前类路径下存在DataSource和JdbcTemplate时，该类就会被自动配置，jdbcTemplate方法则表示，如果开发者没有自己提供一个JdbcOperations的实例的话，系统就自动配置一个JdbcTemplate Bean（JdbcTemplate是JdbcOperations接口的一个实现）。好了，不知道大伙有没有收获呢？  

关于JdbcTemplate，我还有一个小小视频，加入我的知识星球，免费观看：  

![](http://www.javaboy.org/images/sb/1-2.png)  

加入我的星球，和众多大牛一起切磋技术[推荐一个技术圈子，Java技能提升就靠它了](https://mp.weixin.qq.com/s/hAoDC77itM7IpixxHriqUw)。  