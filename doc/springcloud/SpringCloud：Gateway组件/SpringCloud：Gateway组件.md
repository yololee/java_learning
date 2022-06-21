# SpringCloud：Gateway组件

## 一、简介

### 1.网关定义

网关的角色是作为一个 API 架构，用来保护、增强和控制对于 API 服务的访问。

API 网关是一个处于应用程序或服务（提供 REST API 接口服务）之前的系统，用来管理授权、访问控制和流量限制等，这样 REST API 接口服务就被 API 网关保护起来，对所有的调用者透明。因此，隐藏在 API 网关后面的业务系统就可以专注于创建和管理服务，而不用去处理这些策略性的基础设施

### 2.Gateway是什么

Spring Cloud Gateway是Spring官方基于Spring 5.0，Spring Boot 2.0和Project Reactor等技术开发的网关，Spring Cloud Gateway旨在为微服务架构提供一种简单而有效的统一的API路由管理方式。Spring Cloud Gateway作为Spring Cloud生态系中的网关，目标是替代ZUUL，其不仅提供统一的路由方式，并且基于Filter链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等

### 3.三大核心概念

- 路由：路由是网关最基础的部分，路由信息有一个ID、一个目的URL、一组断言和一组Filter组成。如果断言路由为真，则说明请求的URL和配置匹配
- 断言：Java8中的断言函数。Spring Cloud Gateway中的断言函数输入类型是Spring5.0框架中的ServerWebExchange。Spring Cloud Gateway中的断言函数允许开发者去定义匹配来自于http request中的任何信息，比如请求头和参数等
- 过滤器：一个标准的Spring webFilter。Spring cloud gateway中的filter分为两种类型的Filter，分别是Gateway Filter和Global Filter。过滤器Filter将会对请求和响应进行修改处理

![在这里插入图片描述](https://img-blog.csdnimg.cn/04e2e1c39b5745769c1707ea0dda12cd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 二、案例

### 1.环境搭建

![在这里插入图片描述](https://img-blog.csdnimg.cn/ac702d0c826c443da034830118e497b3.png#pic_center)


#### gateway-server模块

> pom文件

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
```

> 启动类

```java
@SpringBootApplication
@EnableEurekaServer
public class GatewayServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayServerApplication.class, args);
    }
}
```

> 配置文件

```yml
server:
  port: 8761
eureka:
  instance:
    hostname: localhost # 主机名
  client:
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka # eureka服务端地址，将来客户端使用该地址和eureka进行通信
    enabled: false  #默认值为true，设置为false，关闭自我保护，线上建议都是打开
```

#### gateway-provider模块

> pom文件

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
    </dependencies>
```

> controller

```java
@RestController
@RequestMapping("/goods")
public class GoodsController {

    @GetMapping("/findOne/{id}")
    @HystrixCommand(fallbackMethod = "findOne_fallback",commandProperties = {
            //设置Hystrix的超时时间，默认1s
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public Goods findOne(@PathVariable("id") int id){
//        //1.造个异常
//        int i = 3/0;
//        try {
//            //2. 休眠2秒
//            Thread.sleep(2000);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
        Goods goods = new Goods(id, "华为手机", 3999, 10000);
        goods.setTitle(goods.getTitle() + ":" + "8899");//
        return goods;
    }

    /**
     * 定义降级方法：
     *  1. 方法的返回值需要和原方法一样
     *  2. 方法的参数需要和原方法一样
     */
    public Goods findOne_fallback(int id){
        Goods goods = new Goods();
        goods.setTitle("降级了~~~provider03");

        return goods;
    }
}
```

> pojo

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Goods {
    private int id;
    private String title;//商品标题
    private double price;//商品价格
    private int count;//商品库存
}
```

> 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker//开启对hystrix的支持
public class GatewayProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayProviderApplication.class, args);
    }

}
```

> 配置文件

