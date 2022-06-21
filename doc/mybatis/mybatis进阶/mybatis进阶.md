## ==接口代理方式实现Dao==

### 1 接口代理的要求【记住】

```java
1、映射配置文件中的名称空间要和接口的全类名相同
2、映射配置文件中标签的id属性值要和接口的方法名相同     形象比喻为namespace+id=接口全类名+方法名
3、映射配置文件中标签的parameterType属性值要和方法的参数类型一致
4、映射配置文件中标签的resultType属性值要和方法的返回值类型一致
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210104215643628.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI5NjMxMw==,size_16,color_FFFFFF,t_70#pic_center)


### 2 代码演示

#### 第一步：准备接口

```java
package com.itheima.mapper;

/*
    持久层接口
 */
public interface StudentMapper {
    //查询全部
    public abstract List<Student> selectAll();
    //根据id查询
    public abstract Student selectById(Integer id);
    //新增数据
    public abstract Integer insert(Student stu);
    //修改数据
    public abstract Integer update(Student stu);
    //删除数据
    public abstract Integer delete(Integer id);
    //多条件查询
    public abstract List<Student> selectCondition(Student stu);
    //根据多个id查询
    public abstract List<Student> selectByIds(List<Integer> ids);
}
```

#### 第二步：编写/修改映射文件

```xml
<!--namespace="com.itheima.mapper.StudentMapper" :接口的全类名-->

