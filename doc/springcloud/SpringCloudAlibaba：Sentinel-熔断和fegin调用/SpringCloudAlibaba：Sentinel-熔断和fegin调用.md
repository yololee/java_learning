# SpringCloudAlibaba：Sentinel-熔断和fegin调用

## 一、环境准备

### 父模块依赖

```xml
<properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
        <spring-cloud.version>Hoxton.SR9</spring-cloud.version>
        <spring-cloud-alibaba.version>2.1.0.RELEASE</spring-cloud-alibaba.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

### nacos-provider

#### pom文件

```xml
<dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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

> application.yml

```yml
server:
  port: 8001

spring:
  application:
    name: provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848

#暴露监控
management:
  endpoints:
    web:
      exposure:
        include: '*'

```

#### 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }

}
```

#### controller

```java
@RestController
@RequestMapping("/provider")
public class ProviderController {
    @Value("${server.port}")
    private Integer port;
    @GetMapping("/hello")
    public String hello(){
        return "Hello nacos , server port is "+port;
    }
}
```

#### 测试

http://localhost:8001/provider/hello

![在这里插入图片描述](https://img-blog.csdnimg.cn/564bae98c7874601ba64739961cc3ff7.png#pic_center)


### nacos-provider-salve

`nacos-provider-salve`模块跟`nacos-provider`模块一样就端口不一样

http://localhost:8002/provider/hello

![在这里插入图片描述](https://img-blog.csdnimg.cn/45d6db8efcad475eab3c1cf6d186ef29.png#pic_center)


### nacos-consumer

#### pom文件

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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

#### application.yml

```yml
server:
  port: 8003
spring:
  application:
    name: customer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848
    sentinel:
      transport:
        #配置sentinel地址,端口
        dashboard: 192.168.31.100:8859   #这里是我linux地址
        #客户端IP(sentinel dashboard进行实时监控的主机ip地址)
        # 默认端口8719端口假如被占用会自动从8719开始依次+1扫描，直到找到未被占用的端口
        port: 8719
        client-ip: 172.100.20.220   #这里是我windows地址
```

#### controller

```java
@RestController
@RequestMapping("/customer")
public class CustomerController {
    @Autowired
    private RestTemplate restTemplate;

    private final String SERVER_URL="http://provider";

    @GetMapping(value = "/hello")
    @SentinelResource(value = "hello")
    public String hello(){
        return restTemplate.getForObject(SERVER_URL+"/provider/hello", String.class);
    }
}
```

#### RestTemplateConfig

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

#### 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }

}
```

#### 测试

> 测试接口

http://localhost:8003/customer/hello

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d5c485be9cf429a9bdf6511726aa78a.png#pic_center)


> 查看nacos

