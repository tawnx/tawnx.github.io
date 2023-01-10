---
title: CSRF（跨站请求伪造）
date: 2022-8-6 15:50:38
tags: 
- Web
- Security
- CSRF
categories: 
- web安全基础漏洞
---

# CSRF（跨站请求伪造）

## 简介

CSRF 的全名是 Cross Site Request Forgery，翻译成中文就是跨站点请求伪造。

它是一种常见的 Web 攻击，但很多开发者对它很陌生。CSRF 也是 Web 安全中最容易被忽 略的一种攻击方式，甚至很多安全工程师都不太理解它的利用条件与危害，因此不予重视。但 CSRF 在某些时候却能够产生强大的破坏性。

## 原理

还记得在“跨站脚本攻击”一章中，介绍 XSS Payload 时的那个“删除搜狐博客”的例子 吗?登录 Sohu 博客后，只需要请求这个 URL，就能够把编号为“156713012”的博客文章删除。 

```
http://blog.sohu.com/manage/entry.do?m=delete&id=156713012
```

这个 URL 同时还存在 CSRF 漏洞。我们将尝试利用 CSRF 漏洞，删除编号为“156714243” 的博客文章。这篇文章的标题是“test1”。

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-06-下午2.22.46.png)

攻击者首先在自己的域构造一个页面:

```
http://www.a.com/csrf.html
```

其内容为:

<img src="http://blog.sohu.com/manage/entry.do?m=delete&id=156714243" />

使用了一个<img>标签，其地址指向了删除博客文章的链接。 攻击者诱使目标用户，也就是博客主“test1test”访问这个页面:

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-06-下午2.35.27.png)

该用户看到了一张无法显示的图片，再回过头看看搜狐博客:

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-06-下午2.35.58.png)

文章被删除 发现原来存在的标题为“test1”的博客文章，已经被删除了!

原来刚才访问 http://www.a.com/csrf.html 时，图片标签向搜狐的服务器发送了一次 GET 请求:

![img]( https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-08/截屏2022-08-06-下午2.36.18.png)

而这次请求，导致了搜狐博客上的一篇文章被删除。

回顾整个攻击过程，攻击者仅仅诱使用户访问了一个页面，就以该用户身份在第三方站点 里执行了一次操作。试想:如果这张图片是展示在某个论坛、某个博客，甚至搜狐的一些用户 空间中，会产生什么效果呢?只需要经过精心的设计，就能够起到更大的破坏作用。

这个删除博客文章的请求，是攻击者所伪造的，所以这种攻击就叫做“跨站点请求伪造”。

## 利用方式

除了利用上述img标签的自动发送get请求之外，还可以利用其他标签：

```
#配合XSS攻击
<script src="http://127.0.0.1/vulnerabilities/csrf/?password_new=111&password_conf=111&Change=Change#"></script>

#iframe标签，添加样式 style="display:none;"
<iframe src="http://127.0.0.1/vulnerabilities/csrf/?password_new=111&password_conf=111&Change=Change#" style="display:none;"></iframe>

#img标签同理，添加样式 border="0" style="display:none;"
<img src="http://127.0.0.1/vulnerabilities/csrf/?password_new=111&password_conf=111&Change=Change#" border="0" style="display:none;">
```

也可以构建一个自动提交的表单，发送post请求

```
<form action="http://127.0.0.1/vulenrabilities/csrf/" id="csrf" method="get">
        <input type="hidden" name="password_new" value="111">
        <input type="hidden" name="password_conf" value="111">
        <input type="hidden" name="Change" value="Change">
</form>
```

## 常见的防御和绕过方式

### 验证码

验证码被认为是对抗 CSRF 攻击最简洁而有效的防御方法。验证码强制用户必须与应用进行交互，才能完成最终请求。因此在通常情况下，验证码能够很好地遏制 CSRF 攻击。 但是验证码会影响用户的体验。

### Referer Check

常见的互联网应用，页面与页面之间都具有一定的逻辑关系，这就使得每个正常请求的 Referer 具有一定的规律。

比如一个“论坛发帖”的操作，在正常情况下需要先登录到用户后台，或者访问有发帖功 能的页面。在提交“发帖”的表单时，Referer 的值必然是发帖表单所在的页面。如果 Referer 的值不是这个页面，甚至不是发帖网站的域，则极有可能是 CSRF 攻击。

即使我们能够通过检查 Referer 是否合法来判断用户是否被 CSRF 攻击，也仅仅是满足了 防御的充分条件。**Referer Check** 的缺陷在于，服务器并非什么时候都能取到 **Referer**。很多用 户出于隐私保护的考虑，限制了 Referer 的发送。在某些情况下，浏览器也不会发送 Referer， 比如从 HTTPS 跳转到 HTTP，出于安全的考虑，浏览器也不会发送 Referer。

### 绕过

我们能控制的就是自己站点的url了，可以在把代码放在以 `blog.sohu.com` 为名称的目录下

```
http://www.a.com/blog.sohu.com/csrf.html
```

或者修改文件名

```
http://www.a.com/blog.sohu.com.html
```

也可以在之后添加参数

```
http://www.a.com/csrf.html?blog.sohu.com
```

### SameSite

Cookie 的`SameSite`属性用来限制第三方 Cookie，从而减少安全风险。

它可以设置三个值：`Strict`, `Lax`, `None`

`Strict` 最为严格，完全禁止第三方 Cookie，跨站点时，任何情况下都不会发送 Cookie。

`Lax` 规则稍稍放宽，大多数情况也是不发送第三方 Cookie，但是导航到目标网址的 Get 请求除外。

导航到目标网址的 GET 请求，只包括三种情况：链接，预加载请求，GET 表单。详见下表。

| 请求类型  | 示例                                 | 正常情况    | Lax         |
| --------- | ------------------------------------ | ----------- | ----------- |
| 链接      | `<a href="..."></a>`                 | 发送 Cookie | 发送 Cookie |
| 预加载    | `<link rel="prerender" href="..."/>` | 发送 Cookie | 发送 Cookie |
| GET 表单  | `<form method="GET" action="...">`   | 发送 Cookie | 发送 Cookie |
| POST 表单 | `<form method="POST" action="...">`  | 发送 Cookie | 不发送      |
| iframe    | `<iframe src="..."></iframe>`        | 发送 Cookie | 不发送      |
| AJAX      | `$.get("...")`                       | 发送 Cookie | 不发送      |
| Image     | `<img src="...">`                    | 发送 Cookie | 不发送      |

设置了`Strict`或`Lax`以后，基本就杜绝了 CSRF 攻击。当然，前提是用户浏览器支持 SameSite 属性。

 `None` 无论是否跨站都会发送 Cookie

Chrome 计划将`Lax`变为默认设置。这时，网站可以选择显式关闭`SameSite`属性，将其设为`None`。不过，前提是必须同时设置`Secure`属性（Cookie 只能通过 HTTPS 协议发送），否则无效。

下面的设置无效。

```
Set-Cookie: widget_session=abc123; SameSite=None
```

下面的设置有效。

```
Set-Cookie: widget_session=abc123; SameSite=None; Secure
```