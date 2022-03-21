# JavaWeb面试题

### Servlet的生命周期

```java
-1 web Client 向servlet服务器，发出Http请求
-2 servlet接受web Client的请求
-3 servlet容器创建一个HttpRequest 对象，将Web Client请求的信息封装到这个对象中
-4 servlet容器创建一个HttpResponset对象
-5 servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给HttpServlet对象
-6 HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息
-7 HttpServlet调用HttpResponst 对象的有关方法，生成响应数据
-8 Servlet容器把 HttpServlet的响应结果给Web Client
    
简单的说:init    service   destroy    
```

### jsp和Servlet的区别

```java
-1 jsp的本质是Servlet jvm只能识别java的类，不能识别jsp的代码
-2 jsp更擅长表现与页面显示，Servlet更擅长与逻辑控制
-3 servlet中没有内置对象，jsp中有九大内置对象    
```

### Jsp九大内置对象

| 变量名      | 真实类型           | 作用                                     |
| ----------- | ------------------ | ---------------------------------------- |
| pageContext | PageContext        | 当前页面共享数据,获取其他8个内置对象     |
| request     | HttpServletRequest | 一次请求访问的多个资源共享数据(请求转发) |
| reponse     | HttpServletReponse | 响应对象                                 |
| session     | HttpSession        | 一次会话的多个请求间共享数据             |
| application | ServletContext     | 所有用户间共享数据                       |
| page        | Object             | 当前Servlet对象（this）                  |
| out         | JspWriter          | 输出对象把数据输出到页面上               |
| config      | ServletConfig      | Servlet的配置对象                        |
| exception   | Throwable          | 异常对象                                 |

```java
重点掌握个对象
    request：  请求对象
    response： 响应对象
    out：字符输出流对象。可以将数据输出到页面上。和response.getWriter()类似

response.getWriter()和out.write()的区别：
    * 在tomcat服务器真正给客户端做出响应之前，会先找response缓冲区数据，再找out缓冲区数据。
    * response.getWriter()数据输出永远在out.write()之前
```

### 四大域对象

| 域对象名称     | 范围     | 级别                       | 备注                                     |
| -------------- | -------- | -------------------------- | ---------------------------------------- |
| PageContext    | 页面范围 | 最小，只能在当前页面用     | 因范围太小，开发中用的很少               |
| ServletRequest | 请求范围 | 一次请求或者当前请求转发用 | 请求转发之后，再次转发时请求域丢失       |
| HttpSession    | 会话范围 | 多次请求数据共享时使用     | 多次请求共享数据，但不同的客户端不能共享 |
| Servletcontext | 应用范围 | 最大，整个应用都可以使用   | 尽力少用，如果对数据有修改需要做同步处理 |

PageContext对象

- 是jsp独有的，Servlet没有
- 是四大域对象之一的页面域对象，可以操作其他三个域对象中的属性
- 可以获取其他八个隐式对象
- 生命周期随着jsp的创建而存在，随着jsp的结束而消失，每个jsp页面都有一个PageConte

### Cookie和Session的区别

**Cookie**（客户端，不是很安全）

- Cookie是由服务端创建，由若干个键值对组成的数据，并且随着响应以文件的形式将Cookie数据保存在客户端本地。当浏览器再次访问服务器时会携带Cookie数据，从而实现多次请求的数据共享。
- 作用：可以保存客户端访问网站的相关内容，从而保证每次访问时先从本地缓存中获取，以提高效率

- Cookie的使用
  - 数量限制，每个网站最多20个cookie，大小不超过4kb，所有网站的cookie不超过300个

**Session**（服务端，安全）

- 第一次请求的时候，服务器会创建一个带有id的Session的对象，然后服务器会把session的id以cookie的形式发送给客户端，第二次请求的时候根据id判断是否是同一个session对象，从而实现数据共享。

### 什么是Token

```java
Token的引入：Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生。

Token的定义：Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。

使用Token的目的：Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮。
```

### session与token区别

- session机制存在服务器压力增大，CSRF跨站伪造请求攻击，扩展性不强等问题；
- session存储在服务器端，token存储在客户端
- token提供认证和授权功能，作为身份认证，token安全性比session好；
- session这种会话存储方式方式只适用于客户端代码和服务端代码运行在同一台服务器上，token适用于项目级的前后端分离（前后端代码运行在不同的服务器下）

### get和post的区别

|                  | get  | post                         |
| ---------------- | ---- | ---------------------------- |
| 后退，刷新       |      | 数据重新提交                 |
| 书签             |      | 不可以收藏书签               |
| 历史             |      | 参数不可以保存在浏览器历史中 |
| 对数据长度的限制 |      | 没有限制                     |
| 安全性           |      | 安全                         |
| 可见性           |      | 数据不会限制在URL中          |

### 转发和重定向

```java
转发：服务端行为，地址不变，数据共享，效率高
    public void forward(HttpServletRequest req, HttpServletResponse resp)
重定向：客户端行为，地址发送改变，数据不共享，效率低   
    resp.sendRedirect(req.getContextPath()+"/servletDemo07");
```

### Servlet是线程安全的吗

```java
Servlet不是线程安全的，多线程并发的读写会导致数据不同步的问题。 

解决的办法是尽量不要在实现servlet接口的类中定义实例变量，而是要把变量分别定义在doGet()和doPost()方法内。虽然使用synchronized(name){}语句块可以解决问题，但是会造成线程的等待，不是很科学的办法。

注意：多线程的并发的读写Servlet类属性会导致数据不同步。但是如果只是并发地读取属性而不写入，则不存在数据不同步的问题。因此Servlet里的只读属性最好定义为final类型的。
```