```yml
server:
  port: 8002
spring:
  application:
    name: gateway-provider # 设置当前应用的名称。将来会在eureka中Application显示。将来需要使用该名称来获取路径
eureka:
  instance:
    hostname: localhost # 主机名
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka # eureka服务端地址，将来客户端使用该地址和eureka进行通信
```

#### gateway-consumer模块

> pom文件

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
    </dependencies>
```

> controller

```java
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    private GoodsFeignClient goodsFeignClient;

    @GetMapping("/goods/{id}")
    public Goods findOne(@PathVariable("id") int id){
        return goodsFeignClient.findOne(id);
    } 
}
```

> 实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Goods {
    private int id;
    private String title;//商品标题
    private double price;//商品价格
    private int count;//商品库存
}
```

> feign接口

```java
@FeignClient(value = "gateway-provider")
public interface GoodsFeignClient {

    @GetMapping("/goods/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id);
}
```

> 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableFeignClients//开启feign接口的支持
public class GatewayConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayConsumerApplication.class, args);
    }

}
```

> 配置类

```yml
server:
  port: 8001
spring:
  application:
    name: gateway-consumer # 设置当前应用的名称。将来会在eureka中Application显示。将来需要使用该名称来获取路径
eureka:
  instance:
    hostname: localhost # 主机名
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka # eureka服务端地址，将来客户端使用该地址和eureka进行通信
# 开启feign对hystrix的支持
feign:
  hystrix:
    enabled: true
```

#### gateway模块

> pom文件

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--引入gateway 网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
    </dependencies>
```

> 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```

> 配置类

```yml
server:
  port: 80

