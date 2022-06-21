# SpringCloud：Hystrix组件实现服务熔断

## 一、Hystrix概述

Hystix 是 Netflix 开源的一个延迟和容错库，用于隔离访问远程服务、第三方库，防止出现级联失败（雪崩）

> Hystrix设计目标

- 资源隔离、MQ解耦、不可用服务调用快速失败等。资源隔离通常指不同服务调用采用不同的线程池；不可用服务调用快速失败一般通过熔断器模式结合超时机制实现
- 阻止故障的连锁反应
- 快速失败并迅速恢复
- 回退并优雅降级
- 提供近实时的监控与告警

> Hystrix如何实现这些设计目标？

- 使用命令模式将所有对外部服务（或依赖关系）的调用包装在HystrixCommand或HystrixObservableCommand对象中，并将该对象放在单独的线程中执行
- 每个依赖都维护着一个线程池（或信号量），线程池被耗尽则拒绝请求（而不是让请求排队）
- 记录请求成功，失败，超时和线程拒绝
- 服务错误百分比超过了阈值，熔断器开关自动打开，一段时间内停止对该服务的所有请求
- 请求失败，被拒绝，超时或熔断时执行降级逻辑
- 近实时地监控指标和配置的修改

### 1.雪崩效应

分布式系统环境下，服务间类似依赖非常常见，一个业务调用通常依赖多个基础服务。如下图，对于同步调用，当库存服务不可用时，商品服务请求线程被阻塞，当有大批量请求调用库存服务时，最终可能导致整个商品服务资源耗尽，无法继续对外提供服务。并且这种不可用可能沿请求调用链向上传递，这种现象被称为雪崩效应

