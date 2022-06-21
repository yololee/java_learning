## ==mybatis注解开发实现CRUD(重要)==

### 1 Mapper接口中使用注解

```java
public interface StudentMapper {

  //查询全部
  @Select("select * from student")
  public abstract List<Student> selectAll();

  //根据id查询
  @Select("select * from student where id=#{id}")
  public abstract Student selectById(Integer id);

  //新增数据
  @Insert("insert into student values(null,#{name},#{age})")
  public abstract Integer insert(Student stu);

  //修改数据
  @Update("update student set name=#{name},age=#{age} where id=#{id}")
  public abstract Integer update(Student stu);

  //删除数据
  @Delete("delete from student where id=#{id}")
  public abstract Integer delete(Integer id);
  
}
```

### 2 核心配置文件引入Mapper

```xml
<!--加载映射文件-->
<mappers>
  <!--引入mapper接口的位置-->
  <!--<mapper class="com.itheima.mapper.StudentMapper"/>-->

  <!--引入整个包-->
  <package name="com.itheima.mapper"/>
</mappers>
```

### 3 测试类

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
    Student student = mapper.selectById(3);   //底层使用sqlSession.selectOne()方法
    //5、处理/打印结果
    System.out.println(student);
    //6、释放资源
    sqlSession.close();
  }
  @Test
  public void testInsert() throws IOException {
    //1、加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //2、获取工厂对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //3、获取SqlSession核心对象
    SqlSession sqlSession = factory.openSession(true);
    //4、获取接口的实现类代理对象，执行操作
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    Integer count = mapper.insert(new Student(null, "刘曲", 30));
    //5、处理/打印结果
    System.out.println("count = " + count);
    //6、释放资源
    sqlSession.close();
  }
  @Test
  public void testUpdate() throws IOException {
    //1、加载核心配置文件
    InputStream is = Resources.getResourceAsStream("MybatisConfig.xml");
    //2、获取工厂对象
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    //3、获取SqlSession核心对象
    SqlSession sqlSession = factory.openSession(true);
    //4、获取接口的实现类代理对象，执行操作
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    Integer count = mapper.update(new Student(13, "刘曲", 40));
    //5、处理/打印结果
    System.out.println("count = " + count);
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
    Integer count = mapper.delete(13);
    //5、处理/打印结果
    System.out.println("count = " + count);
    //6、释放资源
    sqlSession.close();
  }
}
```

## mybatis注解多表开发(注解)

实现复杂关系映射之前我们可以在映射文件中通过配置\<resultMap>来实现，使用注解开发后，我们可以使用@Results注解，@Result注解，@One注解，@Many注解组合完成复杂关系的配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210104215806787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI5NjMxMw==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210104215820116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI5NjMxMw==,size_16,color_FFFFFF,t_70#pic_center)


### 1 一对一

#### OneToOneMapper接口中执行查询

```java
public interface OneToOneMapper {
  //查询全部
  @Select("select * from card")
  @Results({
    //id = true 表示是主键，默认值是false
    @Result(id = true,column = "id",property = "id"), //id标签
    @Result(column = "number",property = "number"), //result标签
    //column = "pid" 此处的column = "pid"表示拿着pid的值传递到selectById方法中查询对应的person信息，封装到property = "p"属性中
    @Result(
      column = "pid",//根据查询出的card表中的pid字段来查询person表
      property = "p",//被包含对象的变量名
      javaType = Person.class,//被包含对象的实际数据类型
        /*
              one=@One 一对一固定写法
              select属性：指定调用哪个接口中的那个方法
                     */
      one = @One(select = "com.itheima.mapper.PersonMapper.selectById")), //result标签
  })
  public abstract List<Card> selectAll();
}
```

#### 新建PersonMaper接口，创建方法根据id查询

```java
public interface PersonMapper {
  @Select("select * from person where id=#{id}")
  Person selectById(Integer id);
}
```

#### 测试方法

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
    if(sqlSession!=null){
      //释放资源
      sqlSession.close();
    }
  }
}
```

### 2 一对多

#### OneToManyMapper接口中执行查询

```java
//查询全部
@Select("select * from classes")
@Results({
  @Result(id = true,column = "id",property = "id"),
  @Result(column = "name",property = "name"),
  @Result(
    property = "students",
    javaType = List.class,
    column = "id",
    many = @Many(select = "com.itheima.mapper.StudentMapper.selectByCid")),
})
public abstract List<Classes> selectAll();
```

#### 新建StudentMaper接口，创建方法根据id查询

```java
public interface StudentMapper {
  @Select("select * from student where cid=#{cid}")
  List<Student> selectByCid(int cid);
}
```

#### 测试方法

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

### 3 多对多

#### ManyToManyMapper接口中执行查询

```java
public interface ManyToManyMapper {
  //查询全部
  //@Select("select * from student") //没有选课的学生也能被查到
  @Select("select distinct s.* from student s,stu_cr sc where s.id=sc.sid") //没有选课的学生查不到
  @Results({
    @Result(id = true,column = "id",property = "id"),
    @Result(column = "name",property = "name"),
    @Result(column = "age",property = "age"),
    @Result(
      property = "courses",
      javaType = List.class,
      column = "id",
      many = @Many(select = "com.itheima.mapper.CourseMapper.selectBySid")),
  })
  public abstract List<Student> selectAll();
}

```

