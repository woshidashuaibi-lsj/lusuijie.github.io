---
title: 由前端网络安全到http，https和浏览器的存储（个人笔记）
date: 2020-06-17
tags: ['笔记']
series: []
featured: false
reading_time: 11
---


<!--more-->

常见的web攻击方式

 -XSS
 - CSRF
 - 点击劫持
 - SQL注入
 - OS注入
 - 请求劫持
 - DDOS
XSS
参考：https://juejin.im/post/6844903685122703367#heading-18

 Cross-Site scripting 跨站脚本攻击，因为和css同名所以改成xss。

就比如一个掘金写文章的页面，假设掘金没有做xss攻击的脚本，我在我的文章里面写了<script>alert("攻击")<script>，那么当用户点开我的文章的时候就会有个alert提醒。过分一点，就用一个while循环无线重复alert。也有可能写上一些js代码获取用户的cookie,来帮助黑客恶意操作网站。

这是存储型的xss攻击。主要运用再存储到数据库这种的网站进行攻击。

还有一个反射型的xss攻击，

<input type="text" value="<%= getParameter("keyword") %>">
<button>搜索</button>
<div>
  您搜索的关键词是：<%= getParameter("keyword") %>
</div作者：美团技术团队链接：https://juejin.im/post/6844903685122703367来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
比如我们在一个input框输入一段恶意代码<script>alert("攻击")<script>，会形成如下的代码。

<input type="text" value=""><script>alert('XSS');</script>">
<button>搜索</button>
<div>
  您搜索的关键词是："><script>alert('XSS');</script>
</div>作者：美团技术团队链接：https://juejin.im/post/6844903685122703367来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
当浏览器请求 http://xxx/search?keyword="><script>alert('XSS');</script> 时是无法分辨的。

反射性的xss攻击是在url上操作的，如网站搜索、跳转等

DOM 型 XSS 的攻击步骤：

攻击者构造出特殊的 URL，其中包含恶意代码。
用户打开带有恶意代码的 URL。
用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行。
恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。
DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

XSS 的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

课外知识热点：

曾经在猫扑大杂烩中存在这样一个XSS漏洞，在用户发表回复的时候，程序对用户发表的内容做了严格的过滤，但是我不知道为什么，当用户编辑回复内容再次发表的时候，他却采用了另外一种不同的过滤方式，而这种过滤方式显然是不严密的，因此导致了XSS漏洞的出现。试想一下，像猫扑这样的大型社区，如果在一篇热帖中，利用XSS漏洞来使所有的浏览这篇帖子的用户都在不知不觉之中访问到了另外一个站点，如果这个站点同样是大型站点还好，但如果是中小型站点那就悲剧了，这将会引来多大的流量啊！更可怕的是，这些流量全部都是真实有效的！

解决方案：
对一些特殊的符号进行转义什么的，比如< > 什么的，

最好的办法就是配置白名单和黑名单。

CSRF
参考：https://www.bilibili.com/video/BV1iW411171s?from=search&seid=6829106872112135941

(Cross Site Request Forgery) 跨站请求伪造。
攻击者盗用你的身份，以你的名义去发送一些恶意请求。



比如你登陆进一个页面，然后进入转账页面



此时页面时保留了你的登录信息的，你没有关闭此页面，而是进入到一个黑客网站。如下网站的源代码，他的图片的路劲就是你转账的路劲。所以此时你的账户会被转账。



解决办法将get请求改成post请求这样就不能看到你的路径了

但是黑客同样有办法：通过表单from来提交接口

1. 解决方法加入一个验证码，因为他只是构造了一个请求，不能操作我们的也页面。

2. 验证Refer 即同源策略，不是同域名同端口同协议的拒绝请求。

3. 在form表单或头信息中传递token,token存储在服务端，服务端通过拦截器验证有效性，检验失败的拒绝请求。



点击劫持
点击劫持是一种视觉欺骗的攻击手段,攻击者将需要攻击的网站通过iframe嵌套的方式嵌入自己的网页， 应将iframe设置为透明，在页面中透出一个按钮诱导用户点击。就比如电影天堂的网站，当我随便点击一个电影。


结果就进入了一个其他页面（还挺想玩一玩的TvT）。



这就是典型的点击劫持。页面使用 iframe 元素，将 iframe 透明化。就是吧这个跳转透明化。



解决办法：

 1. X-FRAME-OPTIONS 是一个HTTP响应头，就是为了防御iframe嵌套的点击劫持攻击
   DENY 页面不允许通过iframe方式展示
   SAPMEORIGIN 页面可以相同域名下通过iframe方式展示
   ALLOW-FROM 页面可以在指定来源的iframe中展示
 SQL注入
sql注入是黑客通过当你输入密码的时候判断你的密码是否和我数据课里面的密码是否相同，黑客通过1'or'1'='1你前面的密码可能不是相同的但是后面的1=1返回的时true,所以系统会判断你的密码时正确的。

解决方法:

1. 所有的查询语句建议使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入的变量嵌套到SQL语句中