<mapper namespace="com.itheima.mapper.StudentMapper">
  
  
  <!--id必须是接口的方法名-->
  <!--查询所有学生信息-->
  <select id="selectAll" resultType="student">
    select * from student
  </select>

  <!--根据id查询学生信息-->
  <select id="selectById" parameterType="int" resultType="student">
    select * from student where id=#{ID}
  </select>

  <!--添加学生-->
  <insert id="insert" parameterType="student">
    insert into student values(null,#{name},#{age})
  </insert>

  <!--修改学生信息-->
  <update id="update" parameterType="student">
    update student set name=#{name},age=#{age} where id=#{id}
  </update>

  <!--删除学生信息-->
  <delete id="delete" parameterType="int">
    delete from student where id=#{id}
  </delete>
</mapper>
```

#### 第三步：编程测试类测试

```java
public class StudentTest {

  @Test
  public void testSelectAll() throws IOException {
    //1、加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //2、获取工厂对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //3、获取SqlSession核心对象
    SqlSession sqlSession = factory.openSession(true);
    
    //4、获取接口的实现类代理对象，执行操作
    //返回了一个代理对象, 这个 代理对象 是通过 jdk的 动态代理来实现的 
    // Proxy.newInstance();
    
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    
    
    List<Student> list = mapper.selectAll();
    
    
    //5、处理/打印结果
    list.forEach(stu-> System.out.println(stu));
    //6、释放资源
    sqlSession.close();
  }


  @Test
  public void testSelectById() throws IOException {
    //1、加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //2、获取工厂对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //3、获取SqlSession核心对象
    SqlSession sqlSession = factory.openSession(true);
    
    //4、获取接口的实现类代理对象，执行操作
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    Student student = mapper.selectById(3);
    //5、处理/打印结果
    System.out.println(student);
    //6、释放资源
    sqlSession.close();
  }

  @Test
  public void testSelectDelete() throws IOException {
    //1、加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //2、获取工厂对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //3、获取SqlSession核心对象
    SqlSession sqlSession = factory.openSession(true);
    //4、获取接口的实现类代理对象，执行操作
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    Integer count = mapper.delete(8);
    //5、处理/打印结果
    System.out.println("count = " + count);//返回影响的行数
    //6、释放资源
    sqlSession.close();
  }
}
```

### 3.@Param注解

```xml
给参数取名字
public abstract List<Student> selectByNameOrAge(@Param("p1")String name,@Param("p2")Integer age)
    
<select id="selectByNameOrAge">
  select * from student name=#{p1} or age=#{p2}
</select>
```



## 动态SQL(理解)

### 1.if 标签判断(重要)

#### 第一步：配置SQL

```xml
<!--多条件查询-->
<select id="selectCondition" resultType="student" parameterType="student">
  select * from student
  <where>
    <!--test="" 是判断条件-->
    <if test="id != null">
      id=#{id}
    </if>
    <if test="name != null and name !='' ">
      and name=#{name}
    </if>
    <if test="age != null">
      and age=#{age}
    </if>
  </where>
</select>
```

#### 第二步：测试

```java
@Test
public void testSelectCondition() throws IOException {
  //1、加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2、获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3、获取SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4、获取接口的实现类代理对象，执行操作
  StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
  //创建条件
  Student stu=new Student();
  //stu.setId(2);
  stu.setAge(25);
  //执行查询
  List<Student> list = mapper.selectCondition(stu);
  
  //5、处理/打印结果
  list.forEach(student -> System.out.println(student));
  //6、释放资源
  sqlSession.close();
}
```

### 2.foreach标签遍历(了解)

#### 第一步：配置SQL

```xml
<!--多条件查询-->
<select id="selectByIds" resultType="student">
  select * from student
  <where>
    <!--
                collection="" 遍历数组写array，遍历集合写list，遍历map集合写map
                open="id in(" 以xxx开头
                close=")" 以xxx结尾
                item="" 遍历的每一个元素的名称,变量名随便写，写啥后面就用啥
                separator="，" 分隔符
                index="" 如果遍历的是数组或者单列集合，就表示索引。如果遍历的是map就表示key
            -->
    <foreach collection="list" open="id in(" close=")" item="id" separator=",">
      <!--#{id} 大括号中的名称就是item的属性值-->
      #{id}
    </foreach>
  </where>
</select>

select * from student where id in(1,2,3,4)
```

#### 第二步：测试

```java
@Test
public void testSelectByIds() throws IOException {
  //1、加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2、获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3、获取SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4、获取接口的实现类代理对象，执行操作
  StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
	
  //创建条件
  List<Integer> ids = List.of(1, 2,3);
  //执行查询
  List<Student> list = mapper.selectByIds(ids);
  //5、处理/打印结果
  list.forEach(student -> System.out.println(student));
  //6、释放资源
  sqlSession.close();
}
```

> **批量插入,批量删除**

```xml
<!--批量删除-->
<delete id="deleteMany" parameterType="int[]">
        delete from student where id  in
        <foreach collection="array" item="id" open="(" close=")" separator=",">
              #{id}
        </foreach>
</delete>
<!--批量插入-->
<insert id="insertMany" parameterType="list">
        insert into student values
        <foreach collection="list" item="list" separator=",">
            (#{list.id},#{list.name},#{list.age})
        </foreach>
</insert>
```

### 	3.SQL片段抽取

```java
- 我们可以将一些重复性的sql语句进行抽取，以达到复用的效果
- <sql>：抽取SQL语句标签
    <sql id="片段的唯一标识">抽取的SQL语句</sql>
- <include>：引入SQL片段标签
    <include refid="片段唯一标识"/>
```



## ==分页插件(重要)==

```java
1、导入jar包
2、在核心配置文件中配置分页插件
  <!--集成PageHelper分页插件-->
  <plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
  </plugins>

3、在查询所有之前设置开始页和每页条数：PageHelper.startPage(页码,每页条数)

4、查询之后使用PageInfo封装分页数据: PageInfo<> info=new PageInfo(list);
```

### 1.导入开发jar包

	`jsqlparser-3.1.jar` 和`pagehelper-5.1.10.jar` , 记得要 add as library

### 2.在mybatis的核心配置文件中集成分页插件

```xml
<!--集成PageHelper分页插件-->
<plugins>
  <plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
```

### 3.在查询所有信息之前设置查询的页数和每页展示条数

```java
@Test
public void testSelectAll() throws IOException {
  //1、加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2、获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3、获取SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4、获取接口的实现类代理对象，执行操作
  StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

  //使用分页助手设置页数和每页条数
  PageHelper.startPage(1,3); //------------------------------

  List<Student> list = mapper.selectAll();
    
  //5、处理/打印结果
  list.forEach(stu-> System.out.println(stu));
  //6、释放资源
  sqlSession.close();
}
```

### 4.在获取分页数据之后封装分页信息

```java
@Test
public void testSelectAll() throws IOException {
  //1、加载核心配置文件
  InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
  //2、获取工厂对象
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
  //3、获取SqlSession核心对象
  SqlSession sqlSession = factory.openSession(true);
  //4、获取接口的实现类代理对象，执行操作
  StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);

  //使用分页助手设置页数和每页条数
  //参数1:当前页码，参数2:每页显示条数
  PageHelper.startPage(1,3); //------------------------------

  List<Student> list = mapper.selectAll();

  //封装分页信息
  PageInfo<Student> info=new PageInfo<>(list);//---------------------------
  
  
  //打印分页信息
  System.out.println("当前页数："+info.getPageNum());  //1
  System.out.println("每页展示条数："+info.getPageSize()); //3
  System.out.println("总页数："+info.getPages());
  System.out.println("总条数："+info.getTotal());
  System.out.println("当前页条数："+info.getSize());
  System.out.println("上一页："+info.getPrePage());
  System.out.println("下一页："+info.getNextPage());
  System.out.println("是否是第一页："+info.isIsFirstPage());
  System.out.println("是否是最后一页："+info.isIsLastPage());

  //5、处理/打印结果
  list.forEach(stu-> System.out.println(stu));
  //6、释放资源
  sqlSession.close();
}
```

## ==mybatis多表操作(重要)==

### 数据库准备

```sql
CREATE DATABASE db2;

USE db2;

-- 一对一
CREATE TABLE person(
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20),
  age INT
);

