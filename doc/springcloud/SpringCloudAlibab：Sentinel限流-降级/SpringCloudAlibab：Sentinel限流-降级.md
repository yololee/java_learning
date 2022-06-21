# SpringCloudAlibab：Sentinel限流-降级

## 一、Sentinel 简介

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性

Sentinel 具有以下特征:

- 丰富的应用场景：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等
- 完备的实时监控：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况
- 广泛的开源生态：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel
- 完善的 SPI 扩展点：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等

> hystrix与sentinel的区别

| 功能           | Sentinel                                                   | Hystrix                 |
| :------------- | :--------------------------------------------------------- | :---------------------- |
| 隔离策略       | 信号量隔离（并发线程数限流）                               | 线程池隔离/信号量隔离   |
| 熔断降级策略   | 基于响应时间、异常比率、异常数                             | 基于异常比率            |
| 实时统计实现   | 滑动窗口（LeapArray）                                      | 滑动窗口（基于 RxJava） |
| 动态规则配置   | 支持多种数据源                                             | 支持多种数据源          |
| 扩展性         | 多个扩展点                                                 | 插件的形式              |
| 基于注解的支持 | 支持                                                       | 支持                    |
| 限流           | 基于 QPS，支持基于调用关系的限流                           | 有限的支持              |
| 流量整形       | 支持预热模式、匀速器模式、预热排队模式(流量规则处可配置)   | 不支持                  |
| 系统自适应保护 | 支持                                                       | 不支持                  |
| 控制台         | 提供开箱即用的控制台，可配置规则、查看秒级监控、机器发现等 | 简单的监控查看          |

## 二、安装Sentinel

