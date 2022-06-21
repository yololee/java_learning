# SpringCloudAlibaba：@SentinelResource

## 一、按资源名称添加流控规则

### 1.新建controller

```java
@RestController
public class SentinelResourceController {
    @GetMapping("/resource")
    @SentinelResource(value = "resource",blockHandler = "handleException")
    public String resource(){
        return "SentinelResourceController invoke resource success";
    }
    
    public String handleException(BlockException e){
        return "SentinelResourceController invoke handleException";
    }
}
```

然后启动项目，在浏览器输入地址，然后在sentinel的控制台就可以看到了

![在这里插入图片描述](https://img-blog.csdnimg.cn/2c9a8d56834b423eaf11abbeabfc8f31.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 2.新建流控规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/a05ecafced3e4bf9b9a8ba52aff1446c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 3.测试

刷新频繁会出现

![在这里插入图片描述](https://img-blog.csdnimg.cn/09c8384c2d5944e99c213b44e6b417ee.png#pic_center)


## 二、自定义异常返回类

### 1.自定义handler

```java
public class ZrsBlockHandler {
    
    public static String handler1Exception(BlockException exception){
        return  "ZrsBlockHandler invoke handler【1】Exception";
    }
    public static String handler2Exception(BlockException exception){
        return  "ZrsBlockHandler invoke handler【2】Exception";
    }
}
```

### 2.修改SentinelResourceController

```java
@RestController
public class SentinelResourceController {
    @GetMapping("/resource")
    @SentinelResource(value = "resource",blockHandler = "handleException")
    public String resource(){
        return "SentinelResourceController invoke resource success";
    }
    public String handleException(BlockException e){
        return "SentinelResourceController invoke handleException";
    }
    @GetMapping("/handler1")
    @SentinelResource(value = "handler1Exception",blockHandlerClass = ZrsBlockHandler.class,
    blockHandler = "handler1Exception")
    public String handler1(){
        return "SentinelResourceController invoke resource success";
    }

    @GetMapping("/handler2")
    @SentinelResource(value = "handler2Exception",blockHandlerClass = ZrsBlockHandler.class,
            blockHandler = "handler2Exception")
    public String handler2(){
        return "SentinelResourceController invoke resource success";
    }
}
```

### 3.启动服务

```
http://localhost:8005/handler1
http://localhost:8005/handler2
```

### 4.添加流控规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/215540877c4042fb9a2209e531d624b5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/91b0959c724f458588c29d176a52be98.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 5.测试

频繁访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/1b1343de8d7b45e7b2acb7d1dc5b7b42.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/bad4a508556c4001a7bc3ec1b9ea5fac.png#pic_center)