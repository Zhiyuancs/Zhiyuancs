---
title: Spring学习笔记
date: 2020-02-04 14:19:52
tags:
	Spring
categories:
	学习笔记
---

### 1. 基于XML的IOC配置

1、在类的根路径下创建一个任意名称的xml文件,如bean.xml,添加依赖。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

2、让spring管理资源，在配置文件中配置service和dao 

```xml
<!-- bean标签：用于配置让spring创建对象，
	并且存入ioc容器之中 
	id属性：对象的唯一标识
	class属性：指定要创建对象的全限定类名 
-->
<bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl"></bean>
```

3、使用配置

```java
//1.使用ApplicationContext接口，就是在获取spring容器 
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml"); 
//2.根据bean的id获取对象 
IAccountService aService = (IAccountService) ac.getBean("accountService");
```

掌握Spring中工厂类的结构图。

### 2. Spring中IOC细节

1、BeanFactory 和ApplicationContext 的区别

```
BeanFactory 是Spring 容器中的顶层接口。
ApplicationContext 是它的子接口。
BeanFactory 和ApplicationContext 的区别：
	创建对象的时间点不一样。
		ApplicationContext：只要一读取配置文件，默认情况下就会创建对象。
		BeanFactory：使用时创建对象。
```

2、ApplicationContext 接口的实现类

```
ClassPathXmlApplicationContext：
它是从类的根路径下加载配置文件 推荐使用这种
FileSystemXmlApplicationContext：
它是从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。
AnnotationConfigApplicationContext:
当我们使用注解配置容器对象时，需要使用此类来创建spring 容器。它用来读取注解。
```

```java
@Test 
public void testUpdateAccount() { 
    ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml"); 	IAccountService as = ac.getBean("accountService",IAccountService.class); 
    Account account = as.findAccountById(1); 
    account.setMoney(20301050f); 
    as.updateAccount(account); 
}
```

3、bean标签和管理对象细节

作用：配置对象让Spring来创建，默认情况下它调用的是类中的无参构造函数，如果没有无参构造函数则无法创建。

属性： id：给对象在容器中提供一个唯一标志，用于获取对象

​	class：指定类的全限定类名，用于反射创建对象。默认情况下调用无参构造函数。

​	scope：指定对象的作用范围。

​				singleton：默认值，单例

​				prototype：多例

​				request：WEB项目中,Spring创建一个Bean的对象,将对象存入到request域中.

​				session：WEB项目中,Spring创建一个Bean的对象,将对象存入到session域中.

​				global session：WEB项目中,应用在Portlet环境.

​	init-method：指定类中的初始化方法名称。

​	destroy-method：指定类中销毁方法名称。

4、实例化Bean的三种方式

```xml
第一种方式：使用默认无参构造函数
<!--在默认情况下： 它会根据默认无参构造函数来创建类对象。
	如果bean中没有默认无参构造函数，将会创建失败。-->
	<bean id="accountService" 
          class="com.itheima.service.impl.AccountServiceImpl"/>

第二种方式：spring管理静态工厂-使用静态工厂的方法创建对象
/** 
* 模拟一个静态工厂，创建业务层实现类 
*/ 
public class StaticFactory { 
	public static IAccountService createAccountService(){ 
		return new AccountServiceImpl(); 
	} 
}

<!-- 此种方式是: 
	使用StaticFactory类中的静态方法createAccountService创建对象，并存入spring容器 
	id属性：指定bean的id，用于从容器中获取 
	class属性：指定静态工厂的全限定类名 
	factory-method属性：指定生产对象的静态方法 
-->
<bean id="accountService" 
      class="com.itheima.factory.StaticFactory" 
      factory-method="createAccountService">
</bean>

第三种方式：spring管理实例工厂-使用实例工厂的方法创建对象
/** 
* 模拟一个实例工厂，创建业务层实现类 
* 此工厂创建对象，必须现有工厂实例对象，再调用方法 
*/
public class StaticFactory { 
	public  IAccountService createAccountService(){ 
		return new AccountServiceImpl(); 
	} 
}
<!-- 此种方式是： 
	先把工厂的创建交给spring来管理。 
	然后在使用工厂的bean来调用里面的方法 
	factory-bean属性：用于指定实例工厂bean的id。 
	factory-method属性：用于指定实例工厂中创建对象的方法。 
-->
<bean id="instancFactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService" 
      factory-bean="instancFactory" 
      factory-method="createAccountService"></bean>
```

### 3. Spring的依赖注入

#### 3.1 构造函数注入

