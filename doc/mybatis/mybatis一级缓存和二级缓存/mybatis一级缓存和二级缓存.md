# mybatis中的一级缓存和二级缓存

## 一级缓存

一级缓存是sqlSession级别的缓存，在操作数据库的时候需要构造sqlSession对象，在对象中有一个数据结构用来存储缓存数据。不同的sqlSession之间的缓存数据区域是互相不影响的，也就是说他只能作用在用一个SqlSession中，不同的sqlSession中的缓存是不能互相读取的。



> 一级缓存的工作原理：

用户发起查询请求，查找某条数据，sqlSession先去缓存中查找，是否有该数据，如果有，读取；如果没有，从数据库中查询，并将查询到的数据放入一级缓存区域，供下次查找使用。

但sqlSession执行commit，即增删改操作时会清空缓存。这么做的目的是避免脏读。

如果commit不清空缓存，会有以下场景：A查询了某商品库存为10件，并将10件库存的数据存入缓存中，之后被客户买走了10件，数据被delete了，但是下次查询这件商品时，并不从数据库中查询，而是从缓存中查询，就会出现错误。

既然有了一级缓存，那么为什么要提供二级缓存呢？

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。二级缓存的作用范围更大。

还有一个原因，实际开发中，MyBatis通常和Spring进行整合开发。Spring将事务放到Service中管理，对于每一个service中的sqlsession是不同的，这是通过mybatis-spring中的org.mybatis.spring.mapper.MapperScannerConfigurer创建sqlsession自动注入到service中的。 每次查询之后都要进行关闭sqlSession，关闭之后数据被清空。所以spring整合之后，如果没有事务，一级缓存是没有意义的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a1b4dda85aa7499295e8bc57704971b4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


> 一级缓存的生命周期有多长？

- MyBatis在开启一个数据库会话时，会 创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象。Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。
- 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用。
- 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用。
- SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用

## 二级缓存

> 二级缓存原理

二级缓存是mapper级别的缓存，多个sqlSession去操作同一个mapper的sql语句，多个sqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

UserMapper有一个二级缓存区域(按namespace分)，其它mapper也有自己的缓存区域(按namespace分)。每一个namespace的maooer都有一个二级缓存区域，俩个mapper的namespace如果相同，这俩个mapper执行sql查询到数据将存在相同的二级缓存区域中

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe37a8db36724df4bf7e9afe052bd064.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


SqlSessionFactory层面上的二级缓存默认是不开启的，二级缓存的开席需要进行配置，实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的。 也就是要求实现Serializable接口，配置方法很简单，只需要在映射XML文件配置就可以开启缓存了<cache/>，如果我们配置了二级缓存就意味着：

- 映射语句文件中的所有select语句将会被缓存。
- 映射语句文件中的所欲insert、update和delete语句会刷新缓存。
- 缓存会使用默认的Least Recently Used（LRU，最近最少使用的）算法来收回。
- 根据时间表，比如No Flush Interval,（CNFI没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用
- 缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改。

## 开启二级缓存

### 1.打开总开关

在mybatis的配置文件中加入

```xml
<span style="font-size:18px;"><settings>    
   <!--开启二级缓存-->    
    <setting name="cacheEnabled" value="true"/>    
</settings> </span>
```

### 2.在需要开启二级缓存的mapper.xml中加入caceh标签

```xml
<cache eviction="LRU" flushInterval="100000" readOnly="true" size="1024"/>
```

> 参数解析

```
<!--
        eviction:代表的是缓存回收策略，目前MyBatis提供以下策略。
        (1) LRU,最近最少使用的，一处最长时间不用的对象
        (2) FIFO,先进先出，按对象进入缓存的顺序来移除他们
        (3) SOFT,软引用，移除基于垃圾回收器状态和软引用规则的对象
        (4) WEAK,弱引用，更积极的移除基于垃圾收集器状态和弱引用规则的对象。这里采用的是LRU，
                移除最长时间不用的对形象
        flushInterval:刷新间隔时间，单位为毫秒，这里配置的是100秒刷新，如果你不配置它，那么当
        SQL被执行的时候才会去刷新缓存。
        size:引用数目，一个正整数，代表缓存最多可以存储多少个对象，不宜设置过大。设置过大会导致内存溢出。
        这里配置的是1024个对象
        readOnly:只读，意味着缓存数据只能读取而不能修改，这样设置的好处是我们可以快速读取缓存，缺点是我们没有
        办法修改缓存，他的默认值是false，不允许我们修改
    -->
```

### 3，让使用二级缓存的POJO类实现Serializable接口

```java
public class Student implements Serializable {
    private static final long serialVersionUID = 735655488285535299L;
    private Integer id;
    private String name;
    private String age;
}
```

## 总结

对于查询多commit少且用户对查询结果实时性要求不高，此时采用mybatis二级缓存技术降低数据库访问量，提高访问速度

但不能滥用二级缓存，二级缓存也有很多弊端，从MyBatis默认二级缓存是关闭的就可以看出来。二级缓存是建立在同一个namespace下的，如果对表的操作查询可能有多个namespace，那么得到的数据就是错误的。