INSERT INTO person VALUES (NULL,'张三',23);
INSERT INTO person VALUES (NULL,'李四',24);
INSERT INTO person VALUES (NULL,'王五',25);

CREATE TABLE card(
  id INT PRIMARY KEY AUTO_INCREMENT,
  number VARCHAR(30),
  pid INT,
  CONSTRAINT cp_fk FOREIGN KEY (pid) REFERENCES person(id)
);

INSERT INTO card VALUES (NULL,'12345',1);
INSERT INTO card VALUES (NULL,'23456',2);
INSERT INTO card VALUES (NULL,'34567',3);


-- 一对多
CREATE TABLE classes( -- 班级表
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20)
);
INSERT INTO classes VALUES (NULL,'黑马一班');
INSERT INTO classes VALUES (NULL,'黑马二班');


CREATE TABLE student( -- 学生表
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(30),
  age INT,
  cid INT,
  CONSTRAINT cs_fk FOREIGN KEY (cid) REFERENCES classes(id)
);
INSERT INTO student VALUES (NULL,'张三',23,1);
INSERT INTO student VALUES (NULL,'李四',24,1);
INSERT INTO student VALUES (NULL,'王五',25,2);
INSERT INTO student VALUES (NULL,'赵六',26,2);

-- 多对多
CREATE TABLE course(-- 课程
  id INT PRIMARY KEY AUTO_INCREMENT,
  NAME VARCHAR(20)
);
INSERT INTO course VALUES (NULL,'语文');
INSERT INTO course VALUES (NULL,'数学');


CREATE TABLE stu_cr(
  id INT PRIMARY KEY AUTO_INCREMENT,
  sid INT,
  cid INT,
  CONSTRAINT sc_fk1 FOREIGN KEY (sid) REFERENCES student(id),
  CONSTRAINT sc_fk2 FOREIGN KEY (cid) REFERENCES course(id)
);
INSERT INTO stu_cr VALUES (NULL,1,1);
INSERT INTO stu_cr VALUES (NULL,1,2);
INSERT INTO stu_cr VALUES (NULL,2,1);
INSERT INTO stu_cr VALUES (NULL,2,2);
```

### 一对一关系

#### 1 javabean类

```java
public class Card {
    private Integer id;     //主键id
    private String number;  //身份证号
  
    private Person p;       //所属人的对象
  
}
public class Person {
    private Integer id;     //主键id
    private String name;    //人的姓名
    private Integer age;    //人的年龄
}
```

#### 2 接口类

```java
public interface OneToOneMapper {
    //查询全部
    public abstract List<Card> selectAll();
}
```

#### 3 映射文件配置OneToOneMapper.xml

resultMap标签的作用：手动定义查询结果和bean对象的映射关系，一般用来处理查询结果和bean的属性不一致情况。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.itheima.mapper.OneToOneMapper">
  <!--配置查询结果和bean对象的对应关系-->
  <resultMap id="oneToOne" type="card">
    <!--封装主键信息，column="id"列名称，property="id"bean中的属性名称-->
    <id column="id" property="id"/>
    <!--封装非主键信息-->
    <result column="number" property="number"/>
    <!--配置包含的对象映射关系,配置一对一关系，javaType="person"表示bean的属性类型-->
    
    <association property="p" javaType="person">
      <!--封装主键信息-->
      <id column="pid" property="id"/>
      <!--封装非主键信息-->
      <result column="name" property="name"/>
      <result column="age" property="age"/>
    </association>
    
  </resultMap>

  <!-- 测试一对一关系：查询所有card的信息，包括每个card对应的person信息-->
  <select id="selectAll" resultMap="oneToOne">
    select c.*,p.NAME,p.age from card c ,person p where c.pid=p.id;
  </select>
  
</mapper>
```

**==注意：封装结果集不再使用resultType属性，而是使用resultMap属性。==**

#### 4 测试类