使用构造函数注入需要在类中提供一个对应参数列表的构造函数。涉及标签为`constructor-arg`，属性为`index`指定参数在构造函数参数列表的索引位置，`type`指定参数在构造函数中的数据类型，`name`指定参数在构造函数中的名称，`value`赋值的为基本数据类型和String类型，`ref`赋值为其他bean类型，即在配置文件中配置过的bean。

```xml
<bean id="accountService" class="com.neu.service.impl.AccountServiceImpl">
	<constructor-arg name="name" value="张三"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
<bean id="now" class="java.util.Date"></bean>
```

#### 3.2 set方法注入

使用set方法给bean属性传值需要先在类中方法定义set方法，涉及标签为`property`，属性为`name`，`ref`，`value`。

```xml
<bean id="accountService" class="com.neu.service.impl……">
	<property name="name" value="test"></property>
    ……
</bean>
```

#### 3.3 使用p名称空间注入数据

此种方式是通过**在xml中导入p名称空间**，使用p:propertyName来注入数据，它的本质仍然是调用类中的set方法实现注入功能。

```xml
<bean id="accountService" class="……" 
      p:name="test" p:age="21" p:birthday-ref="now"/>
```

#### 3.4 注入集合属性

给类中的集合成员传值，它用的也是set方法注入的方式，只不过变量的数据类型都是集合。我们这里介绍注入数组，List,Set,Map,Properties。

```xml
<!-- 
	List结构：array,list,set
	Map结构：map,entry,props,prop
-->
<bean id="accountService" class="……"> 
    <!-- 在注入集合数据时，只要结构相同，标签可以互换 --> 
    <!-- 给数组注入数据 --> 
    <property name="myStrs"> 
        <set> 
            <value>AAA</value> 
            <value>BBB</value> 
            <value>CCC</value> 
        </set> 
    </property>
    <!-- 注入Map数据 --> 
    <property name="myMap"> 
        <props> 
            <prop key="testA">aaa</prop> 
            <prop key="testB">bbb</prop> 
        </props> 
    </property>
</bean>
```

### 4. 基于注解的IOC配置

基于注解整合时，导入约束时需要多导入一个context名称空间下的约束。同时需要告知spring创建容器时需要扫描的包。

```xml
<!-- 告知spring创建容器时要扫描的包 --> 
<context:component-scan base-package="com.neu"></context:component-scan>
```

#### 4.1 用于创建对象的

相当于`<bean id="" class="">`

##### 4.1.1 @Component

作用：把资源让spring来管理，相当于xml中配置一个bean

属性：value：指定bean的id。如果不指定value属性，默认bean的id是当前类的类名。首字母小写。

##### 4.1.2  @Controller   @Service    @Repository

为@Component的衍生注解。

@Controller:一般用于表现层注解

@Service：一般用于业务层注解

@Repository:一般用于持久层注解

属性：value

#### 4.2 用于注入数据的

相当于`<property name="" ref="">`

##### 4.2.1 @Autowired

作用：自动按照类型注入，当有多个类型匹配时，使用要注入的对象变量名称作为bean的id，在spring容器查找，找到了也可以注入成功。

##### 4.2.2 @Qualifier

在自动按照类型注入的基础之上，再按照Bean的id注入。它在给字段注入时不能独立使用，必须和@Autowire一起使用；但是给方法参数注入时，可以独立使用。

属性：value：指定bean的id。

##### 4.2.3 @Resource 

根据名称注入，属性name

##### 4.2.4 @Value

注入基本数据类型和String数据类型，value用于指定值

#### 4.3 用于改变作用范围的

相当于：`<bean id="" class="" scope="">`

##### 4.3.1 @Scope

作用：指定bean的作用范围

属性：value：指定范围的值（singleton，prototype……）

#### 4.4 和生命周期相关

相当于`<bean id="" class="" init-method="" destroy-method=""/>`

##### 4.4.1 @PostConstruct

指定初始化方法

##### 4.4.2 @PreDestroy

用于指定销毁方法

#### 4.5 更多注解

##### 4.5.1 @Configuration

用于指定当前类是一个spring配置类，当创建容器时会从该类上加载注解。获取容器时需要使用AnnotationApplicationContext(有@Configuration注解的类.class)。

属性： value：用于指定配置类的字节码

```java
@Configuration 
@ComponentScan("com.neu")
public class SpringConfiguration { 
}

把配置文件用类来代替
```

##### 4.5.2 @ComponentScan

指定Spring在初始化容器时要扫描的包，属性：basePackage：用于指定要扫描的包。

##### 4.5.3 @Bean

