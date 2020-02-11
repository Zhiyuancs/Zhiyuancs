---
title: Mybatis学习笔记
date: 2020-02-01 19:28:05
tags:
	Mybatis
categories:
	学习笔记
---

### 1.mybatis简单应用

#### 1.1步骤：

1.编写实体类User，建立数据库表对应的对象（变量及其get、set方法）；

2.编写持久层接口IUserDao，数据库操作的接口，如（`List<User> findall()`）;

3.编写持久层接口的映射文件IUserDao.xml，要求：必须和持久层接口在相同的包中，必须以持久层接口名称命名文件名；

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE mapper 
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 			   					"http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
<mapper namespace="com.itheima.dao.IUserDao">
    <!-- 配置查询所有操作 --> 
    <select id="findAll" resultType="com.itheima.domain.User"> 
        select * from user 
    </select>
</mapper>
```

4.编写SqlMapConfig.xml;

```XML
<<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
    <!-- mybatis的主配置文件 -->
<configuration>
    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/eesy_mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="1234"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <mappers>
        <mapper resource="com/itheima/dao/IUserDao.xml"/>
    </mappers>
</configuration>
```

#### 1.2 基于注解的mybatis使用

1.在持久层接口中添加注解；

```java
public interface IUserDao{
    @Select("select * from user")
    List<User> findAll();
}
```

2.修改SqlMapConfig.xml，此时不需要IUserDao.xml。

```xml
<mappers>
	<mapper class="com.neu.dao.IUserDao"/>
</mappers>
```

### 2.基于代理Dao实现CRUD操作

使用要求：

1、持久层接口和持久层接口的映射配置必须在相同的包下；

2、持久层映射配置中mapper标签的namespace属性取值必须是持久层接口的全限定类名；

3、SQL语句的配置标签`<select>,<insert>,<delete>,<update>`的id属性必须和持久层接口的方法名相同。

```xml
<!-- 根据id查询用户 -->
<select id="findById" parameterType="int" resultMap="com.neu.domain.User">
    select * from user where id = #{uid}
</select>
<!-- 
	sql语句中使用#{}，表示占位符，用于替换具体数据，相当于jdbc中的？ 
