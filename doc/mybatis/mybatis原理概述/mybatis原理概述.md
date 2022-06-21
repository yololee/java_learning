# Mybatis原理概述

- MyBatis 是一款优秀的持久层框架
- 它支持定制化 SQL、存储过程以及高级映射。
- MyBatis 避免了几乎所有的 JDBC 代码和 手动设置参数以及获取结果集。
- MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### 1.mybatis架构图

![在这里插入图片描述](https://img-blog.csdnimg.cn/f323a041ed4841658b8ca36ee6d41346.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_19,color_FFFFFF,t_70,g_se,x_16#pic_center)


### 2.mybatis核心组件

- `SqISessionFactoryBuilder (构建器)`: 根据配置信息或Java代码来构建 SqlSessionFactory 对象。作用:创建SqlSessionFactory对象。
- `SqlSessionFactory (会话工厂) `:好比是DataSource(创建连接的数据源) ,线程安全的,在应用运行期间不要重复创建多次,建议使用单例模式。作用: 创建SqlSession对象
- `SqlSession (会话) `:好比是Connection ,线程不安全的,每次使用开启新的SqlSession对象,使用完毕正常关闭,默认使用DefaultSqlSession。提供操作数据库的增删改查方法,可以调用操作方法,也可以操作`Mapper`组件。
- `Executor (执行器)` : SqlSession本身不能直接操作数据库,需要`Executor`来完成,该接口有两个实现: 缓存执行器(缺省)、基本执行器。
- `MappedStatement` :映射语句(MappedStatement)封装 执行语句时的信息: 如SQL、输入参数、输出结果

> 原理图

![在这里插入图片描述](https://img-blog.csdnimg.cn/77360e1ac115467bb210b00cf19d4e97.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


更具体的底层原理图

![在这里插入图片描述](https://img-blog.csdnimg.cn/7292a5cb45b649589fae7caa1a8ece41.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5ouS57ud54as5aSc5ZWK,size_17,color_FFFFFF,t_70,g_se,x_16#pic_center)


涉及的对象：

- Configuration : <font color= 'red'>MyBatis 全局配置对象,封装所有配置信息</font>
- `SqlSession:`表示和数据库交互的会话，完成必要<font color= 'red'>数据库增删改查功能</font>
- `Executor:`执行器，是mybatis调度的核心，<font color= 'red'>负责sql语句的生成和查询缓存的维护</font>
  - BaseExecutor：底层的执行器，先从一级缓存中取查询，如果没有查询数据库
  - CachingExecutor：带有二级缓存的执行器，先去二级缓存中寻找是否有数据

- `StatementHandler :`语句处理器,在创建对象之前先创建参数处理器和结果集处理器。<font color= 'red'>封装了JDBC的DML、DQL 操作、参数设置</font>
  - `ParameterHandler :`参数处理器,把<font color= 'red'>用户传入参数转换为JDBC需要的参数值</font>
  - `ResultSetHandler :`结果集处理器,把<font color= 'red'>结果集中的数据封装到List集合</font>
- `TypeHandler :`类型转换器, <font color= 'red'>Java类型和JDBC类型的相互转换</font>
- `MappedStatement :`映射语句对象,<font color= 'red'>维护了一条< insert|update|delete|select>节点的封装</font>
  - SqlSource : SQL源, <font color= 'red'>根据用户传入的参数生成SQL语句,并封装到BoundSql中</font>
  - BoundSql : SQL绑定, <font color= 'red'>封装SQL语句和对应的参数信息</font>
    
    

### 3.作用域和生命周期

- `SqISessionFactoryBuilder`

  - **这个类可以被实例化、使用和丢弃,一旦创建了SqlSessionFactory ,就不再需要它了**。`因此SqlSessionFactoryBuilder实例的最佳作用域是方法作用域(也就是局部方法变量)`
  - 你可以重用SqlSessionFactoryBuilder 来创建多个SqlSessionFactory 实例,但是最好还是不要让其一直存在, 以保证所有的XML解析资源开放给更重要的事情

- `SqISessionFactory`

  - SqlSessionFactory **一旦被创建就应该在应用的运行期间一直存在 (单例)**,没有任何理由对它进行清除或重建
  - 使用SqISessionFactory 的最佳实践是在应用运行期间<b>不要重复创建多次</b>, 多次重建SqlSessionFactory被视为一种代码"坏味道( bad smell)”.因此SqISessionFactory 的最佳作用域是<b>应用作用域</b>。有很多方法可以做到,<font color = 'red'>最简单的就是使用</font><font color = 'blue'>单例模式</font><font color = 'red'>或者静态单例模式</font>

- `SqlSession`

  - 每个线程都应该有它自己的SqlSession 实例。SqlSession的实例不是线程安全的,因此是不能被共享的,所以它的最佳的作用域是请求或方法作用域
  - **绝对不能将SqlSession 实例的引用放在一个类的静态域,甚至一个类的实例变量也不行**。也绝不能将SqlSession 实例的引用放在任何类型的管理作用域中
  - 比如Servlet架构中的HttpSession. 如果你现在正在使用一种Web框架,要考虑SqISession放在一个和HTTP请求对象相似的作用域中。换句话说,每次收到的HTTP请求,就可以打开一个Sq|Session ,返回一个响应,就关闭它。这个关闭操作是很重要的,你应该把这个关闭操作放到finally 块中以确保每次都能执行关闭。

  