[安装Sentinel的俩种方式](https://blog.csdn.net/weixin_43296313/article/details/120975362)

## 三、整合Sentinel

### 1.pom文件

```xml
<dependencies>
        <!--sentinel nacos-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
            <version>1.8.2</version>
        </dependency>

        <!--sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>

        <!--spring cloud alibaba-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

### 2.配置类

```yml
server:
  port: 8005
spring:
  application:
    name: sentinel-nacos
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848   #这里是我linux地址
    sentinel:
      transport:
        #配置sentinel地址,端口
        dashboard: 192.168.31.100:8858   #这里是我linux地址
        #客户端IP(sentinel dashboard进行实时监控的主机ip地址)
        # 默认端口8719端口假如被占用会自动从8719开始依次+1扫描，直到找到未被占用的端口
        port: 8719
        client-ip: 172.100.20.220   #这里是我windows地址
      # 取消控制台懒加载
      eager: true
      enabled: true

#暴露自己的端点
management:
  endpoints:
    web:
      exposure:
        include: '*'

```

### 3.controller

```java
@RestController
@Slf4j
public class SentinelController {
    @GetMapping("/sentinel/test")
    public String test(){
        log.info("sentinel test");
        return "sentinel test ";
    }

    @GetMapping("/sentinel/test1")
    public String test1(){
        log.info("sentinel test1");
        return "sentinel test1 ";
    }
}
```

### 4.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class SentinelApplication {

    public static void main(String[] args) {
        SpringApplication.run(SentinelApplication.class, args);
    }

}
```

### 5.查看结果

sentinel默认是采用懒加载的，需要访问controller的接口，才会显示。

> http://127.0.0.1:8005/sentinel/test1
>
> http://127.0.0.1:8005/sentinel/test

![在这里插入图片描述](https://img-blog.csdnimg.cn/ee9ed8667fbf409e9b602f7f1669369d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/6c90908c6c0a4794b966c45ea9bd42eb.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 四、流量控制

流量控制（flow control），其原理是监控应用流量的 QPS 或并发线程数等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

FlowSlot 会根据预设的规则，结合前面 NodeSelectorSlot、ClusterNodeBuilderSlot、StatisticSlot 统计出来的实时信息进行流量控制。
限流的直接表现是在执行 Entry nodeA = SphU.entry(resourceName) 的时候抛出 FlowException 异常。FlowException 是 BlockException 的子类，您可以捕捉 BlockException 来自定义被限流之后的处理逻辑。
同一个资源可以创建多条限流规则。FlowSlot 会对该资源的所有限流规则依次遍历，直到有规则触发限流或者所有规则遍历完毕。
一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：

```java
resource：资源名，即限流规则的作用对象
count: 限流阈值
grade: 限流阈值类型（QPS 或并发线程数）
limitApp: 流控针对的调用来源，若为 default 则不区分调用来源
strategy: 调用关系限流策略
controlBehavior: 流量控制效果（直接拒绝、Warm Up、匀速排队）
```

### 1.默认模式

在完成了上面的两节之后，我们在`sentinel-nacos`服务下，点击`簇点链路`菜单，可以看到如下界面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f0f82529b3b145f9b6972722e9b2641d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


其中`/sentinel/test`接口，就是我们上一节中实现并调用过的接口。通过点击`流控`按钮，来为该接口设置限流规则，比如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff902f276b024b438ee190265f7e65b8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


这里做一个最简单的配置：

- 阈值类型选择：QPS
- 单机阈值：2

综合起来的配置效果就是，该接口的限流策略是每秒最多允许2个请求进入。

点击`新增`按钮之后，可以看到如下界面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/776daa83459f456b834553d243aabfab.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


其实就是左侧菜单中`流控规则`的界面，这里可以看到当前设置的所有限流策略。

> 验证限流规则

在完成了上面所有内容之后，我们可以尝试一下快速的调用这个接口，看看是否会触发限流控制，

![在这里插入图片描述](https://img-blog.csdnimg.cn/6fa17acbcfb741afacda1e0b80e1ff1a.png#pic_center)


### 2.关联模式

当关联的资源达到阈值时，就限流自己

例：当接口`/sentinel/test1`达到qps时会使`/sentinel/test`无法访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/7dfedd85eccf41b0a61a3ade385d040e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


postman测试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/145d8d72f31d4cbab5bac11d1da95168.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/38b8411c75b14103954efbd5a0dfd8d9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


在postman测试期间访问a接口：

![在这里插入图片描述](https://img-blog.csdnimg.cn/cc189ea4252348f9a2828947f83e50d1.png#pic_center)


当postman测试完：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f72bc92b489f43cc8e410a33c3556e06.png#pic_center)


### 3.Warm Up

Sentinel的Warm Up（RuleConstant.CONTROL_BEHAVIOR_WARM_UP）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。warm up冷启动主要用于启动需要额外开销的场景，例如建立数据库连接等。
默认 coldFactor为3，即请求QPS从 threshold/3开始,经预热时长逐渐升至设定的QPS阈值。

配置：`/sentinel/test`接口5秒后qps会达到10，开始qps为3 (10/3)

![在这里插入图片描述](https://img-blog.csdnimg.cn/41e6a6e532a340ac9d27263b5427d19e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


开始快速访问：

![在这里插入图片描述](https://img-blog.csdnimg.cn/95857e21c4e04ad2a88d0ae403c36f88.png#pic_center)


后续快速访问：

![在这里插入图片描述](https://img-blog.csdnimg.cn/44084247113345a89499e47f2967c6f7.png#pic_center)


### 4. 排队等待

匀速排队,让请求以均匀的速度通过,阀值类型必须设成QPS,否则无效。
设置含义: a接口每秒1次请求,超过的话就排队等待,等待的超时时间为5毫秒。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0738a967b9f14436b598e2b3f0901cbe.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 五、降级

### 1.降级规则介绍

![在这里插入图片描述](https://img-blog.csdnimg.cn/0f59deed7d5942f68fe4d266e09a86d8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


Sentinel在1.8.0版本对熔断降级做了大的调整，可以定义任意时长的熔断时间，引入了半开启恢复支持。下面梳理下相关特性。

#### 熔断状态

熔断有三种状态，分别为OPEN、HALF_OPEN、CLOSED

- OPEN：表示熔断开启，拒绝所有请求
- HALF_OPEN：探测恢复状态，如果接下来的一个请求顺利通过则结束熔断，否则继续熔断
- CLOSED：表示熔断关闭，请求顺利通过

#### 熔断策略

**熔断降级支持慢调用比例、异常比例、异常数三种熔断策略**。

**慢调用**：指耗时大于阈值RT的请求称为慢调用，阈值RT由用户设置

**最小请求数**：允许通过的最小请求数量，在最小请求数量内不发生熔断，由用户设置

### 2.慢调用比例

慢调用比例：当资源的响应时间超过最大RT（以ms为单位，最大RT即最大响应时间）之后，资源进入准降级状态。如果接下来1s内持续进入5个请求（最小请求数），它们的RT都持续超过这个阈值，那么在接下来的熔断时长之内，就会对这个方法进行服务降级。其中的“比例阈值”我设置发现无效，下次编辑会重置成1

![在这里插入图片描述](https://img-blog.csdnimg.cn/d48384ab61f44ef197d835a4da328309.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)




> 注意Sentinel默认统计的RT上限是4900ms，超出此阈值的都会算作4900ms，若需要变更此上限可以通过启动配置项-Dcsp.sentinel.statistic.max.rt=xxx来配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/1138764dcc0a4a98a0391bc0ff84cb37.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 3.异常比例

**异常比例**：当资源的每秒请求数大于等于最小请求数，并且异常总数占通过量的比例超过比例阈值时，资源进入降级状态。

![在这里插入图片描述](https://img-blog.csdnimg.cn/78ce4a4c84f349fb8ef91765df1ac99b.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/598c008ec32942bb9aeef894f169cafd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 4.异常数

**异常数**：当资源近1分钟的异常数目超过阈值（异常数）之后会进行服务降级。注意由于统计时间窗口是分钟级别的，若熔断时长小于60s，则结束熔断状态后仍可能再次进入熔断状态。

![在这里插入图片描述](https://img-blog.csdnimg.cn/90fa64f43c2a4e1eb9653f4d5b8d4d77.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/8257b06934fa4722a5fa49e06cedc60a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)