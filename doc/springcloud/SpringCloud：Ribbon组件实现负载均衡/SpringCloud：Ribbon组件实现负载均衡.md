# SpringCloud：Ribbon组件实现负载均衡

## 一、Ribbon简介

		Ribbon是Netflix发布的云中间层服务开源项目，其主要功能是提供客户端实现负载均衡算法。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，Ribbon是一个客户端负载均衡器，我们可以在配置文件中Load Balancer后面的所有机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器，我们也很容易使用Ribbon实现自定义的负载均衡算法

Ribbon是一个为客户端提供负载均衡功能的服务，它内部提供了一个叫做ILoadBalance的接口代表负载均衡器的操作，比如有添加服务器操作、选择服务器操作、获取所有的服务器列表、获取可用的服务器列表等等

## 二、案例

### 1.项目结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/8ca6117bc45e4374ba220abc3f0218b3.png#pic_center)


- eureka-server作为注册中心
- eureka-client、eureka-provider作为服务提供者
- eureka-consumer作为服务消费者

### 2.导入依赖

因为Eureka中已经集成了Ribbon，所以我们无需引入新的依赖

![在这里插入图片描述](https://img-blog.csdnimg.cn/012c2e3e514f42fc9e9547f9d1e0c3ac.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 3.使用Ribbon

在restTemplate类上加上@LoadBalanced注解

```java
	@Bean
    @LoadBalanced//开启负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
```

> 以前的controller

这里是直接调用http://localhost:8761这个服务下面的接口，如果这里用到了@LoadBalanced，然后这里不进行修改会出现错误

```java
@GetMapping("/goods/{id}")
    public Goods findGoodsById(@PathVariable("id") int id){
        String url = "http://localhost:8761/goods/findOne/"+id;
        return restTemplate.getForObject(url, Goods.class);

    }
```

> 修改之后的controller

```java
@GetMapping("/goods/{id}")
    public Goods findGoodsById(@PathVariable("id") int id){
        String url = "http://EUREKA-PROVIDER/goods/findOne/"+id;
        return restTemplate.getForObject(url, Goods.class);
    }
```

这里的`EUREKA-PROVIDER`是控制台中容器的名称

![在这里插入图片描述](https://img-blog.csdnimg.cn/641a5e0da7c34a7e954a0fbc2b924d36.png#pic_center)


==而且这里需要注意的是一个消费者从俩个提供者中采用Ribbon来进行拿数据，这俩个ribbon的容器名字应该一样==

**其中{Spring.application.name}都是一样的，不可以变**

![在这里插入图片描述](https://img-blog.csdnimg.cn/37f6cf6ef04b4b368ac8556ba8137bf5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 三、Ribbon组件IRule

默认的是RoundBobinRule(轮询)

| 命名     | **内置负载均衡规则类**    | **规则描述**                                                 |
| -------- | ------------------------- | ------------------------------------------------------------ |
| 轮询     | RoundRobinRule            | 简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| 可用过滤 | AvailabilityFilteringRule | 对以下两种服务器进行忽略：<br>（1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。<br>（2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。 |
| 权重     | WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| 区域权衡 | ZoneAvoidanceRule         | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。 |
| 最低并发 | BestAvailableRule         | 忽略哪些短路的服务器，并选择并发数较低的服务器。             |
| 随机     | RandomRule                | 随机选择一个可用的服务器。                                   |
| 重试     | RetryRule                 | 在一个配置时间段内，当选择server不成功，则一直尝试选择一个可用的server |

### 1.RetryRule

- 先按照RoundRobinRule(轮询)的策略获取服务，如果获取的服务失败侧在指定的时间会进行重试，进行获取可用的服务
- 如多次获取某个服务失败，这不会再再次获取该服务

### 2.使用

```java
@Bean
    @LoadBalanced//开启负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule myRule(){
        return new RoundRobinRule();
//        return new RandomRule();
//        return new RetryRule();
    }
```

## 四、自定义负载均衡算法

所谓的自定义Ribbon Client的主要作用就是使用自定义配置替代Ribbon默认的负载均衡策略，注意：自定义的Ribbon Client是有针对性的，一般一个自定义的Ribbon Client是对一个服务提供者(包括服务名相同的一系列副本)而言的。自定义了一个Ribbon Client 它所设定的负载均衡策略只对某一特定服务名的服务提供者有效，但不能影响服务消费者与别的服务提供者通信所使用的策略。

> MySelfRule.java

```java
@Configuration
public class MySelfRule{
	@Bean
	public IRule myRule(){	
		return new RandomRule_ZY();  // 我自定义为每台机器5次，5次之后在轮询到下一个
	}
}
```

> springboot主程序

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
/*
    配置Ribbon的负载均衡策略
    name：设置服务提供方的应用名称
    configuration:设置负载均衡的Bean
 */
@RibbonClient(name="EUREKA-PROVIDER",configuration= MySelfRule.class)
public class EurekaConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaConsumerApplication.class, args);
    }
}
```

> 自定义LoadBalance

```java
public class RandomRule_ZY extends AbstractLoadBalancerRule{
 
	// total = 0 // 当total==5以后，我们指针才能往下走，
	// index = 0 // 当前对外提供服务的服务器地址，
	// total需要重新置为零，但是已经达到过一个5次，我们的index = 1
	// 分析：我们5次，但是微服务只有8001 8002 8003 三台，OK？
	
	private int total = 0; 	    // 总共被调用的次数，目前要求每台被调用5次
	private int currentIndex = 0;	    // 当前提供服务的机器号
 
	public Server choose(ILoadBalancer lb, Object key){
		if (lb == null) {
			return null;
		}
		Server server = null;
 
		while (server == null) {
			if (Thread.interrupted()) {
				return null;
			}
			List<Server> upList = lb.getReachableServers();  //激活可用的服务
			List<Server> allList = lb.getAllServers();  //所有的服务
 
			int serverCount = allList.size();
			if (serverCount == 0) {
				return null;
			}
		
                        if(total < 5){
	                    server = upList.get(currentIndex);
	                    total++;
                        }else {
	                    total = 0;
	                    currentIndex++;
	                    if(currentIndex >= upList.size()){
	                      currentIndex = 0;
	                    }
                        }							
			if (server == null) {
				Thread.yield();
				continue;
			}
 
			if (server.isAlive()) {
				return (server);
			}
 
			// Shouldn't actually happen.. but must be transient or a bug.
			server = null;
			Thread.yield();
		}
		return server;
	}
	@Override
	public Server choose(Object key){
		return choose(getLoadBalancer(), key);
	}
 
	@Override
	public void initWithNiwsConfig(IClientConfig clientConfig){
	}
}
```