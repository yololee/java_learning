# SpringCloudAlibaba：Sentinel使用nacos进行持久化

## 一、Sentinel持久化介绍

Sentinel自身就支持了多种不同的数据源来持久化规则配置，目前包括以下几种方式：

- 文件配置
- Nacos配置
- ZooKeeper配置
- Apollo配置

这里我们整合的配置中心`Nacos`存储限流规则。

## 二、准备工作

下面我们将同时使用到`Nacos`和`Sentinel Dashboard`，所以可以先把`Nacos`和`Sentinel Dashboard`启动起来。

默认配置下启动后，它们的访问地址（后续会用到）为：

- Nacos：192.168.31.100:8848/nacos/
- Sentinel Dashboard：192.168.31.100:8858/

## 三、整合nacos存储规则

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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
```

### ==2.bootstrap.yml==

> 这里必须为bootstrap格式，不然获取不到数据

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
        dashboard: 192.168.31.100:8859   #这里是我linux地址
        #客户端IP(sentinel dashboard进行实时监控的主机ip地址)
        # 默认端口8719端口假如被占用会自动从8719开始依次+1扫描，直到找到未被占用的端口
        port: 8719
        client-ip: 172.100.20.220   #这里是我windows地址
      datasource:
        flow:
          nacos:
            server-addr: ${spring.cloud.nacos.discovery.server-addr}
            groupId: DEFAULT_GROUP
            dataId: ${spring.application.name}
            rule-type: flow
            data-type: json
#暴露自己的端点
management:
  endpoints:
    web:
      exposure:
        include: '*'
```
> 这个上面的`spring.cloud.sentinel.datasource`下面的配置，根据这个dataId，gourpId，在nacos中必须有对应的配置文件，不然会报错，说找不到对应的规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/f875b97808674ce9a9e67641a4d7a05a.png)
> 其他规则配置如下
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
        dashboard: 192.168.31.100:8859   #这里是我linux地址
        #客户端IP(sentinel dashboard进行实时监控的主机ip地址)
        # 默认端口8719端口，假如被占用会自动从8719开始依次+1扫描，直到找到未被占用的端口
        port: 8719
        client-ip: 172.100.20.220   #这里是我windows地址
      datasource:
      # 这里的flow名字随便取   流控规则
        flow:
          nacos:
            server-addr: ${spring.cloud.nacos.discovery.server-addr}
            groupId: DEFAULT_GROUP
            dataId: ${spring.application.name}
            rule-type: flow
            data-type: json
        # 授权规则    
        authority:
          nacos:
            server-addr: ${spring.cloud.nacos.config.server-addr}
            dataId: ${spring.application.name}-authority-rules
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: authority
          # 降级规则  
          degrade:
            nacos:
              server-addr: ${spring.cloud.nacos.config.server-addr}
              dataId: ${spring.application.name}-degrade-rules
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: degrade
          # 热点规则    
          param-flow:
            nacos:
              server-addr: ${spring.cloud.nacos.config.server-addr}
              dataId: ${spring.application.name}-param-flow-rules
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: param-flow
          # 系统规则    
          system:
            nacos:
              server-addr: ${spring.cloud.nacos.config.server-addr}
              dataId: ${spring.application.name}-system-rules
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: system
```
### 3.nacos添加配置

> 这里的dataId，还有Group要和上面的bootstrap.yml的内容一样

![在这里插入图片描述](https://img-blog.csdnimg.cn/9ff6a59e877e4183811ceb669f0d91f4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_17,color_FFFFFF,t_70,g_se,x_16#pic_center)



> json串

```json
 [{
    "resource": "hello",
    "limitApp": "default",
    "grade": 1,
    "count": 1,
    "strategy": 0,
    "controlBehavior": 0,
    "clusterMode": false
}]
```

> 参数解析

```json
resource:资源名称;
IimitApp:来源应用;
grade:國值类型,0表示线程数,1表示QPS;
count:单机阈值
strategy:流控模式,0表示直接,1表示关联,2表示链路
controlbehavior:流控效果,0表示快速失败,1表示 Warm Up,2表示排队等待;
cluster Mode:是否集群。
```

### 4.修改Controller

```java
@RestController
@Slf4j
public class SentinelController {
    @GetMapping("/hello")
    @SentinelResource(value = "hello",blockHandler = "helloBlockHandler")
    public String hello(){
        log.info("hello");
        return "hello";
    }
    public String helloBlockHandler(BlockException e){
        return "CustomerController invoke blockHandler";
    }
}
```

### 5.启动主程序

```
http://127.0.0.1:8005/hello
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/037f58f836274510882e44cf666fc74d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

### 6.测试

> 频繁访问后：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b6d6b5e5a3774b289cd442dcbd4d0004.png#pic_center)


## 四、注意点

Sentinel控制台修改规则：仅存在于服务的内存中，不会修改Nacos中配置值，重启后恢复原来的值 Nacos控制台修改规则：Nacos持久化规则，服务的内存也同步更新