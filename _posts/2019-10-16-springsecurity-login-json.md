---
title: Spring Security 前后端分离登录，非法请求直接返回 JSON
date: 2019-10-16 21:40:59
tags: [Spring Boot]
categories: Spring Boot
abbrlink: springsecurity-login-json
sidebar:
  nav: docs-zh
---

hello 各位小伙伴，国庆节终于过完啦，松哥也回来啦，今天开始咱们继续发干货！

<!--more-->

关于 Spring Security，松哥之前发过多篇文章和大家聊聊这个安全框架的使用：

1. [手把手带你入门 Spring Security！](https://mp.weixin.qq.com/s/HKJOlatXDS8awBNyCe9JMg)
2. [Spring Security 登录添加验证码](https://mp.weixin.qq.com/s/oDow2miLIst-R4NNzc_i4g)
3. [SpringSecurity 登录使用 JSON 格式数据](https://mp.weixin.qq.com/s/X1t-VCxzxIcQKOAu-pJrdw)
4. [Spring Security 中的角色继承问题](https://mp.weixin.qq.com/s/7D0qJiEIzNuz8VAVvZsXCA)
5. [Spring Security 中使用 JWT!](https://mp.weixin.qq.com/s/riyFQSrkQBQBCyomE__fLA)
6. [Spring Security 结合 OAuth2](https://mp.weixin.qq.com/s/1rVPzJGCtDZKvMoA4BYzIA)

不过，今天要和小伙伴们聊一聊 Spring Security 中的另外一个问题，那就是在 Spring Security 中未获认证的请求默认会重定向到登录页，但是在前后端分离的登录中，这个默认行为则显得非常不合适，今天我们主要来看看如何实现未获认证的请求直接返回 JSON ，而不是重定向到登录页面。

## 前置知识

这里关于 Spring Security 的基本用法我就不再赘述了，如果小伙伴们不了解，可以参考上面的 6 篇文章。

大家知道，在自定义 Spring Security 配置的时候，有这样几个属性：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .formLogin()
            .loginProcessingUrl("/doLogin")
            .loginPage("/login")
            //其他配置
            .permitAll()
            .and()
            .csrf().disable();
}
```

这里有两个比较重要的属性：

- loginProcessingUrl：这个表示配置处理登录请求的接口地址，例如你是表单登录，那么 form 表单中 action 的值就是这里填的值。
- loginPage：这个表示登录页的地址，例如当你访问一个需要登录后才能访问的资源时，系统就会自动给你通过重定向跳转到这个页面上来。

这种配置在前后端不分的登录中是没有问题的，在前后端分离的登录中，这种配置就有问题了。我举个简单的例子，例如我想访问 `/hello` 接口，但是这个接口需要登录之后才能访问，我现在没有登录就直接去访问这个接口了，那么系统会给我返回 302，让我去登录页面，在前后端分离中，我的后端一般是没有登录页面的，就是一个提示 JSON，例如下面这样：

```java
@GetMapping("/login")
public RespBean login() {
    return RespBean.error("尚未登录，请登录!");
}
```

> 完整代码大家可以参考我的微人事项目。

也就是说，当我没有登录直接去访问 `/hello` 这个接口的时候，我会看到上面这段 JSON 字符串。在前后端分离开发中，这个看起来没问题（后端不再做页面跳转，无论发生什么都是返回 JSON）。但是问题就出在这里，系统默认的跳转是一个重定向，就是说当你访问 `/hello` 的时候，服务端会给浏览器返回 302，同时响应头中有一个 Location 字段，它的值为 `http://localhost:8081/login` ，也就是告诉浏览器你去访问 `http://localhost:8081/login` 地址吧。浏览器收到指令之后，就会直接去访问 `http://localhost:8081/login` 地址，如果此时是开发环境并且请求还是 Ajax 请求，就会发生跨域。因为前后端分离开发中，前端我们一般在 NodeJS 上启动，然后前端的所有请求通过 NodeJS 做请求转发，现在服务端直接把请求地址告诉浏览器了，浏览器就会直接去访问 `http://localhost:8081/login` 了，而不会做请求转发了，因此就发生了跨域问题。

## 解决方案

很明显，上面的问题我们不能用跨域的思路来解决，虽然这种方式看起来也能解决问题，但不是最佳方案。

如果我们的 Spring Security 在用户未获认证的时候去请求一个需要认证后才能请求的数据，此时不给用户重定向，而是直接就返回一个 JSON，告诉用户这个请求需要认证之后才能发起，就不会有上面的事情了。

这里就涉及到 Spring Security 中的一个接口 `AuthenticationEntryPoint` ，该接口有一个实现类：`LoginUrlAuthenticationEntryPoint` ，该类中有一个方法 `commence`，如下：

```java
/**
 * Performs the redirect (or forward) to the login form URL.
 */
public void commence(HttpServletRequest request, HttpServletResponse response,
		AuthenticationException authException) {
	String redirectUrl = null;
	if (useForward) {
		if (forceHttps && "http".equals(request.getScheme())) {
			redirectUrl = buildHttpsRedirectUrlForRequest(request);
		}
		if (redirectUrl == null) {
			String loginForm = determineUrlToUseForThisRequest(request, response,
					authException);
			if (logger.isDebugEnabled()) {
				logger.debug("Server side forward to: " + loginForm);
			}
			RequestDispatcher dispatcher = request.getRequestDispatcher(loginForm);
			dispatcher.forward(request, response);
			return;
		}
	}
	else {
		redirectUrl = buildRedirectUrlToLoginPage(request, response, authException);
	}
	redirectStrategy.sendRedirect(request, response, redirectUrl);
}
```

首先我们从这个方法的注释中就可以看出，这个方法是用来决定到底是要重定向还是要 forward，通过 Debug 追踪，我们发现默认情况下 useForward 的值为 false，所以请求走进了重定向。

那么我们解决问题的思路很简单，直接重写这个方法，在方法中返回 JSON 即可，不再做重定向操作，具体配置如下：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .formLogin()
            .loginProcessingUrl("/doLogin")
            .loginPage("/login")
            //其他配置
            .permitAll()
            .and()
            .csrf().disable().exceptionHandling()
                .authenticationEntryPoint(new AuthenticationEntryPoint() {
            @Override
            public void commence(HttpServletRequest req, HttpServletResponse resp, AuthenticationException authException) throws IOException, ServletException {
                resp.setContentType("application/json;charset=utf-8");
                PrintWriter out = resp.getWriter();
                RespBean respBean = RespBean.error("访问失败!");
                if (authException instanceof InsufficientAuthenticationException) {
                    respBean.setMsg("请求失败，请联系管理员!");
                }
                out.write(new ObjectMapper().writeValueAsString(respBean));
                out.flush();
                out.close();
            }
        });
}
```

在 Spring Security 的配置中加上自定义的 `AuthenticationEntryPoint` 处理方法，该方法中直接返回相应的 JSON 提示即可。这样，如果用户再去直接访问一个需要认证之后才可以访问的请求，就不会发生重定向操作了，服务端会直接给浏览器一个 JSON 提示，浏览器收到 JSON 之后，该干嘛干嘛。

## 结语

好了，一个小小的重定向问题和小伙伴们分享下，不知道大家有没有看懂呢？这也是我最近在重构微人事的时候遇到的问题。预计 11 月份，微人事的 Spring Boot 版本会升级到目前最新版，请小伙伴们留意哦。