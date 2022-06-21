# SpringCloud：Gateway之限流

我们之前说过，网关可以做很多的事情，比如，限流，当我们的系统 被频繁的请求的时候，就有可能 将系统压垮，所以 为了解决这个问题，需要在每一个微服务中做限流操作，但是如果有了网关，那么就可以在网关系统做限流，因为所有的请求都需要先通过网关系统才能路由到微服务中

## 1 思路分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/0c499b4152b04ea39e62f107f5a0f0d6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 2 令牌桶算法

令牌桶算法是比较常见的限流算法之一，大概描述如下：

1. 所有的请求在处理之前都需要拿到一个可用的令牌才会被处理；
2. 根据限流大小，设置按照一定的速率往桶里添加令牌；
3. 桶设置最大的放置令牌限制，当桶满时、新添加的令牌就被丢弃或者拒绝；
4. 请求达到后首先要获取令牌桶中的令牌，拿着令牌才可以进行其他的业务逻辑，处理完业务逻辑之后，将令牌直接删除；
5. 令牌桶有最低限额，当桶中的令牌达到最低限额的时候，请求处理完之后将不会删除令牌，以此保证足够的限流

![在这里插入图片描述](https://img-blog.csdnimg.cn/9fa2307d13c941999f32122e6686f9b1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


这个算法的实现，有很多技术，Guava(读音: 瓜哇)是其中之一，redis客户端也有其实现。

## 3 网关限流代码实现

需求：每个ip地址1秒内只能发送1次请求，多出来的请求返回429错误。

### 1.在nacos-provider模块中添加controller

```java
@RestController
@RequestMapping("/goods")
public class GoodsController {

    @GetMapping("/findOne/{id}/{username}")
    public String findOne(@PathVariable("id") int id,@PathVariable("username") String username){
        return id + "=====" + username;
    }
}
```

### 2.创建nacos-gateway

> pom文件

```xml
<dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--引入gateway 网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
    </dependencies>
```

> 启动类

```java
@SpringBootApplication
public class NacosGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosGatewayApplication.class, args);
    }

    //定义一个KeyResolver
    @Bean
    public KeyResolver ipKeyResolver() {
        return new KeyResolver() {
            @Override
            public Mono<String> resolve(ServerWebExchange exchange) {
                return Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
            }
        };
    }

}
```

> application.yml

```yml
server:
  port: 80

spring:
  application:
    name: nacos-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    # 网关配置
    gateway:
      httpclient:
        pool:
          max-idle-time: 10000
      # 路由配置：转发规则
      routes: #集合。
        # id: 唯一标识。默认是一个UUID
        # uri: 转发路径
        # predicates: 条件,用于请求网关路径的匹配规则
        - id: nacos-provider
          # 静态路由
          #uri: http://localhost:8001/
          # 动态路由
          uri: lb://provider    #provider这个为nacos中的服务名
          predicates:
            - Path=/goods/**
          filters:
            - name: RequestRateLimiter  #请求数限流，名字不能随便写
              args:
                key-resolver: "#{@ipKeyResolver}"
                redis-rate-limiter.replenishRate: 1
                redis-rete-limiter.burstCapacity: 1
  redis:
    host: 127.0.0.1
    port: 6379
```

解释：

- burstCapacity：令牌桶总容量。
- replenishRate：令牌桶每秒填充平均速率。
- key-resolver：用于限流的键的解析器的 Bean 对象的名字。它使用 SpEL 表达式根据#{@beanName}从 Spring 容器中获取 Bean 对象。

通过在replenishRate和中设置相同的值来实现稳定的速率burstCapacity。设置burstCapacity高于时，可以允许临时突发replenishRate。在这种情况下，需要在突发之间允许速率限制器一段时间（根据replenishRate），因为2次连续突发将导致请求被丢弃（HTTP 429 - Too Many Requests）

key-resolver: “#{@userKeyResolver}” 用于通过SPEL表达式来指定使用哪一个KeyResolver.

如上配置：

表示 一秒内，允许 一个请求通过，令牌桶的填充速率也是一秒钟添加一个令牌。

最大突发状况 也只允许 一秒内有一次请求，可以根据业务来调整 。

### 3.测试



![在这里插入图片描述](https://img-blog.csdnimg.cn/83cd9c8aeeb0450ebd41038aa19af69f.png#pic_center)


快速刷新，当1秒内发送多次请求，就会返回429错误。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a4b0d62e64d4d46bf1880f2008aeb5a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_13,color_FFFFFF,t_70,g_se,x_16#pic_center)