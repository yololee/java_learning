## ==Mybatis快速入门==

### 相关概念

```java
1、框架：是一个提供给开发人员使用的半成品软件，加上我们自己的代码就可以达到很好的效果，大大提高开发效率。
    
2、ORM：对象关系映射，将表以及表中的数据映射成类和对象。
  		表--->类
  		表中的字段--->类中的属性
  		表中的数据--->类的对象
  
  mybatis, hibernate 这样的框架通过指定表与java类的对应关系,就可以实现将数据封装到java 类的对象中去,这就称之为对象关系映射,这样的框架就称之为ORM 框架.
  
3、mybatis框架：
    1、是一个持久层框架，封装了原始的jdbc操作。
    2、mybatis支持xml和注解两种方式配置sql。
    3、mybatis支持ORM思想封装执行sql的结果。
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103215849979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI5NjMxMw==,size_16,color_FFFFFF,t_70#pic_center)


### 快速入门步骤

```java
1、准备工作(环境搭建：创建module、创建数据库和表、导入jar包、创建实体类)
2、在src下创建并编写mybatis映射配置文件
3、在src下创建并编写mybatis核心配置文件
4、创建测试类，使用mybatis提供的API操作数据
```

#### 准备工作

要做的事：创建module(模块)、创建数据库和表、导入jar包、创建实体类

```sql
CREATE DATABASE if not exists db1;
USE db1;
CREATE TABLE student(
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20),
  age INT
);
INSERT INTO student VALUES (NULL,'张三',23);
INSERT INTO student VALUES (NULL,'李四',24);
INSERT INTO student VALUES (NULL,'王五',25);
SELECT * FROM student;
```

```java
package com.itheima.domain;

public class Student {
  private Integer id;
  private String name;
  private Integer age;
	//构造方法、getter/setter方法、toString方法自己添加...
}
```

#### 创建并编写mybatis ==映射配置文件==

要做的事：主要配置CRUD的sql语句 ,  文件的名字, 建议 文件名叫做  ==StudentMapper.xml== 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
        namespace="StudentMapper":名称空间，名称空间+id值唯一确定一条sql语句，名称空间的作用类似于包名。目前来说值可以任意写，一般使用文件名。
        id="selectAll"：sql语句的唯一标志，后期要用
        resultType="com.itheima.bean.Student"：查询结果要封装的对象类型
    -->
<mapper namespace="StudentMapper">
    <select id="selectAll" resultType="com.itheima.bean.Student">
        select * from student
    </select>
</mapper>
```

#### 创建并编写mybatis==核心配置文件==

要做的事：==名字一般叫 MybatisConfig.xml==**，**配置数据库连接参数和加载映射配置文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--1、mysql连接参数-->
    <!--default="mysql" 默认使用id="mysql"连接环境-->
    <environments default="mysql">
        <!--id="mysql" 该环境的唯一标识id-->
        <environment id="mysql">
            <!--type="jdbc"  使用jdbc事务-->
            <transactionManager type="jdbc"></transactionManager>
            <!--配置数据源，mybatis内置连接池-->
            <dataSource type="pooled">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/db1"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!--2、加载映射文件-->
    <mappers>
        <mapper resource="StudentMapper.xml"/>
    </mappers>
</configuration>
```

#### 创建测试类，使用mybatis提供的API操作数据

```java
//1 加载核心配置文件
//2 获取SqlSessionFactory工厂对象
//3 通过SqlSessionFactory工厂对象获取SqlSession核心对象
//4 执行操作获取结果
//5 打印结果
//6 释放资源

@Test
public void testSelectAll() throws IOException {
  //读取资源--->招建设者--->造工厂--->生产核心对象
  //1 加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2 获取SqlSessionFactory工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3 通过SqlSessionFactory工厂对象获取SqlSession核心对象
  SqlSession sqlSession = factory.openSession();
  //4 执行操作获取结果
  List<Student> list = sqlSession.selectList("StudentMapper.selectAll");
  //5 打印结果
  for(Student student : list){
      System.out.println(student)
  }
  //6 释放资源
  sqlSession.close();
}
```

#### 总结(重点)

```java
1、创建并编写映射配置文件：StudentMapper.xml
   主要配置要执行的sql语句，名称空间+id唯一确定一条sql语句。