该注解只能写在方法上，表名使用此方法创建一个对象，并放入spring容器。

属性：name：给当前@Bean注解方法创建一个指定的名称（即bean的id）

使用该类对象时，可以直接用name代替。

##### 4.5.4 @PropertySource

用于加载.properties文件中的配置。例如我们配置数据源时，可以把连接数据库的信息写到properties配置文件中，就可以使用此注解指定properties配置文件的位置。

属性： value[]：用于指定properties文件位置。如果是在类路径下，需要写上classpath:

```
@Value("${jdbc.driver}")
private String driver;
```

##### 4.5.5 @Import

用于导入其他配置类，属性value()，用于指定其他配置类的字节码。

```java
@Configuration
@ComponentScan(basePackages="com.neu")
//将其他配置类与这个配置类合并
@Import({JdbcConfig.class})
public class SpringConfigruation{
    
}

@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig{
    
}
```

#### 4.6 通过注解获取容器

```java
ApplicationContext ac=
	new AnnotationConfigApplicationContext(SpringConfiguration.class);
```

### 5. Spring整合JUnit

1、使用@RunWith注解替换原有的运行器

```java
@RunWith(SpringJUnit4ClassRunner.class)
public class AccountServiceTest{

}
```

2、使用@ContextConfiguration指定spring配置文件的位置。

```java
@RunWith(SpringJUnit4ClassRunner.class) 
@ContextConfiguration(locations= {"classpath:bean.xml"}) 
public class AccountServiceTest { 
}
```

3、使用@Autowired给测试类中的变量注入数据

```java
@RunWith(SpringJUnit4ClassRunner.class) 
@ContextConfiguration(locations= {"classpath:bean.xml"}) 
public class AccountServiceTest { 
	@Autowired private IAccountService as ;
}
```

### 6. AOP

#### 6.1 AOP简介

作用：在程序运行期间，不修改源码对已有方法进行增强。

优势：减少重复代码，提高开发效率，维护方便。

实现方式：动态代理技术

Joinpoint(连接点)：指那些被拦截到的点，在Spring中指的是方法，Spring只支持方法类型的连接点。通过动态代理技术对这些方法添加额外的操作。

Pointcut(切入点)：指对连接点进行拦截的定义。

Advice(通知/增强)：指拦截到Joinpoint之后所要做的事情就是通知，通知的类型：前置通知、后置通知、异常通知、最终通知、环绕通知。

Introduction(引介)：引介是一种特殊的通知，在不修改代码的前提下，Introduction可以在运行期为类动态地添加一些方法或Field。

Target(目标对象)：代理的目标对象。

Weaving(织入)：是指把增强应用到目标对象来创建新的代理对象的过程。Spring采用动态代理织入。

Proxy(代理)：一个类AOP织入增强后，就产生一个结果代理类。

Aspect(切面)：切入点和通知的结合。

#### 6.2 步骤

1、抽取公共代码制作成通知。

2、把通知类用bean标签配置起来。

```xml
<bean id="txManager" class="com.neu.utils.TransactionManager">
	<property name="dbAssit" ref="dbAssit"></property>
</bean>
```

3、使用`aop:config`声明aop配置。

```xml
<aop:config>
	<!--配置的代码都写在此处-->
</aop:config>
```

4、使用`aop:aspect`配置切面。

```xml
<!--
	id:给切面提供唯一标志
	ref:引用配置好的通知类bean的id
-->
<aop:aspect id="txAdvice" ref="txManager">
	<!--配置通知的类型写在此处-->
</aop:aspect>
```

5、使用`aop:pointcut`配置切入点表达式。

```xml
作用：配置切入点表达式，就是指定对哪些类的方法进行增强。
属性：expression：用于定义切入点表达式；
	id：用于给切入点表达式提供一个唯一标识
<aop:pointcut expression="execution(
     public void com.neu.service.impl.AccountServiceImpl.transfer(
             java.lang.String,java.lang.String,java.lang.Float))"
              id="pt1"/>
```

6、使用`aop:xxx`配置对应的通知类型。

```xml
aop:before
	作用：用于配置前置通知，指定增强的方法在切入点之间执行。
	属性：method：用于指定通知类中的增强方法名称
		pointcut-ref：用于指定切入点表达式的引用
		pointcut:用于指定切入点表达式
<aop:before method="beginTransaction" pointcut-ref="pt1"/>

aop:after-returning
	作用：用于配置后置通知
	执行时间点：切入点方法正常执行之后，它和异常通知只能有一个执行
<aop:after-returning method="commit" pointcut-ref="pt1"/>

aop:after-throwing
	作用：用于配置异常通知
	执行时间点：切入点方法执行异常之后执行，它和后置通知只能执行一个
<aop:after-throwing method="rollback" pointcut-ref="pt1"/>

aop:after
	作用：用于配置最终通知
	执行时间点：无论切入点方法是否有异常，它都会在其后面执行
<aop:after method="release" pointcut-ref="pt1"/>
```

