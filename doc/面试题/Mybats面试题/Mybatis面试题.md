# Mybatis面试题

### MyBatis是什么

```java
MyBatis 是一款优秀的持久层框架，一个半 ORM（对象关系映射）框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
```

### Mybatis优缺点

```java
优点：
    与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接
    基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响
    很好的与各种数据库兼容
    提供映射标签，支持对象与数据库的字段映射；提供对象关系映射标签，支持对象关系组件维护
    能够与Spring很好的集成
缺点：
    SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求
    SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库
```

### MyBatis的工作原理

```java
1、读取 MyBatis 核心配置文件(mybatis-config.xml)
    
2、加载映射文件mapper.xml(映射文件即 SQL 映射文件)
    
3、构造会话工厂,通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory  
    
4、创建会话对象,由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法
    
5、Executor 执行器，mMyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。
    
6、MappedStatement 对象，在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息
    
7、输入参数映射，输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型，输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程
    
8、输出结果映射，输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程    
```

### 为什么需要预编译

```java
sql预编译指的是数据库驱动在发送sql语句和参数给数据库之前对sql语句进行编译，这样数据库执行sql时就不需要重新编译
JDBC 中使用对象 PreparedStatement 来抽象预编译语句，使用预编译。预编译阶段可以优化 SQL 的执行。预编译之后的 SQL 多数情况下可以直接执行，数据库 不需要再次编译，Mybatis在默认情况下，将对所有的 SQL 进行预编译    
```

### #{}和${}的区别

```java
#{}是占位符，预编译处理，可以防止SQL注入
${}是拼接符，字符串替换，没有预编译处理，不能防止SQL注入
    
Mybatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL中的#{}替换为?号，调用PreparedStatement的set方法来赋值；Mybatis在处理${}时，是原值传入，就是把${}替换成变量的值，相当于JDBC中的Statement编译
    
#{} 的变量替换后，#{} 对应的变量自动加上单引号
${} 的变量替换后，${} 对应的变量不会加上单引号    
```

### 如何获取生成的主键

```xml
<!-- useGeneratedKeys 设置为"true"表明 MyBatis 要获取由数据库自动生成的主键，keyColumn指定数据库主键，keyProperty指定 Java 实体类中对应的主键字段 -->
<insert id="insertUser" useGeneratedKeys="true" keyProperty="userId" >
  insert into user(
  user_name, user_password, create_time)
  values(#{userName}, #{userPassword} , #{createTime, jdbcType= TIMESTAMP})
</insert>
```

### 当实体类中的属性名和表中的字段名不一样 ，怎么办

1.通过在查询的SQL语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

```xml
<select id="getOrder" parameterType="int" resultType="com.jourwon.pojo.Order">
      select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
</select>
```

2.通过`<resultMap>`来映射字段名和实体类属性名的一一对应关系

```xml
<select id="getOrder" parameterType="int" resultMap="orderResultMap">
select * from orders where order_id=#{id}
</select>

<resultMap type="com.jourwon.pojo.Order" id="orderResultMap">
  <!–用id属性来映射主键字段–>
   <id property="id" column="order_id">

  <!–用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性–>
   <result property ="orderno" column ="order_no"/>
   <result property="price" column="order_price" />
</reslutMap>
```

### 使用MyBatis的mapper接口调用时有哪些要求

```java
1、Mapper.xml文件中的namespace即是mapper接口的全限定类名
2、Mapper接口方法名和mapper.xml中定义的sql语句id一一对应
3、Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql语句的parameterType的类型相同   4、Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql语句的resultType的类型相同 
```

### MyBatis动态sql是做什么的？都有哪些动态sql

```java
Mybatis动态sql可以让我们在xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能，Mybatis提供了9种动态sql标签
    
trim|where|set|foreach|if|choose|when|otherwise|bind。
```

### MyBatis是如何进行分页的？分页插件的原理是什么？

```java
Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页
    
分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，通过jdk动态代理在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和参数
    
举例：select * from student，拦截sql后重写为：select t.* from (select * from student) t limit 0, 10    
```

### MyBatis的一级、二级缓存

```java
一级缓存:基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，MyBatis默认打开一级缓存

二级缓存:二级缓存与一级缓存机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同之处在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态)，可在它的映射文件中配置<cache/> 标签
    
对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)进行了C/U/D 操作后，默认该作用域下所有缓存将被清理掉    
```

### 什么情况下用注解绑定,什么情况下用 xml 绑定

```java
当 Sql 语句比较简单时候,用注解绑定；当 SQL 语句比较复杂时候,用 xml 绑定,一般用xml 绑定的比较多
```

### Mybatis中的Xml映射文件中，不同的xml映射文件，id是否可以重复

```java
不同的 Xml 映射文件，如果配置了 namespace，那么 id 可以重复；如果没有配置 namespace，那么 id 不能重复；原因就是 namespace+id 是作为 Map<String, MapperStatement>的 key使用的，如果没有 namespace，就剩下 id，那么，id 重复会导致数据互相覆盖。有了 namespace，自然 id 就可以重复，namespace 不同，namespace+id 自然也就不同。
```

