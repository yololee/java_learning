## 一、JWT 实现微服务鉴权

JWT一般用于实现单点登录。单点登录：如腾讯下的游戏有很多，包括lol，飞车等，在qq游戏对战平台上登录一次，然后这些不同的平台都可以直接登陆进去了，这就是单点登录的使用场景。JWT就是实现单点登录的一种技术，其他的还有oath2等。

### 1 什么是微服务鉴权

我们之前已经搭建过了网关，使用网关在网关系统中比较适合进行权限校验。

![在这里插入图片描述](https://img-blog.csdnimg.cn/6736bd46aae54af38cf088bb10629406.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


那么我们可以采用JWT的方式来实现鉴权校验。

### 2.代码实现

思路分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/7681ed9c5b5e487883879e463b2f4565.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_17,color_FFFFFF,t_70,g_se,x_16#pic_center)


```
1. 用户进入网关开始登陆，网关过滤器进行判断，如果是登录，则路由到后台管理微服务进行登录
2. 用户登录成功，后台管理微服务签发JWT TOKEN信息返回给用户
3. 用户再次进入网关开始访问，网关过滤器接收用户携带的TOKEN 
4. 网关过滤器解析TOKEN ，判断是否有权限，如果有，则放行，如果没有则返回未认证错误
```

#### 签发token

（1）创建类： JwtUtil

```java
package com.mye.nacosprovider.jwt;
 
import com.alibaba.fastjson.JSON;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;
 
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;


import java.util.*;

@Component
public class JwtUtil {
 
    //加密 解密时的密钥 用来生成key
    public static final String JWT_KEY = "IT1995";
    
    /**
     * 生成加密后的秘钥 secretKey
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }

 
    public static String createJWT(String id, String subject, long ttlMillis){
 
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256; //指定签名的时候使用的签名算法，也就是header那部分，jjwt已经将这部分内容封装好了。
        long nowMillis = System.currentTimeMillis();//生成JWT的时间
        Date now = new Date(nowMillis);
        SecretKey key = generalKey();//生成签名的时候使用的秘钥secret,这个方法本地封装了的，一般可以从本地配置文件中读取，切记这个秘钥不能外露哦。它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
        JwtBuilder builder = Jwts.builder() //这里其实就是new一个JwtBuilder，设置jwt的body
//                .setClaims(claims)            //如果有私有声明，一定要先设置这个自己创建的私有的声明，这个是给builder的claim赋值，一旦写在标准的声明赋值之后，就是覆盖了那些标准的声明的
                .setId(id)                    //设置jti(JWT ID)：是JWT的唯一标识，根据业务需要，这个可以设置为一个不重复的值，主要用来作为一次性token,从而回避重放攻击。
                .setIssuedAt(now)            //iat: jwt的签发时间
                .setSubject(subject)        //sub(Subject)：代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，可以存放什么userid，roldid之类的，作为什么用户的唯一标志。
                .signWith(signatureAlgorithm, key);//设置签名使用的签名算法和签名使用的秘钥
        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date exp = new Date(expMillis);
            builder.setExpiration(exp);        //设置过期时间
        }
        return builder.compact();            //就开始压缩为xxxxxxxxxxxxxx.xxxxxxxxxxxxxxx.xxxxxxxxxxxxx这样的jwt
    }
 
    public static Claims parseJWT(String jwt){

        SecretKey key = generalKey();  //签名秘钥，和生成的签名的秘钥一模一样
        Claims claims = Jwts.parser()  //得到DefaultJwtParser
                .setSigningKey(key)         //设置签名的秘钥
                .parseClaimsJws(jwt).getBody();//设置需要解析的jwt
        return claims;
    }
 
    public static void main(String[] args){
 
        Map<String, Object> user = new HashMap<>();
        user.put("username", "it1995");
        user.put("password", "123456");
        String jwt = createJWT(UUID.randomUUID().toString(), JSON.toJSONString(user), 3600 * 24);
 
        System.out.println("加密后：" + jwt);
 
        //解密
        Claims claims = parseJWT(jwt);
        System.out.println("解密后：" + claims.getSubject());
    }
}
```

(2)修改login方法,用户登录成功 则 签发TOKEN

```java
@PostMapping("/login")
    public String login(@RequestBody User user){
        //在redis中根据用户名查找密码
        String password = redisTemplate.opsForValue().get(user.getUsername());
        System.out.println(password);
        boolean checkResult = BCrypt.checkpw(user.getPassword(), password);
        if (checkResult){
            Map<String, String> info = new HashMap<>();
            info.put("username", user.getUsername());
            String token = JwtUtil.createJWT(UUID.randomUUID().toString(), user.getUsername(), 3600L*1000);
            info.put("token",token);
            return JSONUtil.toJsonStr(info);
        }else {
            return "登录失败";
        }
    }
```

(3) 测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c7b310f7f5f4afb8649b537d74cca3f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


#### 网关过滤器验证token

（1）网关模块添加依赖

```xml
<!--鉴权-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

（2）创建JWTUtil类

```java
package com.mye.nacosprovider.jwt;
 
import com.alibaba.fastjson.JSON;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;
 
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;


import java.util.*;

@Component
public class JwtUtil {
 
    //加密 解密时的密钥 用来生成key
    public static final String JWT_KEY = "IT1995";
    
    /**
     * 生成加密后的秘钥 secretKey
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }

 
    public static String createJWT(String id, String subject, long ttlMillis){
 
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256; //指定签名的时候使用的签名算法，也就是header那部分，jjwt已经将这部分内容封装好了。
        long nowMillis = System.currentTimeMillis();//生成JWT的时间
        Date now = new Date(nowMillis);
        SecretKey key = generalKey();//生成签名的时候使用的秘钥secret,这个方法本地封装了的，一般可以从本地配置文件中读取，切记这个秘钥不能外露哦。它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
        JwtBuilder builder = Jwts.builder() //这里其实就是new一个JwtBuilder，设置jwt的body
//                .setClaims(claims)            //如果有私有声明，一定要先设置这个自己创建的私有的声明，这个是给builder的claim赋值，一旦写在标准的声明赋值之后，就是覆盖了那些标准的声明的
                .setId(id)                    //设置jti(JWT ID)：是JWT的唯一标识，根据业务需要，这个可以设置为一个不重复的值，主要用来作为一次性token,从而回避重放攻击。
                .setIssuedAt(now)            //iat: jwt的签发时间
                .setSubject(subject)        //sub(Subject)：代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，可以存放什么userid，roldid之类的，作为什么用户的唯一标志。
                .signWith(signatureAlgorithm, key);//设置签名使用的签名算法和签名使用的秘钥
        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date exp = new Date(expMillis);
            builder.setExpiration(exp);        //设置过期时间
        }
        return builder.compact();            //就开始压缩为xxxxxxxxxxxxxx.xxxxxxxxxxxxxxx.xxxxxxxxxxxxx这样的jwt
    }
 
    public static Claims parseJWT(String jwt){

        SecretKey key = generalKey();  //签名秘钥，和生成的签名的秘钥一模一样
        Claims claims = Jwts.parser()  //得到DefaultJwtParser
                .setSigningKey(key)         //设置签名的秘钥
                .parseClaimsJws(jwt).getBody();//设置需要解析的jwt
        return claims;
    }
 
    public static void main(String[] args){
 
        Map<String, Object> user = new HashMap<>();
        user.put("username", "it1995");
        user.put("password", "123456");
        String jwt = createJWT(UUID.randomUUID().toString(), JSON.toJSONString(user), 3600 * 24);
 
        System.out.println("加密后：" + jwt);
 
        //解密
        Claims claims = parseJWT(jwt);
        System.out.println("解密后：" + claims.getSubject());
    }
}
```

（3）创建过滤器，用于token验证

```java
/**
 * 鉴权过滤器 验证token
 */
@Component
public class AuthorizeFilter implements GlobalFilter, Ordered {
    private static final String AUTHORIZE_TOKEN = "token";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		//1. 获取请求
        ServerHttpRequest request = exchange.getRequest();
        //2. 则获取响应
        ServerHttpResponse response = exchange.getResponse();
        //3. 如果是登录请求则放行
        if (request.getURI().getPath().contains("/admin/login")) {
            return chain.filter(exchange);
        }
        //4. 获取请求头
        HttpHeaders headers = request.getHeaders();
        //5. 请求头中获取令牌
        String token = headers.getFirst(AUTHORIZE_TOKEN);

        //6. 判断请求头中是否有令牌
        if (StringUtils.isEmpty(token)) {
            //7. 响应中放入返回的状态吗, 没有权限访问
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            //8. 返回
            return response.setComplete();
        }

        //9. 如果请求头中有令牌则解析令牌
        try {
            JwtUtil.parseJWT(token);
        } catch (Exception e) {
            e.printStackTrace();
            //10. 解析jwt令牌出错, 说明令牌过期或者伪造等不合法情况出现
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            //11. 返回
            return response.setComplete();
        }
        //12. 放行
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

（4）测试：

> 首先进行登录测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a2d4955b237493c8f2cf15fd31c35e4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


> 在进行鉴权测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/5c35df64504447baae89c252677d3836.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)