#### 新建CourseMaper接口，创建方法根据id查询

```java
public interface CourseMapper {
  @Select("select c.* from stu_cr sc,course c where sc.cid=c.id and sc.sid=#{sid}")
  public List<Course> selectBySid(Integer sid);
}
```

#### 测试方法

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

### 4 懒加载 --- 了解

#### 4.1 在核心配置文件中开启懒加载

```xml
<!--开启全局的延迟加载开关-->
<setting name="lazyLoadingEnabled" value="true"/>

<!--取消默认触发懒加载的方法-->
<setting name="lazyLoadTriggerMethods" value=""/>
```

#### 4.2 如果某个多表操作需要取消懒加载，就使用fetchType = FetchType.EAGER取消，那么就是立即加载

```java
public interface ManyToManyMapper {
  //查询全部
  //@Select("select * from student")
  @Select("select distinct s.* from student s,stu_cr sc where s.id=sc.sid")
  @Results({
    @Result(id = true,column = "id",property = "id"),
    @Result(column = "name",property = "name"),
    @Result(column = "age",property = "age"),
    @Result(
      property = "courses",
      column = "id",
      many = @Many(select = "com.itheima.mapper.CourseMapper.selectBySid",fetchType = FetchType.EAGER)),
  })
  public abstract List<Student> selectAll();
```

## mybatis构建SQL(了解)

```java
-查询操作
    @SelectProvider：生成查询用的SQL语句注解
    	type属性：生成SQL语句功能类字节码对象
    	method属性：指定调用方法(方法名字)
    
-新增操作
    @InsertProvider：生成新增用的SQL语句注解
    	type属性：生成SQL语句功能类字节码对象
    	method属性：指定调用方法(方法名字)
    
-更新操作
    @UpdateProvider：生成修改的SQL语句注解
    	type属性：生成SQL语句功能类字节码对象
    	method属性：指定调用方法(方法名字)
    
-删除操作
    @DeleteProvider：生成修改的SQL语句注解
    	type属性：生成SQL语句功能类字节码对象
    	method属性：指定调用方法(方法名字)
```



### 1.提供类和方法返回Sql语句

```java
public class ReturnSql {
  
  public String getSelectCondition(Student stu){
        //select * from student where id=#{id} and name=#{name} and age=#{age}
        //创建初始化语句
    
    		//  select * from student
        SQL sql = new SQL().SELECT("*").FROM("student");

        //判断
        if(stu!=null && stu.getId()!=null){
            sql.WHERE("id=#{id}");
        }
        if(stu!=null && stu.getName()!=null){
            sql.WHERE("name=#{name}"); //where方法会自动添加上and关键字
        }
        if(stu!=null && stu.getAge()!=null){
            sql.WHERE("age=#{age}"); //where方法会自动添加上and关键字
        }
    
    		//  select * from student where id=#{id} and name=#{name} and age=#{age}
        return sql.toString();
    }

  /*public String getSelectCondition(Student stu){
    //select * from student where id=#{id} and name=#{name} and age=#{age}
    //创建初始化语句
    StringBuilder sb=new StringBuilder("select * from student where 1=1");
    //判断
    if(stu!=null && stu.getId()!=null){
      sb.append(" and id=#{id}");
    }
    if(stu!=null && stu.getName()!=null){
      sb.append(" and name=#{name}"); //where方法会自动添加上and关键字
    }
    if(stu!=null && stu.getAge()!=null){
      sb.append(" and age=#{age}"); //where方法会自动添加上and关键字
    }
    return sb.toString();
  }*/
}
```

### 2.在Mapper接口的方法上使用@XxxxProvider注解使用构建的sql

```java
@SelectProvider(type = ReturnSql.class,method = "getSelectCondition")
public abstract List<Student> selectCondition(Student stu);
```

### 3.测试类测试

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

### 补充：注解方式实现动态SQL的脚本方式

```java
//多条件查询
@Select("<script>select * from student\n" +
        "        <where>\n" +
        "            <if test=\"id != null\">\n" +
        "                id=#{id}\n" +
        "            </if>\n" +
        "            <if test=\"name != null and name !='' \">\n" +
        "                and name=#{name}\n" +
        "            </if>\n" +
        "            <if test=\"age != null\">\n" +
        "                and age=#{age}\n" +
        "            </if>\n" +
        "        </where></script>")
public abstract List<Student> selectCondition(Student stu);

//根据多个id查询
@Select("<script>select * from student\n" +
        "        <where>\n" +
        "            <foreach collection=\"list\" open=\"id in(\" close=\")\" item=\"id\" separator=\",\">\n" +
        "                #{id}\n" +
        "            </foreach>\n" +
        "        </where></script>")
public abstract List<Student> selectByIds(List<Integer> ids);
```