2、创建并编写核心配置文件： MybatisConfig.xml
   主要配置连接数据库的相关参数和加载映射配置文件
3、测试类
   读取资源(Resources)-->创建建设者(SqlSessionFactoryBuilder)造工厂(SqlSessionFactory)-->工厂生产核心对象(SqlSession)-->执行CRUD操作-->处理结果-->释放资源
```
## MyBatis的相关api

### Resources类

作用：加载核心配置文件

| 返回值      | 方法名                               | 说明                                 |
| ----------- | ------------------------------------ | ------------------------------------ |
| InputStream | getResourceAsStream(String fileName) | 通过类加载器返回指定资源的字节输入流 |

**==注意：文件必须放在src目录中，如果放到包里面了,那么就必须写“包名/包名/.../MybatisConfig.xml”==**

### SqlSessionFactoryBuilder类：

作用：用来创建SqlSessionFactory工厂对象

| 返回值            | 方法名                | 说明                                         |
| ----------------- | --------------------- | -------------------------------------------- |
| SqlSessionFactory | build(InputStream is) | 通过指定资源字节输入流获取SqlSession工厂对象 |

### SqlSessionFactory类

作用：用来获取SqlSession核心对象

| 返回值     | 方法名                          | 说明                                                         |
| ---------- | ------------------------------- | ------------------------------------------------------------ |
| SqlSession | openSession()                   | 获取SqlSession构造者对象，并开启手动提交事务                 |
| SqlSession | openSession(boolean autoCommit) | 获取SqlSession构造者对象，如果参数为true，则开启自动提交事务 |

### SqlSession类

作用：执行CRUD操作、事务管理，接口代理(明天讲，开发当中使用的方式)

| 返回值  | 方法名                                       | 说明                           |
| ------- | -------------------------------------------- | ------------------------------ |
| List<E> | selectList(String statement,Object paramter) | 执行查询语句，返回List集合     |
| T       | selectOnet(String statement,Object paramter) | 执行查询语句，返回一个结果对象 |
| int     | insert(String statement,Object paramter)     | 执行新增语句，返回影响行数     |
| int     | update(String statement,Object paramter)     | 执行修改语句，返回影响行数     |
| int     | delete(String statement,Object paramter)     | 执行删除语句，返回影响行数     |
| void    | commit()                                     | 提交事务                       |
| void    | rollback()                                   | 回滚事务                       |
| T       | getMapper(Class<T> cls)                      | 获取指定接口的代理实现类对象   |
| void    | close()                                      | 释放资源                       |

## ==MyBatis 映射配置文件==

### 1. 根据id查询学生信息

#### 第一步：在映射配置文件中配置sql语句

```xml
<!--根据id查询学生信息
		select:查询功能的标签
		id属性：唯一的标识
		resultType属性：指定结果映射对象类型
        parameterType属性：指定参数映射的类型
-->
<select id="selectById" resultType="com.itheima.bean.Student" parameterType="int">
  <!--#{id}大括号中的id表示占位符，可以任意写，#{id}在后期执行过程中会被翻译成?-->
  select * from student where id=#{id}
