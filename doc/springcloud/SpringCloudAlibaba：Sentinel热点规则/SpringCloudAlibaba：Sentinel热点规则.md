# SpringCloudAlibaba：Sentinel热点规则

## 一、介绍

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c05e3608d1964199bc84949c2db406d0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 二、代码测试

### 1.新建Controller

```java
@RestController
public class HotkeyController {
    /**
     * SentinelResource: sentinel相关配置，value唯一，blockHandler回调方法
     */
    @GetMapping("/hotkey")
    @SentinelResource(value = "hotkey",blockHandler = "dealHotkey")
    public String hotkey(@RequestParam(value = "key",required = false) String key){
        return "hello hotkey,  O(∩_∩)O";
    }
    public String dealHotkey(String key, BlockException e){
        return "调用失败,   o(╥﹏╥)o";
    }
}
```

然后启动项目，访问一下接口出现，sentinel的控制台

### 2.新建热点规则

解释：资源名为@SentinelResource的value值，阈值是qp的值，统计时长是发生后1秒内降级。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a19d0bcaa389463f8e924eb737a967b1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


这里的参数索引是对应的是第几个参数，例如`http://localhost:8005/hotkey?key=sentinel`这里的key就是第0个参数。

这里的单机阈值0的意思是只要一访问，就直接触发降级业务。如果这里单机阈值设置为3，这快速点击四次后后才触发降级业务

### 3.测试

调用：http://localhost:8005/hotkey

![在这里插入图片描述](https://img-blog.csdnimg.cn/783c7142442a47a1838d5373b98699f2.png#pic_center)


调用http://localhost:8005/hotkey?key=sentinel频繁时：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2e4e6032cd8e43f994d3bea6961e1e0d.png#pic_center)


### 4.参数例外项

期望当key为phone时qps为100，剩下的为1

![在这里插入图片描述](https://img-blog.csdnimg.cn/412788ba3ef84f7392bfa80ab6cbd802.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)


狂点后仍然返回正确的行为：

http://localhost:8005/hotkey?key=phone

![在这里插入图片描述](https://img-blog.csdnimg.cn/d5811c9b708442afb89ceb2364093f5d.png#pic_center)