spring:
  application:
    name: gateway
  cloud:
    # 网关配置
    gateway:
      # 路由配置：转发规则
      routes: #集合。
        # id: 唯一标识。默认是一个UUID
        # uri: 转发路径
        # predicates: 条件,用于请求网关路径的匹配规则
        - id: gateway-provider
          uri: http://localhost:8001/
          predicates:
            - Path=/order/**
```

#### 测试

```json
请求地址(get)： 127.0.0.1:80/order/goods/1
放回数据:
{
	"id": 1,
	"title": "华为手机:8899",
	"price": 3999,
	"count": 10000
}
```

### 2.Gateway-静态路由

application.yml 中的uri是写死的，就是静态路由

```yml
spring:
  application:
    name: gateway
  cloud:
    # 网关配置
    gateway:
      # 路由配置：转发规则
      routes: #集合。
        # id: 唯一标识。默认是一个UUID
        # uri: 转发路径
        # predicates: 条件,用于请求网关路径的匹配规则
        - id: gateway-provider
          uri: http://localhost:8001/  #这里路径写死啦 
          predicates:
            - Path=/order/**
```

### 3.Gateway-动态路由

- **启动类添加@EnableEurekaClient**

- **引入eureka-client配置 , 在application.yml 中修改uri属性：`uri: lb://服务名称`**

```yml
spring:
  application:
    name: gateway
  cloud:
    # 网关配置
    gateway:
      # 路由配置：转发规则
      routes: #集合。
        # id: 唯一标识。默认是一个UUID
        # uri: 转发路径
        # predicates: 条件,用于请求网关路径的匹配规则
        - id: gateway-provider
          # 静态路由
          #uri: http://localhost:8001/
          # 动态路由
          uri: lb://GATEWAY-CONSUMER
          predicates:
            - Path=/order/**
```

### 4.Gateway-微服务名称配置

```yml
server:
  port: 80

spring:
  application:
    name: gateway
  cloud:
    # 网关配置
    gateway:
      # 路由配置：转发规则
      routes: #集合。
        # id: 唯一标识。默认是一个UUID
        # uri: 转发路径
        # predicates: 条件,用于请求网关路径的匹配规则
        - id: gateway-provider
          # 静态路由
          #uri: http://localhost:8001/
          # 动态路由
          uri: lb://GATEWAY-CONSUMER
          predicates:
            - Path=/order/**
      discovery:
        locator:
          enabled: true #设置为true 请求路径前可以添加微服务名称
          lower-case-service-id: true  #允许小写
```

> 测试

```json
请求地址(get)： 127.0.0.1:80/gateway/order/goods/1
放回数据:
{
	"id": 1,
	"title": "华为手机:8899",
	"price": 3999,
	"count": 10000
}
```
### 5.Gateway-微服务网关跨域
```yml
      globalcors:
        cors-configurations:
          '[/**]': # 匹配所有请求
            allowedOrigins: "*" #跨域处理 允许所有的域
            allowedMethods: # 支持的方法
            - GET
            - POST
            - PUT
            - DELETE

```
## 三、Gateway过滤器

### 1.概述

gateway支持过滤功能，对请求或响应进行拦截，完成一些通用操作。

gateway提供俩种过滤器方式：“pre”和“post”

- pre过滤器，在转发之前执行，可以做参数校验，权限校验，流量监控，日志输出，协议转换等
- post过滤器，在响应之前执行，可以做响应内容，响应头的修改，日志输出，流量监控等

gateway还提供了俩种类型过滤器

- GatewayFilter：局部过老板去，针对单个路由
- GolbalFilter：全局过滤器，针对所有路由

### 2.局部过滤器

GatewayFilter 局部过滤器，是针对单个路由的过滤器。

> 修改gateway服务的配置文件

```yml
server:
  port: 80

spring:
  application:
    name: gateway
  cloud:
    # 网关配置
    gateway:
      # 路由配置：转发规则
      routes: #集合。
        # id: 唯一标识。默认是一个UUID
        # uri: 转发路径
        # predicates: 条件,用于请求网关路径的匹配规则
        - id: gateway-provider
          # 静态路由
          #uri: http://localhost:8001/
          # 动态路由
          uri: lb://GATEWAY-PROVIDER
          predicates:
            - Path=/goods/**
          filters:
            - AddRequestParameter=username,zhangsan
      discovery:
        locator:
          enabled: true #设置为true 请求路径前可以添加微服务名称
          lower-case-service-id: true  #允许小写
```

> 修改gateway-provider的controller

```java
 @GetMapping("/findOne/{id}/{username}")
    public Goods findOne(@PathVariable("id") int id,@PathVariable("username") String username){
        Goods goods = new Goods(id, "华为手机", 3999, 10000);
        goods.setTitle(goods.getTitle() + ":" + "8899"+username);//
        return goods;
    }
```

> 测试

```json
请求路径：127.0.0.1:80/goods/findOne/1/
响应数据：
{
	"timestamp": "2021-10-22T06:00:05.587+00:00",
	"status": 404,
	"error": "Not Found",
	"message": "",
	"path": "/goods/findOne/1/"
}

请求路径：127.0.0.1:80/goods/findOne/1/zhangsan
响应数据：
{
	"id": 1,
	"title": "华为手机:8899zhangsan",
	"price": 3999,
	"count": 10000
}
```

### 3.全局过滤器

GlobalFilter 全局过滤器，不需要在配置文件中配置，系统初始化时加载，并作用在每个路由上。

自定义全局过滤器步骤：

1. 定义类实现 GlobalFilter 和 Ordered接口
2. 复写方法
3. 完成逻辑处理

![在这里插入图片描述](https://img-blog.csdnimg.cn/cff69d4a58694c8c9b8314d25e690ca2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


```java
package com.itheima.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class MyFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        System.out.println("自定义全局过滤器执行了~~~");
		ServerHttpRequest request = exchange.getRequest();
        InetSocketAddress remoteAddress = request.getRemoteAddress();
        System.out.println("ip:"+remoteAddress.getHostName());
        return chain.filter(exchange);//放行
    }

    /**
     * 过滤器排序
     * @return 数值越小 越先执行
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

### 4.案例

需求 : 对系统的所有微服务进行权限认证 , 只有登录之后的用户才能够访问微服务

```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Autowired
    private RedisTemplate redisTemplate ;

    /**
     *
     * @param exchange  网关与web环境交换机
     * @param chain  过滤器链
     * @return
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        ServerHttpRequest request = exchange.getRequest();
        String method = request.getMethodValue();
        String path = request.getURI().getPath();

        //登陆请求直接放行
        if(method.equalsIgnoreCase("post") && path.endsWith("/user")){
            return chain.filter(exchange);
        }

        //获取请求中携带的token数据
        List<String> headers = request.getHeaders().get("token");

        //用户未携带token , 直接返回错误信息给客户端
        if(headers==null || headers.size()<=0){
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        //获取客户端发送的token
        String token = headers.get(0);
        //判断token是否存在 , 如果不存在说明未登录,或者登陆状态过期
        Boolean flag = redisTemplate.hasKey(token);
        if(!flag){
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        //放行 , 访问资源
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

## 四、Gateway的使用

### 1.通过时间匹配

Predicate 支持设置一个时间，在请求进行转发的时候，可以通过判断在这个时间之前或者之后进行转发。比如我们现在设置只有在 2019 年 1 月 1 日才会转发到我的网站，在这之前不进行转发，我就可以这样配置：

```yml
spring:
  cloud:
    gateway:
      routes:
       - id: time_route
        uri: http://ityouknow.com
        predicates:
         - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

Spring 是通过 ZonedDateTime 来对时间进行的对比，ZonedDateTime 是 Java 8 中日期时间功能里，用于表示带时区的日期与时间信息的类，ZonedDateTime 支持通过时区来设置时间，中国的时区是：`Asia/Shanghai`

After Route Predicate 是指在这个时间之后的请求都转发到目标地址。上面的示例是指，请求时间在 2018 年 1 月 20 日 6 点 6 分 6 秒之后的所有请求都转发到地址http://ityouknow.com。+08:00是指时间和 UTC 时间相差八个小时，时间地区为Asia/Shanghai。

添加完路由规则之后，访问地址`http://localhost:8080`会自动转发到`http://ityouknow.com`

efore Route Predicate 刚好相反，在某个时间之前的请求的请求都进行转发。我们把上面路由规则中的 After 改为 Before，如下

```yml
spring:
  cloud:
    gateway:
      routes:
       - id: after_route
        uri: http://ityouknow.com
        predicates:
         - Before=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

就表示在这个时间之前可以进行路由，在这时间之后停止路由，修改完之后重启项目再次访问地址`http://localhost:8080`，页面会报 404 没有找到地址。

除过在时间之前或者之后外，Gateway 还支持限制路由请求在某一个时间段范围内，可以使用 Between Route Predicate 来实现。

```yml
spring:
  cloud:
    gateway:
      routes:
       - id: after_route
        uri: http://ityouknow.com
        predicates:
         - Between=2018-01-20T06:06:06+08:00[Asia/Shanghai], 2019-01-20T06:06:06+08:00[Asia/Shanghai]
```

这样设置就意味着在这个时间段内可以匹配到此路由，超过这个时间段范围则不会进行匹配。通过时间匹配路由的功能很酷，可以用在限时抢购的一些场景中。

### 2.通过 Cookie 匹配

Cookie Route Predicate 可以接收两个参数，一个是 Cookie name , 一个是正则表达式，路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行。

```yml
spring:
  cloud:
    gateway:
      routes:
       - id: cookie_route
         uri: http://ityouknow.com
         predicates:
         - Cookie=ityouknow, kee.e
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080 --cookie "ityouknow=kee.e"
```

则会返回页面代码，如果去掉`--cookie "ityouknow=kee.e"`，后台汇报 404 错误

Header Route Predicate 和 Cookie Route Predicate 一样，也是接收 2 个参数，一个 header 中属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: http://ityouknow.com
        predicates:
        - Header=X-Request-Id, \d+
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080  -H "X-Request-Id:666666" 
```

则返回页面代码证明匹配成功。将参数`-H "X-Request-Id:666666"`改为`-H "X-Request-Id:neo"`再次执行时返回 404 证明没有匹配

### 3.通过 Host 匹配

Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用`.`号作为分隔符。它通过参数中的主机地址作为匹配规则。

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Host=**.ityouknow.com
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080  -H "Host: www.ityouknow.com" 
curl http://localhost:8080  -H "Host: md.ityouknow.com" 
```

经测试以上两种 host 均可匹配到 host_route 路由，去掉 host 参数则会报 404 错误

### 4.通过请求方式匹配

可以通过是 POST、GET、PUT、DELETE 等不同的请求方式来进行路由

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: http://ityouknow.com
        predicates:
        - Method=GET
```

使用 curl 测试，命令行输入:

```
# curl 默认是以 GET 的方式去请求
curl http://localhost:8080
```

测试返回页面代码，证明匹配到路由，我们再以 POST 的方式请求测试。

```
# curl 默认是以 GET 的方式去请求
curl -X POST http://localhost:8080
```

返回 404 没有找到，证明没有匹配上路由

### 5.通过请求路径匹配

Path Route Predicate 接收一个匹配路径的参数来判断是否走路由。

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Path=/foo/{segment}
```

如果请求路径符合要求，则此路由将匹配，例如：/foo/1 或者 /foo/bar

使用 curl 测试，命令行输入:

```
curl http://localhost:8080/foo/1
curl http://localhost:8080/foo/xx
curl http://localhost:8080/boo/xx
```

经过测试第一和第二条命令可以正常获取到页面返回值，最后一个命令报 404，证明路由是通过指定路由来匹配

### 6.通过请求参数匹配

Query Route Predicate 支持传入两个参数，一个是属性名一个为属性值，属性值可以是正则表达式

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://ityouknow.com
        predicates:
        - Query=smile
```

这样配置，只要请求中包含 smile 属性的参数即可匹配路由

使用 curl 测试，命令行输入:

```
curl localhost:8080?smile=x&id=2
```

经过测试发现只要请求汇总带有 smile 参数即会匹配路由，不带 smile 参数则不会匹配

还可以将 Query 的值以键值对的方式进行配置，这样在请求过来时会对属性值和正则进行匹配，匹配上才会走路由

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://ityouknow.com
        predicates:
        - Query=keep, pu.
```

这样只要当请求中包含 keep 属性并且参数值是以 pu 开头的长度为三位的字符串才会进行匹配和路由

使用 curl 测试，命令行输入:

```
curl localhost:8080?keep=pub
```

测试可以返回页面代码，将 keep 的属性值改为 pubx 再次访问就会报 404, 证明路由需要匹配正则表达式才会进行路由。

### 7.通过请求 ip 地址进行匹配

Predicate 也支持通过设置某个 ip 区间号段的请求才会路由，RemoteAddr Route Predicate 接受 cidr 符号 (IPv4 或 IPv6) 字符串的列表(最小大小为 1)，例如 192.168.0.1/16 (其中 192.168.0.1 是 IP 地址，16 是子网掩码)。

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: http://ityouknow.com
        predicates:
        - RemoteAddr=192.168.1.1/24
```

可以将此地址设置为本机的 ip 地址进行测试。

如果请求的远程地址是 192.168.1.10，则此路由将匹配。

### 8.组合使用

上面为了演示各个 Predicate 的使用，我们是单个单个进行配置测试，其实可以将各种 Predicate 组合起来一起使用

例如：

```yml
spring:
  cloud:
    gateway:
      routes:
       - id: host_foo_path_headers_to_httpbin
        uri: http://ityouknow.com
        predicates:
        - Host=**.foo.org
        - Path=/headers
        - Method=GET
        - Header=X-Request-Id, \d+
        - Query=foo, ba.
        - Query=baz
        - Cookie=chocolate, ch.p
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配

> 一个请求满足多个路由的谓词条件时，请求只会被首个成功匹配的路由转发