-->
<!-- 插入用户-->
<insert id="saveUser" parameterType="com.neu.domain.User">
    insert into user(username,birthday,sex)
    	values(#{username},#{birthday},#{sex})
</insert>
```

```java
//测试类
public class MybatisTest {

    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    @Before//用于在测试方法执行之前执行
    public void init()throws Exception{
        //1.读取配置文件，生成字节输入流
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.获取SqlSessionFactory
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        //3.获取SqlSession对象
        sqlSession = factory.openSession();
        //4.获取dao的代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    @After//用于在测试方法执行之后执行
    public void destroy()throws Exception{
        //提交事务
        sqlSession.commit();
        //6.释放资源
        sqlSession.close();
        in.close();
    }
    
    @Test 
    public void testFindOne() { 
        //执行操作 
        User user = userDao.findById(41); 
        System.out.println(user); 
    }
```

### 3. Mybatis参数

parameterType 配置参数：

1、基本类型和String我们可以直接写类型名称，也可以使用包名.类名的方式，例如：java.lang.String。

2、实体类类型，目前我们只能使用全限定类名。原因是mybaits在加载时已经把常用的数据类型注册了别名，从而我们在使用时可以不写包名，而我们的是实体类并没有注册别名，所以必须写全限定类名。

### 4.Mybatis输出结果封装

当实体类属性和数据库表的列名不一致时，是无法将查询结果封装到实体类对象的，可以使用别名查询解决。

```xml
<!-- 别名与实体类属性名一致-->
<select id="findAll" resultType="com.neu.domain.User">
    select id as userId,username as userName,sex as userSex from user
</select>
```

#### resultMap结果类型

resultMap标签可以建立查询的列名和实体类的属性名称不一致时建立对应关系。从而实现封装。

1、定义resultMap

```xml
<!-- 建立User实体和数据库表的对应关系
	type属性：指定实体类的全限定类名
	id属性：给定一个唯一标识，是给查询select标签引用用的。
-->
<resultMap type="com.neu.domain.User" id="userMap">
    <id column="id" property="userId"/>
    <result column="usermane" property="userName"/>
    <result column="sex" property="userSex"/>
</resultMap>
<!-- 
	id标签：用于指定主键字段
	result标签：用于指定非主键字段
	column标签：用于指定数据库列名
	property标签：用于指定实体类属性名称
-->
```

2、映射配置

```xml
<select id="findAll" resultMap="userMap">
     select * from user;
</select>
```

### 5. SqlMapConfig.xml配置文件

#### 5.1 SqlMapConfig.xml中配置的内容和顺序

```
-properties(属性)
	--property
-settings(全局配置参数)
	--setting
-typeAliases(类型别名)
	--typeAliase
	--package
-typerHandlers(类型处理器)
-objectFactory(对象工厂)
-plugins(插件)
-environments(环境集合属性对象)
	--environment(环境子属性对象)
		---transactionManager(事务管理)
		---dataSource(数据源)
-mappers(映射器)
	--mapper
	--package
```

#### 5.2 properties

##### 5.2.1 第一种配置方式

```xml
<properties>
	<property name="jdbc.driver" value="com.mysql.jdbc.Driver"/>
    <property name="jdbc.url" value="jdbc:mysql://localhost/3306/ssm"/>
    ……
</properties>
```

##### 5.2.2 第二种方式

1.在classpath下定义db.properties文件

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/eesy_mybatis
jdbc.username=root
jdbc.password=1234
```

2.properties标签配置

```xml
<!-- 配置properties-->
<properties url="绝对路径"></properties>
<properties resource="jdbc.properties"></properties>

<!--配置连接池-->
<dataSource type="POOLED">
    <property name="driver" value="${jdbc.driver}"></property>
    <property name="url" value="${jdbc.url}"></property>
    <property name="username" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</dataSource>
```

#### 5.3 typeAliases类型别名

自定义别名

```xml
<typeAliases>
    <!-- 单个别名定义 -->
	<typeAlias alias="user" type="com.neu.dao.User"/>
    <!-- 批量别名定义，扫描整个包下的类，别名为类名（首字母大写或小写都可以） -->
    <package name="com.neu.domain"/>
	<package name="其他包"/>
</typeAliases>
```

#### 5.4 mappers

1.`<mapper resource="" />`

```
使用相对于类路径的资源
<mapper resource="com/neu/dao/IUserDao.xml"/>
```

2.`<mapper class="" />`

```
使用mapper接口类路径
<mapper class="com.neu.dao.UserDao"/>
注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。
```

3.`<package name="" />`

```
注册指定包下的所有mapper接口
<package name="cn.itcast.mybatis.mapper"/>
注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。
```

### 6. Mybatis的事务控制

Mybatis中需要手动进行事务的提交，原因是在连接池中取出的连接都会调用`connection.setAutoCommit(false)`方法，所以要加上`sqlSession.commit()`提交事务。

可以通过设置实现自动提交。

```java
@Before//用于在测试方法执行之前执行
public void init()throws Exception{
    //1.读取配置文件
    in = Resources.getResourceAsStream("SqlMapConfig.xml");
    //2.获取SqlSessionFactory
    SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
    //3.获取SqlSession对象(实现自动提交，查看源码，参数含有autocommit)
    sqlSession = factory.openSession(true);
    //4.获取dao的代理对象
    userDao = sqlSession.getMapper(IUserDao.class);
}
```

### 7. 动态SQL

通过 if, choose, when, otherwise, trim, where, set, foreach等标签，可组合成非常灵活的SQL语句。

#### 7.1 `<if>`标签

```xml
<select id="findByUser" resultType="user" parameterType="user">
	select * from user where 1=1
    <if test="username!=null and username!=' ' ">
    	and username like #{username}
    </if>
    <if test="address!= null">
    	and address like #{address}
    </if>
</select>
<!-- test属性中写的是为对象的属性名及其条件，
	符合条件的，在sql语句中加上标签内的语句 
-->
```

#### 7.2 `<where>`标签

为了简化上面`where 1=1`的条件拼装，可以使用where标签简化。

```xml
<!-- sql重用 -->
<sql id="defaultSql">
	select * from user 
</sql>
<select id="findByUser" resultType="user" parameterType="user">
    <include refid="defaultSql"/>
    <where>
        <if test="username!=null and username!=' ' ">
            username like #{username}
        </if>
        <!-- where标签会自动插入where，如果开头为and，会剔除掉and -->
        <if test="address!= null">
            and address like #{address}
        </if>
    </where>
</select>
```

#### 7.3 `<foreach>`标签

```xml
<!-- 查询所有用户在id的集合之中 -->
<!-- select * from user where id in (1,2,3,4,5); -->
<select id="findInIds" resultType="user" parameterType="queryvo">
    <include refid="defaultSql"></include>
    <where>
        <!-- ids为queryvo中的属性名，为list -->
    	<if test="ids != null and ids.size() > 0">
            <foreach collection="ids" open="id in (" 
                     close=")" item="id" separator=",">
                #{uid}
            </foreach>
        </if>
    </where>
</select>

collection:代表要遍历的集合
open：语句的开始部分
item：代表遍历集合的每个元素，生成的变量名
sperator：分隔符
```

### 8. 多表查询

一对一

```xml
<resultMap type="com.neu.domain.User" id="userMap">
    <id column="id" property="userId"/>
    <result column="usermane" property="userName"/>
    <result column="sex" property="userSex"/>
    <association property=" " javaType=" ">
        <!-- 指定从表方的引用实体属性 -->
    </association>
</resultMap>
```

一对多（左外连接）

```xml
<!-- collection是用于建立一对多中集合属性的对应关系
	ofType用于指定集合元素的数据类型
-->
<!-- 如一个用户可以有多个账户，分用户表、账户表，
	在User对象中加入账户集合
-->
<resultMap type="com.neu.domain.User" id="userMap">
    <id column="id" property="userId"/>
    <result column="usermane" property="userName"/>
    <result column="sex" property="userSex"/>
    <!-- property:关联查询结果存储在User对象的属性名称 -->
    <collection property="accounts" ofType="account">
    	<id column="aid" property="id"/>
    	<result column="uid" property="uid"/>
	</collection>
</resultMap>
```

### 9. 延迟加载

延迟加载就是在需要用到数据时才进行加载，不需要用到数据时就不加载数据。延迟加载也称懒加载。在多表查询中，即将其分成多次单表查询，resultMap中会加入其他表的对象，`association和collection`有`select和column`属性。SqlMapConfig.xml需要配置setting信息。

```xml
<settings>
    <!-- 打开延迟加载的开关 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 将积极加载改为消极加载，即延迟加载 -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

### 10. Mybatis缓存

#### 10.1 一级缓存

一级缓存是SqlSession级别的缓存，只要SqlSession没有关闭，就一直存在。

#### 10.2 二级缓存

二级缓存是mapper映射级别的缓存，多个SqlSession去操作同一个Mapper映射的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

1、需要在SqlMapConfig.xml文件中开启二级缓存。

2、配置相关的Mapper映射文件。`<cache></cache>`

3、配置statement上的useCache属性。

当我们在使用二级缓存时，所缓存的类一定要实现java.io.Serializable接口，这种就可以使用序列化方式来保存对象。

### 11. 注解方式

#### 11.1 常用注解

```
@Insert:实现新增 
@Update:实现更新 
@Delete:实现删除 
@Select:实现查询 
@Result:实现结果集封装 
@Results:可以与@Result一起使用，封装多个结果集 
@ResultMap:实现引用@Results定义的封装 
@One:实现一对一结果集封装 
@Many:实现一对多结果集封装 
@SelectProvider: 实现动态SQL映射 
@CacheNamespace:实现注解二级缓存的使用
```

#### 11.2 注解说明

```
@Results注解
代替标签<resultMap>
该注解中可以使用单个@Result注解，也可以使用@Result集合

@Result注解
代替了<id>标签和<result>标签
属性：id 是否是主键字段
column 数据库的列名
property需要装配的属性名
one 需要使用的@One注解（@Result（one=@One）（）））
many 需要使用的@Many注解（@Result（many=@many）（）））

@One注解（一对一）
代替了<assocation>标签，是多表查询的关键，在注解中用来指定子查询返回单一对象。
@One注解属性介绍：
select 指定用来多表查询的sqlmapper
fetchType会覆盖全局的配置参数lazyLoadingEnabled
@Result(column=" ",property="",one=@One(select=""))

@Many注解（多对一）
代替了<Collection>标签,是多表查询的关键，在注解中用来指定子查询返回对象集合。
@Result(property="",column="",many=@Many(select=""))
```

#### 11.3 基于注解的二级缓存

1、在SqlMapConfig中开启二级缓存支持；

```xml
<!-- 配置二级缓存 --> 
<settings> 
    <!-- 开启二级缓存的支持 --> 
    <setting name="cacheEnabled" value="true"/> 
</settings>
```

2、在持久层接口中使用注解配置二级缓存；

```java
@CacheNamespace(blocking=true)//mybatis基于注解方式实现配置二级缓存
public interface IUserDao {}
```