![在这里插入图片描述](https://img-blog.csdnimg.cn/a2c4fd2b8cdd405e8422a432e301a195.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_14,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 2.雪崩效应常见场景及解决措施

- 硬件故障：如服务器宕机，机房断电，光纤被挖断等

  ```
  解决措施：多机房容灾、异地多活等
  ```

- 流量激增：如异常流量，重试加大流量等

  ```
  解决措施：服务自动扩容、流量控制（限流、关闭重试）等
  ```

- 缓存穿透：一般发生在应用重启，所有缓存失效时，以及短时间内大量缓存失效时。大量的缓存不命中，使请求直击后端服务，造成服务提供者超负荷运行，引起服务不可用

  ```
  解决措施：缓存预加载、缓存异步加载等
  ```

- 程序BUG：如程序逻辑导致内存泄漏，JVM长时间FullGC等

  ```
  解决措施：修改程序bug、及时释放资源等
  ```

- 同步等待：服务间采用同步调用模式，同步等待造成的资源耗尽

  ```
  解决措施：资源隔离、MQ解耦、不可用服务调用快速失败等。
  		资源隔离通常指不同服务调用采用不同的线程池；
  		不可用服务调用快速失败一般通过熔断器模式结合超时机制实现
  ```

## 二、Hystrix容错

Hystrix的容错主要是通过添加容许延迟和容错方法，帮助控制这些分布式服务之间的交互。 还通过隔离服务之间的访问点，阻止它们之间的级联故障以及提供回退选项来实现这一点，从而提高系统的整体弹性

Hystrix主要提供了以下几种容错方法：

- 资源隔离：
  - 线程池隔离
  - 信号量隔离
- 降级：异常，超时
- 熔断

## 三、Hystrix-资源隔离

### 线程隔离-线程池

Hystrix通过命令模式对发送请求的对象和执行请求的对象进行解耦，将不同类型的业务请求封装为对应的命令请求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210407224523206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI5NjMxMw==,size_16,color_FFFFFF,t_70#pic_center)

- Hystrix为每个依赖服务调用分配一个小的线程池，如果线程池已满调用将被立即拒绝，默认不采用排队，加速失败判定时间
- 用户的请求将不再直接访问服务，而是通过线程池中的空闲线程来访问服务，如果线程池已满，或者请求超时，则会进行降级处理

### 线程隔离-信号量

当依赖延迟极低的服务时，线程池隔离技术引入的开销超过了它所带来的好处。这时候可以使用信号量隔离技术来代替，通过设置信号量来限制对任何给定依赖的并发调用量

```java
public class QueryByOrderIdCommandSemaphore extends HystrixCommand<Integer> {
    private final static Logger logger = LoggerFactory.getLogger(QueryByOrderIdCommandSemaphore.class);
    private OrderServiceProvider orderServiceProvider;
 
    public QueryByOrderIdCommandSemaphore(OrderServiceProvider orderServiceProvider) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("orderService"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("queryByOrderId"))
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withCircuitBreakerRequestVolumeThreshold(10)至少有10个请求，熔断器才进行错误率的计算
                        .withCircuitBreakerSleepWindowInMilliseconds(5000)//熔断器中断请求5秒后会进入半打开状态,放部分流量过去重试
                        .withCircuitBreakerErrorThresholdPercentage(50)//错误率达到50开启熔断保护
                        .withExecutionIsolationStrategy(HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE)
                        .withExecutionIsolationSemaphoreMaxConcurrentRequests(10)));//最大并发请求量
        this.orderServiceProvider = orderServiceProvider;
    }
 
    @Override
    protected Integer run() {
        return orderServiceProvider.queryByOrderId();
    }
 
    @Override
    protected Integer getFallback() {
        return -1;
    }
}
```

由于Hystrix默认使用线程池做线程隔离，使用信号量隔离需要显示地将属性execution.isolation.strategy设置为ExecutionIsolationStrategy.SEMAPHORE，同时配置信号量个数，默认为10。客户端需向依赖服务发起请求时，首先要获取一个信号量才能真正发起调用，由于信号量的数量有限，当并发请求量超过信号量个数时，后续的请求都会直接拒绝，进入fallback流程。

信号量隔离主要是通过控制并发请求量，防止请求线程大面积阻塞，从而达到限流和防止雪崩的目的

### 线程隔离总结

|        | 线程切换 | 支持异步 | 支持超时 | 支持熔断 | 限流 | 开销 |
| ------ | -------- | -------- | -------- | -------- | ---- | ---- |
| 信号量 | 否       | 否       | 否       | 是       | 是   | 小   |
| 线程池 | 是       | 是       | 是       | 是       | 是   | 大   |

## 四、Hystrix-降级实现

### 环境搭建

![在这里插入图片描述](https://img-blog.csdnimg.cn/700e921b3f22418f93f545a5e8f68abc.png#pic_center)


具体如何搭建，[eureka环境搭建](https://blog.csdn.net/weixin_43296313/article/details/120847896?spm=1001.2014.3001.5501)

### 1.服务端降级

> 在服务提供方，引入 hystrix 依赖

```xml
	<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
```

> 定义降级方法

```java
/**
 * 定义降级方法：
 *  1. 方法的返回值需要和原方法一样
 *  2. 方法的参数需要和原方法一样
 */
public Goods findOne_fallback(int id){
    Goods goods = new Goods();
    goods.setTitle("降级了~~~");

    return goods;
}
```

> 使用 @HystrixCommand 注解配置降级方法

```java
/**
     * 降级：
     *  1. 出现异常
     *  2. 服务调用超时
     *      * 默认1s超时
     *
     *  @HystrixCommand(fallbackMethod = "findOne_fallback")
     *      fallbackMethod：指定降级后调用的方法名称
     */
    @GetMapping("/findOne/{id}")
    @HystrixCommand(fallbackMethod = "findOne_fallback",commandProperties = {
            //设置Hystrix的超时时间，默认1s
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public Goods findOne(@PathVariable("id") int id){
        //1.造个异常
        int i = 3/0;
        try {
            //2. 休眠2秒
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Goods goods = new Goods(id, "苹果手机", 3999, 10000);
        goods.setTitle(goods.getTitle() + ":" + "8899");//
        return goods;
    }
```

> 在启动类上开启Hystrix功能：@EnableCircuitBreaker

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker//// 开启Hystrix功能
public class HystrixProvider02Application {

    public static void main(String[] args) {
        SpringApplication.run(HystrixProvider02Application.class, args);
    }
}
```

> 测试结果

```json
get请求：127.0.0.1:8002/order/goods/1
响应结果：
{
	"id": 0,
	"title": "降级了~~~provider02",
	"price": 0,
	"count": 0
}
```

### 2.消费端降级

消费方一般使用feign调用服务, feign 组件中已经集成了 hystrix 组件。我们不需要再引入依赖

> 定义feign 调用接口实现类，复写方法，即 降级方法

```java
package com.mye.hystrixconsumer.feign;
import com.mye.hystrixconsumer.pojo.Goods;
import org.springframework.stereotype.Component;

/**
 * Feign 客户端的降级处理类
 * 1. 定义类 实现 Feign 客户端接口
 * 2. 使用@Component注解将该类的Bean加入SpringIOC容器
 */
@Component
public class GoodsFeignClientFallback implements GoodsFeignClient {

    @Override
    public Goods findOne(int id) {
        Goods goods = new Goods();
        goods.setTitle("又被降级了~~~");
        return goods;
    }
}
```

> 在 @FeignClient 注解中使用 fallback 属性设置降级处理类。

```java
package com.mye.hystrixconsumer.feign;
import com.mye.hystrixconsumer.config.FeignLogConfig;
import com.mye.hystrixconsumer.pojo.Goods;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 *
 * feign声明式接口。发起远程调用的。
 *
 String url = "http://FEIGN-PROVIDER/goods/findOne/"+id;
 Goods goods = restTemplate.getForObject(url, Goods.class);
 *
 * 1. 定义接口
 * 2. 接口上添加注解 @FeignClient,设置value属性为 服务提供者的 应用名称
 * 3. 编写调用接口，接口的声明规则 和 提供方接口保持一致。
 * 4. 注入该接口对象，调用接口方法完成远程调用
 */
@FeignClient(value = "EUREKA-PROVIDER", configuration = FeignLogConfig.class,fallback = GoodsFeignClientFallback.class)
public interface GoodsFeignClient {

    @GetMapping("/goods/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id);
}

```

> 配置开启 `feign.hystrix.enabled = true`

```yml
# 开启feign对hystrix的支持
feign:
  hystrix:
    enabled: true
```

> 测试结果

```json
get请求：127.0.0.1:8002/order/goods/1
响应结果：
{
	"id": 0,
	"title": "又被降级了~~~",
	"price": 0,
	"count": 0
}
```

## 五、Hystrix-熔断

### 1.熔断器简介

Hystrix在运行过程中会向每个commandKey对应的熔断器报告成功、失败、超时和拒绝的状态，熔断器维护并统计这些数据，并根据这些统计信息来决策熔断开关是否打开。如果打开，熔断后续请求，快速返回。隔一段时间（默认是5s）之后熔断器尝试半开，放入一部分流量请求进来，相当于对依赖服务进行一次健康检查，如果请求成功，熔断器关闭

> 修改服务提供者中的controller

```java
@GetMapping("/{id}")
    @HystrixCommand(fallbackMethod = "circuitBreakerFallback",commandProperties = {
            //开启熔断
            @HystrixProperty(name="circuitBreaker.enabled",value = "true"),
            //设置Hystrix的超时时间，默认1s
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value = "3000"),
            //监控时间 默认5000 毫秒
            @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds",value = "5000"),
            //失败次数。默认20次
            @HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value = "10"),
            //失败率 默认50%
            @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value = "50") })
    public String circuitBreaker(@PathVariable("id") Integer id) {
        if (id<0){
            throw  new RuntimeException("id ="+id+"，不能为负数");
        }
        return "调用成功 id=" + id;
    }
    public String circuitBreakerFallback(@PathVariable("id") Integer id) {
        return "id =" + id + ", 不能为负数";
    }
```

> 修改服务消费者的feign接口

```java
@FeignClient(value = "EUREKA-PROVIDER",
        configuration = FeignLogConfig.class
        )
public interface GoodsFeignClient {

    @GetMapping("/goods/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id);

    @GetMapping("/goods/threeseconds")
    public String threeseconds();

    @GetMapping("/goods/{id}")
    public String circuitBreaker(@PathVariable("id") Integer id);
}
```

> 修改服务消费者的controller

```java
@GetMapping("/{id}")
    public String circuitBreaker(@PathVariable("id") int id){
        return goodsFeignClient.circuitBreaker(id);
    }
```

这里需要注意的是在配置文件中开启feign对hystrix的支持

```java
feign:
  hystrix:
    enabled: true
```

注意 : 以上配置如果配置在`@HystrixCommand`注解中, 只对当前方法有效, 如果想对所有控制方法配置降级参数, 可以在application.yml总统一配置 , 配置如下 :

```yml
hystrix:
  command:
    default:
      circuitBreaker:
        errorThresholdPercentage: 50 # 触发熔断错误比例阈值，默认值50%
        sleepWindowInMilliseconds: 10000 # 熔断后休眠时长，默认值5秒
        requestVolumeThreshold: 10 # 熔断触发最小请求次数，默认值是20
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000 # 熔断超时设置，默认为1秒
```

### 2.测试

> 正常访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/7b5c2b1cc7fa46e6844b5344b59f2611.png#pic_center)


> 错误访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/09f17f40fdeb495d8176b756bb7d7169.png#pic_center)


> 多次错误访问之后

![在这里插入图片描述](https://img-blog.csdnimg.cn/f035e5e7e97745edaf06edd0c4806992.png#pic_center)


## 六、HystrixDashboard-服务监控

创建一个hystrix-dashboard模块

> pom文件

```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-hystrix-dashboard</artifactId>
            <version>2.2.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
```

> application.yml

```yml
server:
  port: 9001

hystrix:
  dashboard:
    proxy-stream-allow-list: "localhost"
```

> 启动类

```java
package com.mye.hystrixdashboard;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class, args);
    }

}
```

> 访问浏览器

地址：http://127.0.0.1:9001/hystrix

![在这里插入图片描述](https://img-blog.csdnimg.cn/c03308d6161b407f8b29e78537070f17.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


> 在服务提供者里添加HystrixConfig配置文件，重启服务

```java
@Configuration
public class HystrixConfig {
    @Bean
    public ServletRegistrationBean myServlet(){
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```

> dashboard添加hystrix-provider02

```
http://localhost:8001/hystrix.stream
这里端口为服务提供者的端口
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/38cd6509618041e289e80d0d44e603f5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)




![在这里插入图片描述](https://img-blog.csdnimg.cn/323e9337b32f47f8ae3458ca1d2d5c14.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)