</select>
```

#### 第二步：在测试类中编写代码测试

```java
@Test
public void testSelectById() throws IOException {
  //1 加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2 获取SqlSessionFactory工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3 获取SqlSession核心对象
  SqlSession sqlSession = factory.openSession();
  //4 执行操作获取结果
  Student stu = sqlSession.selectOne("StudentMapper.selectById",2);
  //5 打印结果
  System.out.println("stu = " + stu);
  //6 释放资源
  sqlSession.close();
}
```

### 2 .添加学生信息

#### 第一步：在映射配置文件中配置sql语句

```xml
<!--添加学生信息-->
<insert id="insert" parameterType="com.itheima.bean.Student">
  <!--#{name},#{age}占位符的名称必须是student中对应的属性名称，不能随便书写-->
  insert into student values(null,#{name},#{age})
</insert>
```

#### 第二步：在测试类中编写代码测试

```java
@Test
public void testInsert() throws IOException {
  //1 加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2 创建建设中获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3 通过工厂对象生产SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4 执行添加操作
  Student stu=new Student(null,"刘枫",40);
  int count = sqlSession.insert("StudentMapper.insert", stu);
  //5 打印结果：返回影响的行数
  System.out.println("count = " + count);
  //手动提交事务
  //sqlSession.commit();
  //6 释放资源
  sqlSession.close();
}
```

### 3. 修改学生信息

#### 第一步：在映射配置文件中配置sql语句

```xml
<!--修改学生信息-->
<update id="update" parameterType="com.itheima.bean.Student">
  update student set name=#{name},age=#{age} where id=#{id}
</update>
```

#### 第二步：在测试类中编写代码测试

```java
@Test
public void testUpdate() throws IOException {
  //1 加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2 创建建设中获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3 通过工厂对象生产SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4 执行添加操作
  Student stu=new Student(5,"刘枫",42);
  int count = sqlSession.update("StudentMapper.update", stu);
  //5 打印结果：返回影响的行数
  System.out.println("count = " + count);
  //手动提交事务
  //sqlSession.commit();
  //6 释放资源
  sqlSession.close();
}
```

### 4 删除学生信息

#### 第一步：在映射配置文件中配置sql语句

```xml
<!--删除学生信息-->
<delete id="delete" parameterType="int">
  delete from student where id=#{id}
</delete>
```

#### 第二步：在测试类中编写代码测试

```java
@Test
public void testDelete() throws IOException {
  //1 加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2 创建建设中获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3 通过工厂对象生产SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4 执行添加操作
  int count = sqlSession.delete("StudentMapper.delete", 5);
  //5 打印结果：返回影响的行数
  System.out.println("count = " + count);
  //手动提交事务
  //sqlSession.commit();
  //6 释放资源
  sqlSession.close();
}
```

## ==Mybatis核心配置文件介绍==

### 1. 事务管理器type="jdbc"取值(了解)

在 MyBatis 中有两种类型的事务管理器（也就是 type=”[JDBC|MANAGED]”）：

- **JDBC** – 这个配置直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务。
- **MANAGED** – 这个配置几乎没做什么。它从来不提交或回滚一个连接。也就是说不使用事务管理。

### 2. 数据源dataSource的type="pooled"取值(了解)

有三种内建的数据源类型（也就是 type=”[UNPOOLED|POOLED|JNDI]”）：

- **UNPOOLED**– 这个数据源的实现只是每次被请求时打开和关闭连接，也就是说不使用连接池。
- **OOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。
- **JNDI**– 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源。

### ==3. 加载外部properties属性文件(重点)==

#### 第一步：创建jdbc.properties属性文件

```properties
driverClass=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/db1
username=root
password=root
```

#### 第二步：在mybatisConfig.xml中引入属性文件

```xml
<!--引入属性文件-->
<properties resource="jdbc.properties"/>
```

#### 第三步：使用${key}性值

```xml
<!--配置数据源，mybatis内置连接池-->
<dataSource type="pooled">
  <property name="driver" value="${driverClass}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

### ==4 .配置别名==

```xml
<!--配置别名-->
<typeAliases>
  <!--给单个类配置别名 alias="student"表示别名，不写的话类名就是别名，不区分大小写，比较少用-->
  <typeAlias type="com.itheima.bean.Student" alias="student" />
  <!--给一个包中的类配置别名，类名就是别名，不区分大小写，常用-->
  <!--<package name="com.itheima.bean"/>-->
</typeAliases>
```

```java
//@Alias("stu"); //前提是先使用了<typeAliases>配置了别名，如果某个bean不想使用类名作为别名，那么就可以使用@Alias单独设置别名
public class Student {
    private Integer id;
    private String name;
    private Integer age;
  	//...
}
```

### 5.加载映射配置

```xml
<mapper resource="com/itheima/mapper/UserMapping.xml"></mapper>
```

### 6.mybatis集成log4j日志

#### 第一步：导入log4j-1.2.17.jar包

#### 第二步：将log4j.properties配置文件复制到src中

#### 第三步：在==核心配置文件==中使用settings标签配置日志对象。

```xml
<settings>
  <!--配置日志管理信息-->
  <setting name="logImpl" value="LOG4J"/>
</settings>
```

### 7.MyBatis核心配置文件层级关系

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103215946215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI5NjMxMw==,size_16,color_FFFFFF,t_70#pic_center)