2. 对进入数据库的特殊字符(',",\,<,>,&)等进行转义
OS注入
和sql注入差不多的原理，不过它是针对操作系统的，通过shell偷偷开启你的终端，操作你的电脑。

解决方法：同sql解决方法

请求劫持
  DNS劫持：DNS服务器（DNS解析各个步骤）被纂改，修改域名解析的结果，使得访问到的不是预期的ip。

 http劫持: 运营商劫持。升级为https

DDOS
distributed denial of service（分布式拒绝服务）
DDOS不是一个攻击的名称而是指一群攻击的合称。举个例子一个餐厅一次只能容纳300人一起吃饭，但是黑客一次叫3000个人去餐厅吃饭，餐厅根本容不下这么多人，于是就瘫痪了。DDOS就是这样的原理操作了，我同时叫100个ip地址同时发送请求，你的网站承受不了如此大的点击量。

解决方案

1.备用一个网站（全静态的网站）。如果瘫痪了先暂时上这个网站。

2。http请求拦截，如果同一个ip一直恶意请求，就直接把这个ip给封了。

3.有也那种IP你根本找不到的，那就可以带宽扩容，简单点就是你的饭店原来只能300人，现在我装修3000人也能坐的下，但是成本大，就给服务器怎加容量，这种看不到ip的攻击，黑客的成本也大。



以上就是前端常见的面试常问的网络安全放面的考点。上面也简单的说到过get 和post,http和https。下面我就顺带着一起讲一下，因为他们也是常考点。

get和post的区别
先从原理上来了解get和post,先将两个知识点，幂等和副作用。

副作用: 服务器上的资源是否改变。比如搜索服务器上的资源资源没有改动，是无副作用的。注册账号，资源改动是，副作用的。

幂等：指的是发送M 和 N 次请求(M N 不相等且都大于1)，服务器上的资源状态是否改变。比如对文章修改10次和11次是幂等的。而注册10次和注册11次是一样的，是不幂等的。

get一般用于无副作用幂等的。即资源和它的状态都不改动。

post一般用于副作用不幂等的。即资源和它的状态都改动了。

先前上面也提到过，get请求的参数会被携带在url里面，很容易被人利用，而post不会，所以post比get更加安全一点。
post支持更多的编码类型，且不被数据类型所限制。
get请求能缓存，post不能
http和https
http 是基于TCP/IP来传送超文本数据的。

https是基于http的基础上加了一个TLS协议。

TLS是使用了两种加密方式：

对称加密和非对称加密

对称加密：是指双方都有相同的密钥，双方用同一套加密规则加密，然后解密。

非对称加密：是我向大众宣布我的数据加密方法，然后解密方法只在我这里。

一般TLS加密是两种方法合并使用，我先用非对称加密，向大众发布我的加密方式，然后别人用这种加密方式，加密它的密钥给我，然后我就有了它的密钥，然后我们有了相同的密钥，这时候就用对称加密。

-------------------------------------------------------------------------------------------

更新：看了一些大佬的文章，感觉自己http和https还是没有悟到，在整理整理这里。

TLS是传输层加密协议，前身是SSL协议，由网景公司1995年发布，有时候两者不区分

http协议以明文的方式发送传输数据，不提供任何加密方式，所以就很有可能被人窃取数据，并且读懂数据内容，所以http不能用来传输一些很重要的信息，比如密码什么的。

https协议则是以http为基础来传输数据，不过在这上面加了一层SSL协议，用来加密传输的数据。

https协议使用的是443端口，http是80端口。

https工作原理：

https协议是需要申请证书的，这个证书就是一对私钥和密钥。客户端会检查对方的证书是否安全，不安全就会弹出证书安全的警告，如果安全就会用它的公钥对你私钥也就是一段随机值进行加密，发送给服务端，服务端和客户端就可以进行上面提到的对称加密了。

-------------------------------------------------------------------------------------

HTTP1.0，HTTP1.1，HTTP2.0
http1.0
特性：无状态，无连接的

无状态是指每次都无法保留用户的登陆记录和状态等等，只能通过cookie session来保存（服务器不跟踪不记录请求过的状态）

无连接是指每次发送请求都需要tcp三次握手，效率很低，并且在前一个请求响应到达之前才能发送下一个请求，如果前面的阻塞的话后面的都会被阻塞。

http1.1
为了解决http1.0的问题，http1.1新增了一些新特性：长连接，管道化，缓存处理，断点传输。

长连接：数据传输完毕不关闭tcp连接，继续用这个通道传输数据。

管道化：没有管道化和长连接时，我们时 请求一=》响应一  请求二 =》响应二 请求三 =》 响应三。有管道化和长连接以后，我们 请求一 =》请求二=》请求三 =》响应一=》 响应二=》 响应三  管道化就不需要等到上一个请求响应之后才发第二个请求，但是即使你的请求二比请求一先完成但是响应依旧要按照顺序响应。

缓存处理：第一次请求到的一些数据放到缓存中，这样下次请求的时候就可以从本地缓存中拿。

http2.0
二进制分帧
多路复用： 在共享TCP链接的基础上同时发送请求和响应
头部压缩
服务器推送：服务器可以额外的向客户端推送资源，而无需客户端明确的请求
二进制分帧:

将所有传输的信息分割为更小的消息和帧,并对它们采用二进制格式的编码

多路复用:

基于二进制分帧，在同一域名下所有访问都是从同一个tcp连接中走，http消息被分解为独立的帧，乱序发送，服务端根据标识符和首部将消息重新组装起来

区别：

http1.0 到http1.1的主要区别，就是从无连接到长连接
http2.0对比1.X版本主要区别就是多路复用
最后：
有时间再把cookie,token哪几种存储方式一起加到这里吧。