### Servlet执行流程

```java
web客户向Servlet容器发出HTTP请求;

Servlet容器解析web的HTTP请求.

Servlet容器创建一个HttpRequest对象，在这个对象中封装了http请求信息;

Servlet容器创建一个HttpResponse对象;

Servlet容器（如果访问的该servlet不是在服务器启动时创建的，则先创建servlet实例并调用init()方法初始化对象）调用HttpServlet的service()方法，把HttpRequest和HttpResponse对象为service方法的参数传给HttpServlet对象;

HttpServlet调用HttpRequest的有关方法，获取HTTP请求信息;

HttpServlet调用HttpResponse的有关方法，生成响应数据;

Servlet容器把HttpServlet的响应结果传给web客户.
```

### 三次握手和四次挥手

```java
三次握手:
	(1)客户端向服务器发出连接请求等待服务器确认
	(2)服务器向客户端返回一个响应告诉客户端收到了请求
	(3)客户端向服务器再次发出确认信息,此时连接建立
四次挥手:
	(1)客户端向服务器发出取消连接请求
	(2)服务器向客户端返回一个响应,表示收到客户端取消请求
	(3)服务器向客户端发出确认取消信息(向客户端表明可以取消连接了)
	(4)客户端再次发送确认消息,此时连接取消
```

### TCP和UDP的区别

```java
1、TCP ：面向连接，UDP ：面向无连接
2、TCP ：传输效率低，UDP ：传输效率高(有大小限制，一次限定在64kb之内)
3、TCP：可靠，UDP ：不可靠
```

### 如何解决跨域问题？

```java
跨域指的是浏览器不能执行其它网站的脚本，它是由浏览器的同源策略造成的，是浏览器对JavaScript 施加的安全限制。
    
所谓同源指的是：协议、域名、端口号都相同，只要有一个不相同，那么都是非同源。
    
解决方案：
    1：使用ajax的jsonp
    2：nginx 转发：利用nginx反向代理，将请求分发到部署相应项目的tomcat服务器，当然也不存在跨域问题。
    3：使用cors：写一个配置类实现WebMvcConfigurer接口或者配置FilterRegistrationBean
```

### 什么是 CSRF 攻击？如何防御CSRF 攻击

```java
CSRF（Cross-site request forgery） 跨站请求伪造。CSRF 攻击是在受害者毫不知情的情况下，以受害者名义伪造请求发送给受攻击站点，从而在受害者并未授权的情况下执行受害者权限下的各种操作。
    
CSRF 攻击专门针对状态改变请求，而不是数据窃取，因为攻击者无法查看对伪造请求的响应。
    
目前防御 CSRF 攻击主要有三种策略：
    验证 HTTP Referer 字段
    在请求地址中添加 token 并验证
    在 HTTP 头中自定义属性并验证
```

### http和https的基本概念

```java
-HTTP:
	是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于计算机之间传输文字，图片，音频，视频等超文本数据的协议，它可以使浏览器更加高效，使网络传输减少
-HTTPS:
	是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层，HTTPS的安全基础是SSL，HTTPS就是从HTTP加上加密处理（一般是SSL安全通信线路）+认证+完整性保护

-HTTPS协议的主要作用：
    建立一个信息安全通道，来保证数据传输的安全
    确认网站的真实性    
```

### http和https的区别？

| 区别     | **HTTP**                                                     | **HTTPS**                                                    |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 协议     | 运行在 TCP 之上，明文传输，**客户端与服务器端都无法验证对方的身份** | 身披 SSL( Secure Socket Layer )外壳的 HTTP，运行于 SSL 上，SSL 运行于 TCP 之上， **是添加了加密和认证机制的 HTTP**。 |
| 端口     | 80                                                           | 443                                                          |
| 资源消耗 | 较少                                                         | 由于加解密处理，会消耗更多的 CPU 和内存资源                  |
| 证书     | 无需证书                                                     | 需要证书，而证书一般需要向认证机构购买                       |
| 加密机制 | 无                                                           | 共享密钥加密和公开密钥加密并用的混合加密机制安全性弱由于加密机制，安全性强 |
| 安全性   | 弱                                                           | 由于加密机制，安全性强                                       |

### 一次完整的HTTP请求所经历几个步骤?

```java
根据域名和 DNS 解析到服务器的IP地址 (DNS + CDN)

通过ARP协议获得IP地址对应的物理机器的MAC地址

浏览器对服务器发起 TCP 3 次握手

建立 TCP 连接后发起 HTTP 请求报文

服务器响应 HTTP 请求，将响应报文返回给浏览器

短连接情况下，请求结束则通过 TCP 四次挥手关闭连接，长连接在没有访问服务器的若干时间后，进行连接的关闭

浏览器得到响应信息中的 HTML 代码， 并请求 HTML 代码中的资源（如js、css、图片等）

浏览器对页面进行渲染并呈现给用户
```

### 常用HTTP状态码是怎么分类的?

| 状态码 | **类别**         | **描述**                                       |
| ------ | ---------------- | ---------------------------------------------- |
| 1xx    | 信息状态码       | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2xx    | 成功状态码       | 成功，操作被成功接收并处理                     |
| 3xx    | 重定向状态码     | 重定向，需要进一步的操作以完成请求             |
| 4xx    | 客户端错误状态码 | 客户端错误，请求包含语法错误或无法完成请求     |
| 5xx    | 服务器错误状态码 | 服务器错误，服务器在处理请求的过程中发生了错误 |

### HTTP1.0和HTTP1.1的区别

```java
缓存处理，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。

带宽优化及网络连接的使用，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

错误通知的管理，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

Host头处理，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

长连接，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。
```