#### 6.3 切入点表达式说明

```
execution(表达式)
表达式语法：execution([修饰符] 返回值类型 包名.类名.方法名(参数))
写法说明：
	1.全匹配方式：
	2.访问修饰符可以省略
	3.返回值可以使用*号，表示任意返回值
	4.包名可以使用*号，表示任意包，但是有几级包，需要写几个*
	* *.*.*.*.AccountServiceImpl.saveAccount(com.itheima.domain.Account)
	5.使用..来表示当前包，及其子包
	* com..AccountServiceImpl.saveAccount(com.itheima.domain.Account)
	6.类名可以使用*号，表示任意类
	7.方法名可以使用*号，表示任意方法
	8.参数列表可以使用*，表示参数可以是任意数据类型，但是必须有参数
		* com..*.*(*)
	9.参数列表可以使用..表示有无参数均可，有参数可以是任意类型
		* com..*.*(..)
	10.全通配方式：
    	* *..*.*(..)
注：通常情况下，我们都是对业务层的方法进行增强，所以切入点表达式都是切到业务层实现类。 
execution(* com.itheima.service.impl.*.*(..))
```

#### 6.4 环绕通知

```xml
aop:around： 
	作用： 用于配置环绕通知 
	属性： method：指定通知中方法的名称。 
		pointct：定义切入点表达式 
		pointcut-ref：指定切入点表达式的引用 
	说明： 它是spring框架为我们提供的一种可以在代码中手动控制增强代码什么时候执行的方式。
	注意： 通常情况下，环绕通知都是独立使用的。
<aop:config> 
    <aop:pointcut expression="execution
         (* com.itheima.service.impl.*.*(..))" id="pt1"/> 
    <aop:aspect id="txAdvice" ref="txManager"> 
        <!-- 配置环绕通知 --> 
        <aop:around method="transactionAround" pointcut-ref="pt1"/> 			</aop:aspect> 
</aop:config>
```

```java
//spring框架为我们提供了一个接口：ProceedingJoinPoint，它可以作为环绕通知的方法参数。
//在环绕通知执行时，spring框架会为我们提供该接口的实现类对象，我们直接使用就行。
public Object transactionAround(ProceedingJoinPoint pjp) {
    //定义返回值 
    Object rtValue = null;
    try { 
        //获取方法执行所需的参数 
        Object[] args = pjp.getArgs(); 
        //前置通知：开启事务 
        beginTransaction(); 
        //执行方法 
        rtValue = pjp.proceed(args); 
        //后置通知：提交事务 
        commit();
        }catch(Throwable e) { 
        //异常通知：回滚事务 
        rollback(); 
        e.printStackTrace(); 
    }finally { 
        //最终通知：释放资源 
        release(); } 
    return rtValue;
}
```

#### 6.5 基于注解的AOP

1、在通知类上使用@Aspect注解声明为切面

2、在增强的方法上使用注解配置通知

```java
//开启事务 
//value:用于指定切入点表达式
@Before("execution(* com.itheima.service.impl.*.*(..))")
public void beginTransaction() { 
    try { 
        dbAssit.getCurrentConnection().setAutoCommit(false); 
    } catch (SQLException e) { 
        e.printStackTrace(); 
    } 
}
//后置通知
@AfterReturning
//异常通知
@AfterThrowing
//最终通知
@After
```

3、在spring配置文件中开启spring对注解AOP的支持

```xml
<!-- 开启spring对注解AOP的支持 --> 
<aop:aspectj-autoproxy/>
```

4、环绕通知注解配置

```java
@Around
作用： 把当前方法看成是环绕通知。
@Around("execution(* com.itheima.service.impl.*.*(..))") 
public Object transactionAround(ProceedingJoinPoint pjp) {
}
```

5、切入点表达式注解

```java
@Pointcut 
作用： 指定切入点表达式
@Pointcut("execution(* com.itheima.service.impl.*.*(..))") 
private void pt1() {}

//引用方式，注意括号
@Around("pt1()")
```

6、不使用XML配置

```java
@Configuration 
@ComponentScan(basePackages="com.itheima") 
@EnableAspectJAutoProxy 
public class SpringConfiguration { }
```

