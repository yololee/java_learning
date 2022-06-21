# SpringCloudAlibaba：nacos实现服务注册和发现

## 一、nacos安装

[docker安装单节点nacos](https://blog.csdn.net/weixin_43296313/article/details/115498821)
访问地址：192.168.31.100:8848/nacos

## 二、nacos入门

### 1.父工程

![在这里插入图片描述](https://img-blog.csdnimg.cn/045bbeb1ceaf47b0af6025f0eb9f5373.png#pic_center)


#### 1.pom文件

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

### 2.服务提供者

![在这里插入图片描述](https://img-blog.csdnimg.cn/996d02ebe60540459d71abe7317237af.png#pic_center)


#### 1.pom文件

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

#### 2.application.yml

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

#### 3.controller

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

#### 4.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }

}
```

#### 5.测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/0967266ff8af4571ab56bbbae45b7dbc.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/352f973da047432f982a55dede5d48a2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 3.再次创建服务提供者

这里跟上面的服务提供方一样，只需要改一下配置类中的端口

![在这里插入图片描述](https://img-blog.csdnimg.cn/011597b516d143c0b907a4d9598ffa72.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


这里实例数变成了俩个

![在这里插入图片描述](https://img-blog.csdnimg.cn/69d68217b57c4e5398c8067b39f01cb1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 4.创建消费者

#### 1.pom文件

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

#### 2.application.yml

```yml
server:
  port: 9003
spring:
  application:
    name: customer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.31.100:8848
```

#### 3.配置类

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

#### 4.controller

```java
@RestController
@RequestMapping("/customer")
public class CustomerController {
    @Autowired
    private RestTemplate restTemplate;

    private final String SERVER_URL="http://provider";

    @GetMapping(value = "/hello")
    public String hello(){
        return restTemplate.getForObject(SERVER_URL+"/provider/hello", String.class);
    }
}
```

#### 5.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }

}
```

#### 6.测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/281cb79ed4f1478abc44934ad358e242.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/f2f5c9ddb65a4db9aa12fa654490eefa.png#pic_center)


这里`@LoadBalanced`注解起作用了，实现了轮询

![在这里插入图片描述](https://img-blog.csdnimg.cn/f57bc3c17caa4e168d99f74e45249363.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)