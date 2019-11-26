---
title: Spring Boot 开发微信公众号后台
date: 2019-10-29 10:32:07
tags: [weixin,Spring Boot]
categories: Spring Boot
abbrlink: springboot-weixin
sidebar:
  nav: docs-zh
---

Hello 各位小伙伴，松哥今天要和大家聊一个有意思的话题，就是使用 Spring Boot 开发微信公众号后台。

<!--more-->

很多小伙伴可能注意到松哥的个人网站（http://www.javaboy.org）前一阵子上线了一个公众号内回复口令解锁网站文章的功能，还有之前就有的公众号内回复口令获取超 2TB 免费视频教程的功能（[免费视频教程](https://mp.weixin.qq.com/s/yOVbTBVk4CJy6a0lrKjLXA)），这两个都是松哥基于 Spring Boot 来做的，最近松哥打算通过一个系列的文章，来向小伙伴们介绍下如何通过 Spring Boot 来开发公众号后台。

## 1. 缘起

今年 5 月份的时候，我想把我自己之前收集到的一些视频教程分享给公众号上的小伙伴，可是这些视频教程大太了，无法一次分享，单次分享分享链接立马就失效了，为了把这些视频分享给大家，我把视频拆分成了很多份，然后设置了不同的口令，小伙伴们在公众号后台通过回复口令就可以获取到这些视频，口令前前后后有 100 多个，我一个一个手动的在微信后台进行配置。这么搞工作量很大，前前后后大概花了三个晚上才把这些东西搞定。

于是我就在想，该写点代码了。

上个月买了服务器，也备案了，该有的都有了，于是就打算把这些资源用代码实现下，因为大学时候搞过公众号开发，倒也没什么难度，于是说干就干。

## 2. 实现思路

其实松哥这个回复口令获取视频链接的实现原理很简单，说白了，就是一个数据查询操作而已，回复的口令是查询关键字，回复的内容则是查询结果。这个原理很简单。

另一方面大家需要明白微信公众号后台开发消息发送的一个流程，大家看下面这张图：

