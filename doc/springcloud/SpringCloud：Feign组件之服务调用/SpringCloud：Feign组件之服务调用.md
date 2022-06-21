# SpringCloud：Feign组件之服务调用

## 一、Feign概述


### 1.Feign是什么

通过RestTemplate调用其它服务的API时，所需要的参数须在请求的URL中进行拼接，如果参数少的话或许我们还可以忍受，一旦有多个参数的话，这时拼接请求字符串就会效率低下

Feign 是一个声明式的 REST 客户端，它用了基于接口的注解方式，很方便实现客户端配置

而Feign则会完全代理HTTP请求，我们只需要像调用方法一样调用它就可以完成服务请求及相关处理。Feign整合了Ribbon和Hystrix，可以让我们不再需要显式地使用这两个组件

### 2.Feign集成了 Ribbon

利用 Ribbon维护了 Paymente的服务列表信息,并且通过轮询实现了客户端的负载均衡。而与 Ribbon不同的是,通过fign只需要定义服务绑定接口且以声明式的方法,优雅而简单的实现了服务调用

### 3.Fegin和OpenFegion区别

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是SpringCloud组件的一个清量级restful的http服务客户端Feign内置了Ribbon，用来做客户端均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign是SpringCloud在Feign的基础上支持了SpringMVC的注解，如@RequestMapper等，OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapper注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用到其他服务 |
| `<dependency><groupId>org.springframework.cloud</groupId>    <artifactId>spring-cloud-starter-feign</artifactId>    <version>1.4.7.RELEASE</version></dependency>` | `<dependency>     <groupId>org.springframework.cloud</groupId>     <artifactId>spring-cloud-starter-openfeign</artifactId>     <version>2.2.6.RELEASE</version> </dependency> ` |



## 二、快速入门

### 1.项目结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/fbff48cca30d418eaed63345d2175b0d.png#pic_center)


- eureka-server：注册中心
- eureka-comsumer：服务消费端
- eureka-client，eureka-provider：服务提供端

### 2.消费端引入依赖

```xml
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
```

### 3.定义feign接口

这里feign接口里面的方法要跟服务提供者的接口保存一致

```java
package com.mye.eurekaconsumer.feign;



import com.mye.eurekaconsumer.pojo.Goods;
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
@FeignClient(value = "EUREKA-PROVIDER")
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id);
}
```

### 4.修改controller

```java
package com.mye.eurekaconsumer.controller;

import com.mye.eurekaconsumer.feign.GoodsFeignClient;
import com.mye.eurekaconsumer.pojo.Goods;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequestMapping("/order")
public class OrderController {
    @Autowired
    private GoodsFeignClient goodsFeignClient;

    @GetMapping("/goods/{id}")
    public Goods findOne(@PathVariable("id") int id){
        return goodsFeignClient.findOne(id);
    }
}
```

### 5.在启动类 添加 @EnableFeignClients 注解，开启Feign功能

```java
package com.mye.eurekaconsumer;

import com.mye.eurekaconsumer.config.MySelfRule;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.ribbon.RibbonClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
/*
    配置Ribbon的负载均衡策略
    name：设置服务提供方的应用名称
    configuration:设置负载均衡的Bean
 */
@RibbonClient(name="EUREKA-PROVIDER",configuration= MySelfRule.class)
@EnableFeignClients//开启Feign的功能
public class EurekaConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }

}
```

## 三、 Feign超时配置


Feign 底层依赖于 Ribbon 实现负载均衡和远程调用。

> 修改服务提供者的controller

```java
	@GetMapping("/threeseconds")
    public String threeseconds(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "hello word";
    }
```

> 修改controller层的feign接口

```java
@FeignClient(value = "EUREKA-PROVIDER",
        configuration = FeignLogConfig.class
        )
public interface GoodsFeignClient {

    @GetMapping("/goods/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id);

    @GetMapping("/goods/threeseconds")
    public String threeseconds();
}
```

现在启动服务访问，会出现连接超时的错误

![在这里插入图片描述](https://img-blog.csdnimg.cn/a5b4484507384812accb3c90da4b19fd.png#pic_center)


>  修改配置文件

Ribbon默认1秒超时。

```yml
# 设置Ribbon的超时时间
ribbon:
  ConnectTimeout: 1000 # 连接超时时间 默认1s  默认单位毫秒
  ReadTimeout: 3000 # 逻辑处理的超时时间 默认1s 默认单位毫秒
```

这里idea没有给提示，直接用就可以

因为feign集成了ribbon，可以这样配置

```yml
feign:
  hystrix:
    enabled: false
  client:
    config:
      default:
        #建立连接所用的时间，适用于网络状况正常的情况下，两端连接所需要的时间
        ConnectTimeOut: 4000
        #指建立连接后从服务端读取到可用资源所用的时间
        ReadTimeOut: 10000
```

==这里要注意关闭feign对hystrix的支持，不然会报错==

## 四、Feign日志记录

Feign 只能记录 debug 级别的日志信息

```yml
# 设置当前的日志级别 debug，feign只支持记录debug级别的日志
logging:
  level:
    com.mye.eurekaconsumer: debug
```

**定义Feign日志级别Bean**

```java
package com.mye.eurekaconsumer.config;
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignLogConfig {
    /*
        NONE,不记录
        BASIC,记录基本的请求行，响应状态码数据
        HEADERS,记录基本的请求行，响应状态码数据，记录响应头信息
        FULL;记录完成的请求 响应数据
     */
    @Bean
    public Logger.Level level(){
        return Logger.Level.FULL;
    }
}
```

**启动bean**

```java
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
@FeignClient(value = "EUREKA-PROVIDER", configuration = FeignLogConfig.class)
public interface GoodsFeignClient {
    @GetMapping("/goods/findOne/{id}")
    public Goods findOne(@PathVariable("id") int id);
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae233a328e9545a28b3c0a52389b0c65.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)