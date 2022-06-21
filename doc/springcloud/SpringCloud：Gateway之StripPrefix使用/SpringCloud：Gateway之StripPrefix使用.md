## StripPrefix 过滤配置

很多时候也会有这么一种请求，用户请求路径是`/api/goods`,而真实路径是`/goods`，这时候我们需要去掉`/api`才是真实路径，此时可以使用StripPrefix功能来实现路径的过滤操作，如下配置：

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
          uri: lb://provider
          predicates:
            - Path=/api/goods/**
          #filters:
            #- StripPrefix=1  
      default-filters:
        - StripPrefix=1
```

>  测试

```json
请求地址：http://127.0.0.1:80/api/goods/findOne/1/zhangsan
返回结果：1=====zhangsan

- StripPrefix=1的意思是去掉上面路径的api，也就是第一个前缀，也可以设置为- StripPrefix=2，以此类推
```