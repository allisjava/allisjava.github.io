---
sidebar:
  nav: docs-zh
title: Spring Boot 邮件发送的 5 种姿势！
date: 2019-07-17 08:30:51
tags: [Spring Boot,Mail]
categories: Spring Boot
abbrlink: springboot-mail
---

邮件发送其实是一个非常常见的需求，用户注册，找回密码等地方，都会用到，使用 JavaSE 代码发送邮件，步骤还是挺繁琐的，Spring Boot 中对于邮件发送，提供了相关的自动化配置类，使得邮件发送变得非常容易，本文我们就来一探究竟！看看使用 Spring Boot 发送邮件的 5 中姿势。

<!--more-->

# 邮件基础

我们经常会听到各种各样的邮件协议，比如 SMTP、POP3、IMAP ，那么这些协议有什么作用，有什么区别？我们先来讨论一下这个问题。  

SMTP 是一个基于 TCP/IP 的应用层协议，江湖地位有点类似于 HTTP，SMTP 服务器默认监听的端口号为 25 。看到这里，小伙伴们可能会想到既然 SMTP 协议是基于 TCP/IP 的应用层协议，那么我是不是也可以通过 Socket 发送一封邮件呢？回答是肯定的。  

生活中我们投递一封邮件要经过如下几个步骤：  

1. 深圳的小王先将邮件投递到深圳的邮局  
2. 深圳的邮局将邮件运送到上海的邮局  
3. 上海的小张来邮局取邮件   

这是一个缩减版的生活中邮件发送过程。这三个步骤可以分别对应我们的邮件发送过程，假设从 aaa@qq.com 发送邮件到 111@163.com ：  

1. aaa@qq.com 先将邮件投递到腾讯的邮件服务器  
2. 腾讯的邮件服务器将我们的邮件投递到网易的邮件服务器  
3. 111@163.com 登录网易的邮件服务器查看邮件  

邮件投递大致就是这个过程，这个过程就涉及到了多个协议，我们来分别看一下。  

SMTP 协议全称为 Simple Mail Transfer Protocol，译作简单邮件传输协议，它定义了邮件客户端软件与 SMTP 服务器之间，以及 SMTP 服务器与 SMTP 服务器之间的通信规则。

也就是说 aaa@qq.com 用户先将邮件投递到腾讯的 SMTP 服务器这个过程就使用了 SMTP 协议，然后腾讯的 SMTP 服务器将邮件投递到网易的 SMTP 服务器这个过程也依然使用了 SMTP 协议，SMTP 服务器就是用来收邮件。

而 POP3 协议全称为 Post Office Protocol ，译作邮局协议，它定义了邮件客户端与 POP3 服务器之间的通信规则，那么该协议在什么场景下会用到呢？当邮件到达网易的 SMTP 服务器之后， 111@163.com 用户需要登录服务器查看邮件，这个时候就该协议就用上了：邮件服务商都会为每一个用户提供专门的邮件存储空间，SMTP 服务器收到邮件之后，就将邮件保存到相应用户的邮件存储空间中，如果用户要读取邮件，就需要通过邮件服务商的 POP3 邮件服务器来完成。

最后，可能也有小伙伴们听说过 IMAP 协议，这个协议是对 POP3 协议的扩展，功能更强，作用类似，这里不再赘述。  
 
# 准备工作

目前国内大部分的邮件服务商都不允许直接使用用户名/密码的方式来在代码中发送邮件，都是要先申请授权码，这里以 QQ 邮箱为例，向大家演示授权码的申请流程：首先我们需要先登录 QQ 邮箱网页版，点击上方的设置按钮：  