![在这里插入图片描述](https://img-blog.csdnimg.cn/247c7b693a074a6c9e0cfd62c9036e22.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


> 查看sentinel

![在这里插入图片描述](https://img-blog.csdnimg.cn/3cf7e8ae22a046b6aad08a9812ef53f5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 二、fallback

### 1.修改CustomerController

```java
@RestController
@RequestMapping("/customer")
public class CustomerController {
    @Autowired
    private RestTemplate restTemplate;

    private final String SERVER_URL="http://provider";

    @GetMapping(value = "/hello/{id}")
    @SentinelResource(value = "hello",fallback = "fallbackHandler")
    public String hello(@PathVariable("id") String id){
        if (id.equals("1")){
            throw new RuntimeException("id不能为1");
        }
        return restTemplate.getForObject(SERVER_URL+"/provider/hello", String.class);
    }

    public String fallbackHandler(@PathVariable("id") String id ,Throwable e){
        return "CustomerController invoke fallbackHandler";
    }
}
```

### 2.测试

http://localhost:8003/customer/hello/1

![在这里插入图片描述](https://img-blog.csdnimg.cn/08c47123a56e469fb1253dbb7fe2c48e.png#pic_center)


http://localhost:8003/customer/hello/2

![在这里插入图片描述](https://img-blog.csdnimg.cn/b934920f55d94c4e890a2999c7f22eba.png#pic_center)


### ==3.注意==

fallback：是进行降级处理

## 三、blockHandler

### 1.修改CustomerController

```java
@RestController
@RequestMapping("/customer")
public class CustomerController {

    @Autowired
    private RestTemplate restTemplate;

    private final String SERVER_URL="http://provider";

    /**
     *  blockHandler：仅负责sentinel违规
     */
    @GetMapping(value = "/hello/{id}")
    @SentinelResource(value = "hello",blockHandler = "helloBlockHandler")
    public String hello(@PathVariable("id") String id){
        if (id.equals("1")){
            throw new RuntimeException("id不能为1");
        }
        return restTemplate.getForObject(SERVER_URL+"/provider/hello", String.class);
    }
    public String fallbackHandler(@PathVariable("id") String id ,Throwable e){
        return "CustomerController invoke fallbackHandler";
    }
    public String helloBlockHandler(String id, BlockException e){
        return "CustomerController invoke blockHandler";
    }
}
```

### 2.sentinel配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/321d03d46e804c5c9b9f0ed447037646.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/180dc9a67883410fa381d1f31518b284.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/50e0f543d4f04136ad068a56c9856721.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 3.测试

```java
#访问1时直接报错，java没有对应fallback处理
http://localhost:8003/customer/hello/1
#当频繁访问，出现限流响应
http://localhost:8003/customer/hello/2
```

### ==4.注意==

blockHandler：仅负责sentinel违规

## 四、fallback、blockHandler同时配置

### 1.修改controller

```java
@RestController
@RequestMapping("/customer")
public class CustomerController {

    @Autowired
    private RestTemplate restTemplate;

    private final String SERVER_URL="http://provider";

    /**
     *  blockHandler：仅负责sentinel违规
     */
    @GetMapping(value = "/hello/{id}")
    @SentinelResource(value = "hello",blockHandler = "helloBlockHandler",fallback = "fallbackHandler")
    public String hello(@PathVariable("id") String id){
        System.out.println(id);
        if (id.equals("1")){
            throw new RuntimeException("id不能为1");
        }
        return restTemplate.getForObject(SERVER_URL+"/provider/hello", String.class);
    }
    public String fallbackHandler(@PathVariable("id") String id ,Throwable e){
        return "CustomerController invoke fallbackHandler";
    }
    public String helloBlockHandler(String id,BlockException e){
        return "CustomerController invoke blockHandler";
    }
}
```

### 2.测试

```java
#java异常fallback->fallbackHandler
http://localhost:8003/customer/hello/1
#限流控制->helloBlockHandler
http://localhost:8003/customer/hello/2
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f426f07467a4a149fe9d97824e0d4e0.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/3cd543693fac4c0fa97e2e54a9ff78a7.png#pic_center)


## 五、OpenFegin

### 1.pom文件

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--sentinel nacos-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <!--sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--spring cloud alibaba-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
        </dependency>
    </dependencies>
```

### 2.application.yml

```yml
server:
  port: 9005
spring:
  application:
    name: customer-fegin
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848
    sentinel:
      transport:
        #配置sentinel地址,端口
        dashboard: 192.168.31.100:8859
        port: 8719
        #客户端IP
        client-ip: 172.100.20.220
#开启sentinel 对fegin的支持
feign:
  sentinel:
    enabled: true

```

### 3.启动类

```java
@EnableFeignClients
@SpringBootApplication
@EnableDiscoveryClient
public class OpenfeginApplication {

    public static void main(String[] args) {
        SpringApplication.run(OpenfeginApplication.class, args);
    }

}
```

### 4.降级类

```java
@Service
public class ProviderServiceFallback implements ProviderService {
    @Override
    public String hello() {
        return "ProviderServiceFallback invoke hello";
    }
}
```

### 5.Fegin服务类

```java
@FeignClient(value = "provider",path = "/provider",fallback = ProviderServiceFallback.class)
@Service
public interface ProviderService {
    @GetMapping("/hello")
    String hello();
}
```

### 6.controller

```
@RestController
@RequestMapping("/fegin")
public class FeginController {

    @Autowired
    private ProviderService providerService;

    @GetMapping("/hello")
    @SentinelResource(value = "hello")
    public String hello(){
        return providerService.hello();
    }
}
```

### 7.启动主程序

```java
#报错
com.alibaba.cloud.sentinel.feign.SentinelContractHolder.parseAndValidateMetadata(Ljava/lang/Class;)
#解决办法，修改父pomspringcloud版本为Hoxton.SR1。- -，BUG。期待后续解决 - -！！！，修改后重启解决
<spring-cloud.version>Hoxton.SR1</spring-cloud.version>
```

### 8.测试

http://localhost:9005/fegin/hello/

![在这里插入图片描述](https://img-blog.csdnimg.cn/99d0b30a1fce46a1a1b12d76b2de8cd9.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/335979e173ec4a19a992568f39d73884.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)