```java
/*
        测试一对一关系：查询所有card的信息，包括每个card对应的person信息
     */
@Test
public void testOneToOneMapper(){
  //加载资源
  SqlSession sqlSession = null;
  try {
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //获取SqlSessionFactory对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //获取SqlSession对象
    sqlSession = factory.openSession(true);
    //获取mapper接口的代理对象，执行方法
    OneToOneMapper mapper = sqlSession.getMapper(OneToOneMapper.class);
    List<Card> cards = mapper.selectAll();
    //打印结果
    cards.forEach(card -> System.out.println(card));
  } catch (IOException e) {
    e.printStackTrace();
  } finally {
    //释放资源
    sqlSession.close();
  }
}
```

### 一对多关系 (重要)

#### 1 javabean类

```java
public class Classes {
  private Integer id;     //主键id
  private String name;    //班级名称

  private List<Student> students; //班级中所有学生对象,一个班级中有多名学生
  
  // 生成 get/set 方法 
}

public class Student {
    private Integer id;     //主键id
    private String name;    //学生姓名
    private Integer age;    //学生年龄
  
    // 生成 get/set 方法 
}
```

#### 2 接口类

```java
public interface OneToManyMapper {
  //查询全部
  public abstract List<Classes> selectAll();
}
```

#### 3 映射文件配置

```xml
<resultMap id="OneToMany" type="classes">
        <id column="cid" property="id"/>
        <result column="cname" property="name"/>
        <!--
            collection:配置被包含的集合对象映射关系
            property:被包含对象变量名
            ofType：被包含对象的实际数据类型
        -->
        <collection property="students" ofType="student">
            <id column="sid" property="id"/>
            <result column="sname" property="name"/>
            <result column="sage" property="age"/>
        </collection>
    </resultMap>

    <select id="selectAll" resultMap="OneToMany">
        select c.id cid,c.NAME cname,s.id sid,s.NAME sname,s.age sage from classes c,student s where c.id=s.cid
    </select>
```

#### 4 测试类

```java
/*
        测试一对多关系：查询所有班级以及该班级的所有学生信息
     */
@Test
public void testOneToManyMapper(){
  //加载资源
  SqlSession sqlSession = null;
  try {
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //获取SqlSessionFactory对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //获取SqlSession对象
    sqlSession = factory.openSession(true);
    //获取mapper接口的代理对象，执行方法
    OneToManyMapper mapper = sqlSession.getMapper(OneToManyMapper.class);
    List<Classes> classes = mapper.selectAll();
    //打印结果
    classes.forEach(cla -> System.out.println(cla));
  } catch (IOException e) {
    e.printStackTrace();
  } finally {
    if(sqlSession!=null){
      //释放资源
      sqlSession.close();
    }
  }
}
```

### 多对多关系重要

#### 1 javabean类

```java
public class Student {
    private Integer id;     //主键id
    private String name;    //学生姓名
    private Integer age;    //学生年龄
    private List<Course> courses;   // 学生所选择的课程集合
}
public class Course {
    private Integer id;     //主键id
    private String name;    //课程名称
}
```

#### 2 接口类

```java
public interface ManyToManyMapper {
  //查询全部
  public abstract List<Student> selectAll();
}
```

#### 3 映射文件配置

```xml
<resultMap id="manyToMany" type="student">
  <id column="sid" property="id"/>
  <result column="sname" property="name"/>
  <result column="sage" property="age"/>

  <!--封装数据到集合中-->
  <collection property="courses" ofType="Course">
    <id column="cid" property="id"/>
    <result column="cname" property="name"/>
  </collection>
  
</resultMap>

<select id="selectAll"  resultMap="manyToMany">
  select s.id sid,s.NAME sname,s.age sage,c.id cid,c.NAME cname from student s,stu_cr sc,course c where s.id=sc.sid and c.id=sc.cid;
</select>
```

#### 4 测试类

```java
/*
        测试多对多关系：查询所有学生及其选课信息
     */
@Test
public void testManyToManyMapper(){
  //加载资源
  SqlSession sqlSession = null;
  try {
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //获取SqlSessionFactory对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //获取SqlSession对象
    sqlSession = factory.openSession(true);
    //获取mapper接口的代理对象，执行方法
    ManyToManyMapper mapper = sqlSession.getMapper(ManyToManyMapper.class);
    List<Student> list = mapper.selectAll();
    //打印结果
    list.forEach(stu -> System.out.println(stu));
  } catch (IOException e) {
    e.printStackTrace();
  } finally {
    if(sqlSession!=null){
      //释放资源
      sqlSession.close();
    }
  }
}
```

### ==注意：javaType属性和ofType属性的区别？==

**javaType的值用来表示bean对象的属性类型，ofType的值用来表示集合中元素的类型。**