![p266](http://www.javaboy.org/images/mail/24-1.jpg)  

然后点击账户选项卡：  

![p267](http://www.javaboy.org/images/mail/24-2.jpg)  

在账户选项卡中找到开启POP3/SMTP选项，如下：  

![p268](http://www.javaboy.org/images/mail/24-3.jpg)  

点击开启，开启相关功能，开启过程需要手机号码验证，按照步骤操作即可，不赘述。开启成功之后，即可获取一个授权码，将该号码保存好，一会使用。

# 项目创建

接下来，我们就可以创建项目了，Spring Boot 中，对于邮件发送提供了自动配置类，开发者只需要加入相关依赖，然后配置一下邮箱的基本信息，就可以发送邮件了。

- 首先创建一个 Spring Boot 项目，引入邮件发送依赖：

![](http://www.javaboy.org/images/mail/24-4.png)

创建完成后，项目依赖如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 配置邮箱基本信息

项目创建成功后，接下来在 application.properties 中配置邮箱的基本信息：

```properties
spring.mail.host=smtp.qq.com
spring.mail.port=587
spring.mail.username=1510161612@qq.com
spring.mail.password=ubknfzhjkhrbbabe
spring.mail.default-encoding=UTF-8
spring.mail.properties.mail.smtp.socketFactoryClass=javax.net.ssl.SSLSocketFactory
spring.mail.properties.mail.debug=true
```

配置含义分别如下：

- 配置 SMTP 服务器地址
- SMTP 服务器的端口
- 配置邮箱用户名
- 配置密码，注意，不是真正的密码，而是刚刚申请到的授权码
- 默认的邮件编码
- 配饰 SSL 加密工厂
- 表示开启 DEBUG 模式，这样，邮件发送过程的日志会在控制台打印出来，方便排查错误

如果不知道 smtp 服务器的端口或者地址的的话，可以参考 腾讯的邮箱文档

- https://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=371

做完这些之后，Spring Boot 就会自动帮我们配置好邮件发送类，相关的配置在 `org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration` 类中，部分源码如下：

```java
@Configuration
@ConditionalOnClass({ MimeMessage.class, MimeType.class, MailSender.class })
@ConditionalOnMissingBean(MailSender.class)
@Conditional(MailSenderCondition.class)
@EnableConfigurationProperties(MailProperties.class)
@Import({ MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class })
public class MailSenderAutoConfiguration {
}
```

从这段代码中，可以看到，导入了另外一个配置 `MailSenderPropertiesConfiguration` 类，这个类中，提供了邮件发送相关的工具类：

```java
@Configuration
@ConditionalOnProperty(prefix = "spring.mail", name = "host")
class MailSenderPropertiesConfiguration {
        private final MailProperties properties;
        MailSenderPropertiesConfiguration(MailProperties properties) {
                this.properties = properties;
        }
        @Bean
        @ConditionalOnMissingBean
        public JavaMailSenderImpl mailSender() {
                JavaMailSenderImpl sender = new JavaMailSenderImpl();
                applyProperties(sender);
                return sender;
        }
}
```

可以看到，这里创建了一个 `JavaMailSenderImpl` 的实例， `JavaMailSenderImpl` 是 `JavaMailSender` 的一个实现，我们将使用 `JavaMailSenderImpl` 来完成邮件的发送工作。

做完如上两步，邮件发送的准备工作就算是完成了，接下来就可以直接发送邮件了。

具体的发送，有 5 种不同的方式，我们一个一个来看。

## 发送简单邮件

简单邮件就是指邮件内容是一个普通的文本文档：

```java
@Autowired
JavaMailSender javaMailSender;
@Test
public void sendSimpleMail() {
    SimpleMailMessage message = new SimpleMailMessage();
    message.setSubject("这是一封测试邮件");
    message.setFrom("1510161612@qq.com");
    message.setTo("25xxxxx755@qq.com");
    message.setCc("37xxxxx37@qq.com");
    message.setBcc("14xxxxx098@qq.com");
    message.setSentDate(new Date());
    message.setText("这是测试邮件的正文");
    javaMailSender.send(message);
}
```

从上往下，代码含义分别如下：

1. 构建一个邮件对象
2. 设置邮件主题
3. 设置邮件发送者
4. 设置邮件接收者，可以有多个接收者
5. 设置邮件抄送人，可以有多个抄送人
6. 设置隐秘抄送人，可以有多个
7. 设置邮件发送日期
8. 设置邮件的正文
9. 发送邮件

最后执行该方法，就可以实现邮件的发送，发送效果图如下：

![](http://www.javaboy.org/images/mail/24-5.png)

## 发送带附件的邮件

邮件的附件可以是图片，也可以是普通文件，都是支持的。

```java
@Test
public void sendAttachFileMail() throws MessagingException {
    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true);
    helper.setSubject("这是一封测试邮件");
    helper.setFrom("1510161612@qq.com");
    helper.setTo("25xxxxx755@qq.com");
    helper.setCc("37xxxxx37@qq.com");
    helper.setBcc("14xxxxx098@qq.com");
    helper.setSentDate(new Date());
    helper.setText("这是测试邮件的正文");
    helper.addAttachment("javaboy.jpg",new File("C:\\Users\\sang\\Downloads\\javaboy.png"));
    javaMailSender.send(mimeMessage);
}
```

注意这里在构建邮件对象上和前文有所差异，这里是通过 javaMailSender 来获取一个复杂邮件对象，然后再利用 MimeMessageHelper 对邮件进行配置，MimeMessageHelper 是一个邮件配置的辅助工具类，创建时候的 true 表示构建一个 multipart message 类型的邮件，有了 MimeMessageHelper 之后，我们针对邮件的配置都是由 MimeMessageHelper 来代劳。

最后通过 addAttachment 方法来添加一个附件。

执行该方法，邮件发送效果图如下：

![](http://www.javaboy.org/images/mail/24-6.png)

## 发送带图片资源的邮件

图片资源和附件有什么区别呢？图片资源是放在邮件正文中的，即一打开邮件，就能看到图片。但是一般来说，不建议使用这种方式，一些公司会对邮件内容的大小有限制（因为这种方式是将图片一起发送的）。

```java
@Test
public void sendImgResMail() throws MessagingException {
    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
    helper.setSubject("这是一封测试邮件");
    helper.setFrom("1510161612@qq.com");
    helper.setTo("25xxxxx755@qq.com");
    helper.setCc("37xxxxx37@qq.com");
    helper.setBcc("14xxxxx098@qq.com");
    helper.setSentDate(new Date());
    helper.setText("<p>hello 大家好，这是一封测试邮件，这封邮件包含两种图片，分别如下</p><p>第一张图片：</p><img src='cid:p01'/><p>第二张图片：</p><img src='cid:p02'/>",true);
    helper.addInline("p01",new FileSystemResource(new File("C:\\Users\\sang\\Downloads\\javaboy.png")));
    helper.addInline("p02",new FileSystemResource(new File("C:\\Users\\sang\\Downloads\\javaboy2.png")));
    javaMailSender.send(mimeMessage);
}
```

这里的邮件 text 是一个 HTML 文本，里边涉及到的图片资源先用一个占位符占着，setText 方法的第二个参数 true 表示第一个参数是一个 HTML 文本。

setText 之后，再通过 addInline 方法来添加图片资源。

最后执行该方法，发送邮件，效果如下：

![](http://www.javaboy.org/images/mail/24-7.png)

在公司实际开发中，第一种和第三种都不是使用最多的邮件发送方案。因为正常来说，邮件的内容都是比较的丰富的，所以大部分邮件都是通过 HTML 来呈现的，如果直接拼接 HTML 字符串，这样以后不好维护，为了解决这个问题，一般邮件发送，都会有相应的邮件模板。最具代表性的两个模板就是 `Freemarker` 模板和 `Thyemeleaf` 模板了。

## 使用 Freemarker 作邮件模板

首先需要引入 Freemarker 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

然后在 `resources/templates` 目录下创建一个 `mail.ftl` 作为邮件发送模板：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p>hello 欢迎加入 xxx 大家庭，您的入职信息如下：</p>
<table border="1">
    <tr>
        <td>姓名</td>
        <td>${username}</td>
    </tr>
    <tr>
        <td>工号</td>
        <td>${num}</td>
    </tr>
    <tr>
        <td>薪水</td>
        <td>${salary}</td>
    </tr>
</table>
<div style="color: #ff1a0e">一起努力创造辉煌</div>
</body>
</html>
```

接下来，将邮件模板渲染成 HTML ，然后发送即可。

```java
@Test
public void sendFreemarkerMail() throws MessagingException, IOException, TemplateException {
    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
    helper.setSubject("这是一封测试邮件");
    helper.setFrom("1510161612@qq.com");
    helper.setTo("25xxxxx755@qq.com");
    helper.setCc("37xxxxx37@qq.com");
    helper.setBcc("14xxxxx098@qq.com");
    helper.setSentDate(new Date());
    //构建 Freemarker 的基本配置
    Configuration configuration = new Configuration(Configuration.VERSION_2_3_0);
    // 配置模板位置
    ClassLoader loader = MailApplication.class.getClassLoader();
    configuration.setClassLoaderForTemplateLoading(loader, "templates");
    //加载模板
    Template template = configuration.getTemplate("mail.ftl");
    User user = new User();
    user.setUsername("javaboy");
    user.setNum(1);
    user.setSalary((double) 99999);
    StringWriter out = new StringWriter();
    //模板渲染，渲染的结果将被保存到 out 中 ，将out 中的 html 字符串发送即可
    template.process(user, out);
    helper.setText(out.toString(),true);
    javaMailSender.send(mimeMessage);
}
```

需要注意的是，虽然引入了 `Freemarker` 的自动化配置，但是我们在这里是直接 `new Configuration` 来重新配置 `Freemarker` 的，所以 Freemarker 默认的配置这里不生效，因此，在填写模板位置时，值为 `templates` 。

调用该方法，发送邮件，效果图如下：

![](http://www.javaboy.org/images/mail/24-8.png)

## 使用 Thymeleaf 作邮件模板

推荐在 Spring Boot 中使用 Thymeleaf 来构建邮件模板。因为 Thymeleaf 的自动化配置提供了一个 TemplateEngine，通过 TemplateEngine 可以方便的将 Thymeleaf 模板渲染为 HTML ，同时，Thymeleaf 的自动化配置在这里是继续有效的 。

首先，引入 Thymeleaf 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

然后，创建 `Thymeleaf` 邮件模板：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p>hello 欢迎加入 xxx 大家庭，您的入职信息如下：</p>
<table border="1">
    <tr>
        <td>姓名</td>
        <td th:text="${username}"></td>
    </tr>
    <tr>
        <td>工号</td>
        <td th:text="${num}"></td>
    </tr>
    <tr>
        <td>薪水</td>
        <td th:text="${salary}"></td>
    </tr>
</table>
<div style="color: #ff1a0e">一起努力创造辉煌</div>
</body>
</html>
```

接下来发送邮件：

```java
@Autowired
TemplateEngine templateEngine;

@Test
public void sendThymeleafMail() throws MessagingException {
    MimeMessage mimeMessage = javaMailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
    helper.setSubject("这是一封测试邮件");
    helper.setFrom("1510161612@qq.com");
    helper.setTo("25xxxxx755@qq.com");
    helper.setCc("37xxxxx37@qq.com");
    helper.setBcc("14xxxxx098@qq.com");
    helper.setSentDate(new Date());
    Context context = new Context();
    context.setVariable("username", "javaboy");
    context.setVariable("num","000001");
    context.setVariable("salary", "99999");
    String process = templateEngine.process("mail.html", context);
    helper.setText(process,true);
    javaMailSender.send(mimeMessage);
}
```

调用该方法，发送邮件，效果图如下：

![](http://www.javaboy.org/images/mail/24-8.png)

好了，这就是我们今天说的 5 种邮件发送姿势，不知道你掌握了没有呢？

本文案例已经上传到 GitHub：[https://github.com/lenve/javaboy-code-samples](https://github.com/lenve/javaboy-code-samples)。

有问题欢迎留言讨论。