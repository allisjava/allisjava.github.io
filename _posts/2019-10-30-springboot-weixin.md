---
title: Spring Boot 开发微信公众号后台(二)
date: 2019-10-31 10:32:24
tags: [weixin,Spring Boot]
categories: Spring Boot
abbrlink: springboot-weixin
sidebar:
  nav: docs-zh
---

hello 各位小伙伴，今天我们来继续学习如何通过 Spring Boot 开发微信公众号。还没阅读过上篇文章的小伙伴建议先看看上文，有助于理解本文：

<!--more-->

- [Spring Boot 开发微信公众号后台](https://mp.weixin.qq.com/s/f3QexxLp9vT6aE1Pl3jHGw)

上篇文章中我们将微信服务器和我们自己的服务器对接起来了，并且在自己的服务器上也能收到微信服务器发来的消息，本文我们要看的就是如何给微信服务器回复消息。

## 消息分类

在讨论如何给微信服务器回复消息之前，我们需要先来了解下微信服务器发来的消息主要有哪些类型以及我们回复给微信的消息都有哪些类型。

在上文中大家了解到，微信发送来的 xml 消息中有一个 MsgType 字段，这个字段就是用来标记消息的类型。这个类型可以标记出这条消息是普通消息还是事件消息还是图文消息等。

普通消息主要是指：

- 文本消息
- 图片消息
- 语音消息
- 视频消息
- 小视频消息
- 地址位置消息
- 链接消息

不同的消息类型，对应不同的 MsgType，这里我还是以普通消息为例，如下：

|消息类型|MsgType|
|:--|:--|
|文本消息|text|
|图片消息|image|
|语音消息|voice|
|视频消息|video|
|小视频消息|shortvideo|
|地址位置消息|location|
|链接消息|link|

大家千万不要以为不同类型消息的格式是一样的，其实是不一样的，也就是说，MsgType 为 text 的消息和 MsgType 为 image 的消息，微信服务器发给我们的消息内容是不一样的，这样带来一个问题就是我无法使用一个 Bean 去接收不同类型的数据，因此这里我们一般使用 Map 接收即可。

这是消息的接收，除了消息的接收之外，还有一个消息的回复，我们回复的消息也有很多类型，可以回复普通消息，也可以回复图片消息，回复语音消息等，不同的回复消息我们可以进行相应的封装。因为不同的返回消息实例也是有一些共同的属性的，例如消息是谁发来的，发给谁，消息类型，消息 id 等，所以我们可以将这些共同的属性定义成一个父类，然后不同的消息再去继承这个父类。

## 返回消息类型定义

首先我们来定义一个公共的消息类型：

```java
public class BaseMessage {
    private String ToUserName;
    private String FromUserName;
    private long CreateTime;
    private String MsgType;
    private long MsgId;
    //省略 getter/setter
}
```

在这里：

- ToUserName 表示开发者的微信号
- FromUserName 表示发送方账号（用户的 OpenID）
- CreateTime 消息的创建时间
- MsgType 表示消息的类型
- MsgId 表示消息 id

这是我们的基本消息类型，就是说，我们返回给用户的消息，无论是什么类型的消息，都有这几个基本属性。然后在此基础上，我们再去扩展出文本消息、图片消息 等。

我们来看下文本消息的定义：

```java
public class TextMessage extends BaseMessage {
    private String Content;
    //省略 getter/setter
}
```

文本消息在前面消息的基础上多了一个 Content 属性，因此文本消息继承自 BaseMessage ，再额外添加一个 Content 属性即可。

其他的消息类型也是类似的定义，我就不一一列举了，至于其他消息的格式，大家可以参考微信开放文档（http://1t.click/aPXK）。

## 返回消息生成

消息类型的 Bean 定义完成之后，接下来就是将实体类生成 XML。

首先我们定义一个消息工具类，将常见的消息类型枚举出来：

```java
/**
 * 返回消息类型：文本
 */
public static final String RESP_MESSAGE_TYPE_TEXT = "text";
/**
 * 返回消息类型：音乐
 */
public static final String RESP_MESSAGE_TYPE_MUSIC = "music";
/**
 * 返回消息类型：图文
 */
public static final String RESP_MESSAGE_TYPE_NEWS = "news";
/**
 * 返回消息类型：图片
 */
public static final String RESP_MESSAGE_TYPE_Image = "image";
/**
 * 返回消息类型：语音
 */
public static final String RESP_MESSAGE_TYPE_Voice = "voice";
/**
 * 返回消息类型：视频
 */
public static final String RESP_MESSAGE_TYPE_Video = "video";
/**
 * 请求消息类型：文本
 */
public static final String REQ_MESSAGE_TYPE_TEXT = "text";
/**
 * 请求消息类型：图片
 */
public static final String REQ_MESSAGE_TYPE_IMAGE = "image";
/**
 * 请求消息类型：链接
 */
public static final String REQ_MESSAGE_TYPE_LINK = "link";
/**
 * 请求消息类型：地理位置
 */
public static final String REQ_MESSAGE_TYPE_LOCATION = "location";
/**
 * 请求消息类型：音频
 */
public static final String REQ_MESSAGE_TYPE_VOICE = "voice";
/**
 * 请求消息类型：视频
 */
public static final String REQ_MESSAGE_TYPE_VIDEO = "video";
/**
 * 请求消息类型：推送
 */
public static final String REQ_MESSAGE_TYPE_EVENT = "event";
/**
 * 事件类型：subscribe(订阅)
 */
public static final String EVENT_TYPE_SUBSCRIBE = "subscribe";
/**
 * 事件类型：unsubscribe(取消订阅)
 */
public static final String EVENT_TYPE_UNSUBSCRIBE = "unsubscribe";
/**
 * 事件类型：CLICK(自定义菜单点击事件)
 */
public static final String EVENT_TYPE_CLICK = "CLICK";
/**
 * 事件类型：VIEW(自定义菜单 URl 视图)
 */
public static final String EVENT_TYPE_VIEW = "VIEW";
/**
 * 事件类型：LOCATION(上报地理位置事件)
 */
public static final String EVENT_TYPE_LOCATION = "LOCATION";
/**
 * 事件类型：LOCATION(上报地理位置事件)
 */
public static final String EVENT_TYPE_SCAN = "SCAN";
```

大家注意这里消息类型的定义，以 RESP 开头的表示返回的消息类型，以 REQ 表示微信服务器发来的消息类型。然后在这个工具类中再定义两个方法，用来将返回的对象转换成 XML：

```java
public static String textMessageToXml(TextMessage textMessage) {
    xstream.alias("xml", textMessage.getClass());
    return xstream.toXML(textMessage);
}
private static XStream xstream = new XStream(new XppDriver() {
    public HierarchicalStreamWriter createWriter(Writer out) {
        return new PrettyPrintWriter(out) {
            boolean cdata = true;
            @SuppressWarnings("rawtypes")
            public void startNode(String name, Class clazz) {
                super.startNode(name, clazz);
            }
            protected void writeText(QuickWriter writer, String text) {
                if (cdata) {
                    writer.write("<![CDATA[");
                    writer.write(text);
                    writer.write("]]>");
                } else {
                    writer.write(text);
                }
            }
        };
    }
});
```

textMessageToXML 方法用来将 TextMessage 对象转成 XML 返回给微信服务器，类似的方法我们还需要定义 imageMessageToXml、voiceMessageToXml 等，不过定义的方式都基本类似，我就不一一列出来了。

## 返回消息分发

由于用户发来的消息可能存在多种情况，我们需要分类进行处理，这个就涉及到返回消息的分发问题。因此我在这里再定义一个返回消息分发的工具类，如下：

```java
public class MessageDispatcher {
    public static String processMessage(Map<String, String> map) {
        String openid = map.get("FromUserName"); //用户 openid
        String mpid = map.get("ToUserName");   //公众号原始 ID
        if (map.get("MsgType").equals(MessageUtil.REQ_MESSAGE_TYPE_TEXT)) { 
            //普通文本消息
            TextMessage txtmsg = new TextMessage();
            txtmsg.setToUserName(openid);
            txtmsg.setFromUserName(mpid);
            txtmsg.setCreateTime(new Date().getTime());
            txtmsg.setMsgType(MessageUtil.RESP_MESSAGE_TYPE_TEXT);
            txtmsg.setContent("这是返回消息");
            return MessageUtil.textMessageToXml(txtmsg);
        }
        return null;
    }
    public String processEvent(Map<String, String> map) {
        //在这里处理事件
    }
}
```

这里我们还可以多加几个 elseif 去判断不同的消息类型，我这里因为只有普通文本消息，所以一个 if 就够用了。

在这里返回值我写死了，实际上这里需要根据微信服务端传来的 Content 去数据中查询，将查询结果返回，数据库查询这一套相信大家都能搞定，我这里就不重复介绍了。

最后在消息接收 Controller 中调用该方法，如下：

```java
@PostMapping(value = "/verify_wx_token",produces = "application/xml;charset=utf-8")
public String handler(HttpServletRequest request, HttpServletResponse response) throws Exception {
    request.setCharacterEncoding("UTF-8");
    Map<String, String> map = MessageUtil.parseXml(request);
    String msgType = map.get("MsgType");
    if (MessageUtil.REQ_MESSAGE_TYPE_EVENT.equals(msgType)) {
        return messageDispatcher.processEvent(map);
    }else{
        return messageDispatcher.processMessage(map);
    }
}
```

在 Controller 中，我们首先判断消息是否是事件，如果是事件，进入到事件处理通道，如果不是事件，则进入到消息处理通道。

**注意，这里需要配置一下返回消息的编码，否则可能会出现中文乱码。**

如此之后，我们的服务器就可以给公众号返回消息了。

上篇文章发出后，有小伙伴问松哥这个会不会开源，我可以负责任的告诉大家，肯定会开源，这个系列截稿后，我把代码处理下就上传到 GitHub。

好了，本文我们就先说到这里。