![](http://www.javaboy.org/images/boot/44-2.jpeg)

这是大家在公众号后台回复关键字的情况。那么这个消息是怎么样一个传递流程呢？我们来看看下面这张图：

![](http://www.javaboy.org/images/boot/44-1.png)

这张图，我给大家稍微解释下：

1. 首先 `javaboy4096` 这个字符从公众号上发送到了微信服务器
2. 接下来微信服务器会把 `javaboy4096` 转发到我自己的服务器上
3. 我收到 `javaboy4096` 这个字符之后，就去数据库中查询，将查询的结果，按照腾讯要求的 XML 格式进行返回
4. 微信服务器把从我的服务器收到的信息，再发回到微信上，于是小伙伴们就看到了返回结果了

大致的流程就是这个样子。

接下来我们就来看一下实现细节。

## 3. 公众号后台配置

开发的第一步，是微信服务器要验证我们自己的服务器是否有效。

首先我们登录微信公众平台官网后，在公众平台官网的 **开发-基本设置** 页面，勾选协议成为开发者，然后点击“修改配置”按钮，填写：

- 服务器地址（URL）
- Token
- EncodingAESKey

![](http://www.javaboy.org/images/boot/44-3.jpeg)

这里的 URL 配置好之后，我们需要针对这个 URL 开发两个接口，一个是 GET 请求的接口，这个接口用来做服务器有效性验证，另一个则是 POST 请求的接口，这个用来接收微信服务器发送来的消息。也就是说，微信服务器的消息都是通过 POST 请求发给我的。

Token 可由开发者可以任意填写，用作生成签名（该 Token 会和接口 URL 中包含的 Token 进行比对，从而验证安全性）。

EncodingAESKey 由开发者手动填写或随机生成，将用作消息体加解密密钥。

同时，开发者可选择消息加解密方式：明文模式、兼容模式和安全模式。明文模式就是我们自己的服务器收到微信服务器发来的消息是明文字符串，直接就可以读取并且解析，安全模式则是我们收到微信服务器发来的消息是加密的消息，需要我们手动解析后才能使用。

## 4. 开发

公众号后台配置完成后，接下来我们就可以写代码了。

### 4.1 服务器有效性校验

我们首先来创建一个普通的 Spring Boot 项目，创建时引入 `spring-boot-starter-web` 依赖，项目创建成功后，我们创建一个 Controller ，添加如下接口：

```java
@GetMapping("/verify_wx_token")
public void login(HttpServletRequest request, HttpServletResponse response) throws UnsupportedEncodingException {
    request.setCharacterEncoding("UTF-8");
    String signature = request.getParameter("signature");
    String timestamp = request.getParameter("timestamp");
    String nonce = request.getParameter("nonce");
    String echostr = request.getParameter("echostr");
    PrintWriter out = null;
    try {
        out = response.getWriter();
        if (CheckUtil.checkSignature(signature, timestamp, nonce)) {
            out.write(echostr);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        out.close();
    }
}
```

关于这段代码，我做如下解释：

1. 首先通过 request.getParameter 方法获取到微信服务器发来的 signature、timestamp、nonce 以及 echostr 四个参数，这四个参数中：signature 表示微信加密签名，signature 结合了开发者填写的 token 参数和请求中的timestamp参数、nonce参数；timestamp 表示时间戳；nonce	表示随机数；echostr	则表示一个随机字符串。
2. 开发者通过检验 signature 对请求进行校验，如果确认此次 GET 请求来自微信服务器，则原样返回 echostr 参数内容，则接入生效，成为开发者成功，否则接入失败。
3. 具体的校验就是松哥这里的 CheckUtil.checkSignature 方法，在这个方法中，首先将token、timestamp、nonce 三个参数进行字典序排序，然后将三个参数字符串拼接成一个字符串进行 sha1 加密，最后开发者获得加密后的字符串可与 signature 对比，标识该请求来源于微信。

校验代码如下：

```java
public class CheckUtil {
    private static final String token = "123456";
    public static boolean checkSignature(String signature, String timestamp, String nonce) {
        String[] str = new String[]{token, timestamp, nonce};
        //排序
        Arrays.sort(str);
        //拼接字符串
        StringBuffer buffer = new StringBuffer();
        for (int i = 0; i < str.length; i++) {
            buffer.append(str[i]);
        }
        //进行sha1加密
        String temp = SHA1.encode(buffer.toString());
        //与微信提供的signature进行匹对
        return signature.equals(temp);
    }
}
public class SHA1 {
    private static final char[] HEX_DIGITS = {'0', '1', '2', '3', '4', '5',
            '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};
    private static String getFormattedText(byte[] bytes) {
        int len = bytes.length;
        StringBuilder buf = new StringBuilder(len * 2);
        for (int j = 0; j < len; j++) {
            buf.append(HEX_DIGITS[(bytes[j] >> 4) & 0x0f]);
            buf.append(HEX_DIGITS[bytes[j] & 0x0f]);
        }
        return buf.toString();
    }
    public static String encode(String str) {
        if (str == null) {
            return null;
        }
        try {
            MessageDigest messageDigest = MessageDigest.getInstance("SHA1");
            messageDigest.update(str.getBytes());
            return getFormattedText(messageDigest.digest());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

OK，完成之后，我们的校验接口就算是开发完成了。接下来就可以开发消息接收接口了。

### 4.2 消息接收接口

接下来我们来开发消息接收接口，消息接收接口和上面的服务器校验接口地址是一样的，都是我们一开始在公众号后台配置的地址。只不过消息接收接口是一个 POST 请求。

我在公众号后台配置的时候，消息加解密方式选择了明文模式，这样我在后台收到的消息直接就可以处理了。微信服务器给我发来的普通文本消息格式如下：

```xml
<xml>
  <ToUserName><![CDATA[toUser]]></ToUserName>
  <FromUserName><![CDATA[fromUser]]></FromUserName>
  <CreateTime>1348831860</CreateTime>
  <MsgType><![CDATA[text]]></MsgType>
  <Content><![CDATA[this is a test]]></Content>
  <MsgId>1234567890123456</MsgId>
</xml>
```

这些参数含义如下：

|参数|描述|
|:--|:--|
|ToUserName|开发者微信号|
|FromUserName|发送方帐号（一个OpenID）|
|CreateTime|消息创建时间 （整型）|
|MsgType|消息类型，文本为text|
|Content|文本消息内容|
|MsgId|消息id，64位整型|

看到这里，大家心里大概就有数了，当我们收到微信服务器发来的消息之后，我们就进行 XML 解析，提取出来我们需要的信息，去做相关的查询操作，再将查到的结果返回给微信服务器。

这里我们先来个简单的，我们将收到的消息解析并打印出来：

```java
@PostMapping("/verify_wx_token")
public void handler(HttpServletRequest request, HttpServletResponse response) throws Exception {
    request.setCharacterEncoding("UTF-8");
    response.setCharacterEncoding("UTF-8");
    PrintWriter out = response.getWriter();
    Map<String, String> parseXml = MessageUtil.parseXml(request);
    String msgType = parseXml.get("MsgType");
    String content = parseXml.get("Content");
    String fromusername = parseXml.get("FromUserName");
    String tousername = parseXml.get("ToUserName");
    System.out.println(msgType);
    System.out.println(content);
    System.out.println(fromusername);
    System.out.println(tousername);
}
public static Map<String, String> parseXml(HttpServletRequest request) throws Exception {
    Map<String, String> map = new HashMap<String, String>();
    InputStream inputStream = request.getInputStream();
    SAXReader reader = new SAXReader();
    Document document = reader.read(inputStream);
    Element root = document.getRootElement();
    List<Element> elementList = root.elements();
    for (Element e : elementList)
        map.put(e.getName(), e.getText());
    inputStream.close();
    inputStream = null;
    return map;
}
```

大家看到其实都是一些常规代码，没有什么难度。

做完这些之后，我们将项目打成 jar 包在服务器上部署启动。启动成功之后，确认微信的后台配置也没问题，我们就可以在公众号上发一条消息了，这样我们自己的服务端就会打印出来刚刚消息的信息。

好了，篇幅限制，今天就和大家先聊这么多，后面再聊不同消息类型的解析和消息的返回问题。

不知道小伙伴们看懂没？有问题欢迎留言讨论。

参考资料：微信开放文档