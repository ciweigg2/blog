title: 【面试】Java面试题
date: 2019-07-16 15:33:57
tags: [java面试题]
categories: [综合]
---
### Spring

#### Spring Framework有哪些优点？

控制反转（IOC）： 使用反转控制技术在Spring中实现松耦合。对象提供依赖关系，而不是创建或查找依赖对象。

面向方面（AOP）： Spring支持面向方面的编程，并将应用程序业务逻辑与系统服务分开。

事务管理： Spring提供了一致的事务管理界面，可以缩小到本地事务并扩展到全局事务（JTA）。

<!--more-->

#### 什么是Spring IoC容器？

Spring IoC负责创建对象，管理它们（使用依赖注入（DI）），将它们连接在一起，配置它们，以及管理它们的整个生命周期。

#### IOC有什么好处？

IOC或依赖注入最小化应用程序中的代码量。它使测试应用程序变得容易，因为在单元测试中不需要单例或JNDI查找机制。以最小的努力和最少的侵入机制促进松散耦合。IOC容器支持急切的实例化和延迟加载服务。

#### Bean Factory和ApplicationContext有什么区别？

ApplicationContex提供了一种解析文本消息的方法，一种加载文件资源（如图像）的通用方法，它们可以将事件发布到注册为侦听器的bean。此外，可以在应用程序上下文中以声明方式处理容器中的容器或容器上的操作，这些操作必须以编程方式与Bean Factory一起处理。ApplicationContex实现MessageSource，一个用于获取本地化消息的接口，实际的实现是可插入的。

#### 有哪些不同类型的IoC（依赖注入）？

基于构造函数的依赖注入：当容器调用具有许多参数的类构造函数时，完成基于构造函数的DI，每个参数表示对其他类的依赖。 基于Setter的依赖注入：基于Setter的DI是在调用无参数构造函数或无参数静态工厂方法来实例化bean之后，通过容器调用bean上的setter方法来完成的。

#### Spring bean是什么？

Spring Beans是构成Spring应用程序主干的Java对象。它们由Spring IoC容器实例化，组装和管理。这些bean是使用提供给容器的配置元数据创建的，例如，以XML定义的形式。

在spring框架中定义的bean是singleton bean。如果指定为true，则bean标记中有一个名为“singleton”的属性，然后bean变为singleton，如果设置为false，则bean将成为原型bean。默认情况下，它设置为true。因此，spring框架中的所有bean都是默认的单例bean。

#### 如何为Spring容器提供配置元数据？

为Spring容器提供配置元数据有三种重要方法：

- 基于XML的配置文件。

- 基于注释的配置。

- 基于Java的配置。

#### 如何定义bean的范围？

在Spring中定义一个时，我们也可以为bean声明一个范围。它可以通过bean定义中的scope属性定义。例如，当Spring每次需要生成一个新的bean实例时，bean'sscope属性就是原型。另一方面，当每次需要Spring都必须返回相同的bean实例时，bean scope属性必须设置为singleton。

#### Spring Framework中的Singleton bean线程安全吗？

不，单例bean在Spring框架中不是线程安全的。

#### 解释Spring框架中的Bean生命周期

spring容器从XML文件中查找bean的定义并实例化bean。

Spring填充bean定义（DI）中指定的所有属性。

如果bean实现了StringNameAware接口，则spring将bean的id传递给setBeanName（）

如果Bean implementsBeanFactoryAware接口，spring将beanfactory传递给setBeanFactory（）

如果有任何与bean关联的beanBeanPostProcessors，则Spring调用postProcesserBeforeInitialization（）

如果bean implementsIntializingBean，则调用其afterPropertySet（）方法。

如果bean具有init方法声明，则调用指定的初始化方法。

如果有任何与Bean关联的BeanPostProcessors，则将调用它们的postProcessAfterInitialization（）方法。

如果bean实现了DisposableBean，它将调用destroy（）

#### 哪些是重要的bean生命周期方法？可以覆盖它们吗？

bean标记有两个重要的属性（init-method和destroy-method），您可以使用它们定义自己的自定义初始化和销毁方法。还有相应的注释（@PostConstruct和@PreDestroy）。

#### spring中用到的设计模式

1. 第一种：简单工厂

又叫做静态工厂方法（StaticFactory Method）模式，但不属于23种GOF设计模式之一。

简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。

spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。如下配置，就是在 HelloItxxz 类中创建一个 itxxzBean。

```java
<beans>
<bean id="singletonBean" class="com.itxxz.HelloItxxz">
   <constructor-arg>
       <value>Hello! 这是singletonBean!value>
   </constructor-arg>
  </ bean>

<bean id="itxxzBean" class="com.itxxz.HelloItxxz"
   singleton="false">
   <constructor-arg>
       <value>Hello! 这是itxxzBean! value>
   </constructor-arg>
</bean>

</beans>
```

2. 第二种：工厂方法

通常由应用程序直接使用new创建新的对象，为了将对象的创建和使用相分离，采用工厂模式,即应用程序将对象的创建及初始化职责交给工厂对象。

一般情况下,应用程序有自己的工厂对象来创建bean.如果将应用程序自己的工厂对象交给Spring管理,那么Spring管理的就不是普通的bean,而是工厂Bean。

就以工厂方法中的静态方法为例讲解一下：

```java
import java.util.Random;
public class StaticFactoryBean {
 public static Integer createRandom() {
      return new Integer(new Random().nextInt());
  }
}
```

建一个config.xm配置文件，将其纳入Spring容器来管理,需要通过factory-method指定静态方法名称

```java
<bean id="random"
class="example.chapter3.StaticFactoryBean" factory-method="createRandom" 
//createRandom方法必须是static的,才能找到 scope="prototype"
/>
```

测试:

```java
public static void main(String[] args) {
//调用getBean()时,返回随机数.如果没有指定factory-method,会返回StaticFactoryBean的实例,即返回工厂Bean的实例       
XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("config.xml"));       
System.out.println("我是IT学习者创建的实例:"+factory.getBean("random").toString());
}
```

3. 第三种：单例模式（Singleton）

保证一个类仅有一个实例，并提供一个访问它的全局访问点。  

spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory。但没有从构造器级别去控制单例，这是因为spring管理的是是任意的java对象。

核心提示点：Spring下默认的bean均为singleton，可以通过singleton=“true|false” 或者 scope=“？”来指定

4. 第四种：适配器（Adapter）

在Spring的Aop中，使用的Advice（通知）来增强被代理类的功能。Spring实现这一AOP功能的原理就使用代理模式（1、JDK动态代理。2、CGLib字节码生成技术代理。）对类进行方法级别的切面增强，即，生成被代理类的代理类， 并在代理类的方法前，设置拦截器，通过执行拦截器重的内容增强了代理方法的功能，实现的面向切面编程。

5. 第五种：包装器

多数据源的实现

6. 第六种：代理（Proxy）

为其他对象提供一种代理以控制对这个对象的访问。 从结构上来看和Decorator模式类似，但Proxy是控制，更像是一种对功能的限制，而Decorator是增加职责。  

spring的Proxy模式在aop中有体现，比如JdkDynamicAopProxy和Cglib2AopProxy。

7. 第七种：观察者（Observer）

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。  

spring中Observer模式常用的地方是listener的实现。如ApplicationListener。

8. 第八种：策略（Strategy）

定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换。本模式使得算法可独立于使用它的客户而变化。

spring中在实例化对象的时候用到Strategy模式

SimpleInstantiationStrategy中有实现的

9. 第九种：模板方法

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Template Method使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤

Template Method模式一般是需要继承的。这里想要探讨另一种对Template Method的理解。spring中的JdbcTemplate，在用这个类时并不想去继承这个类，因为这个类的方法太多，但是我们还是想用到JdbcTemplate已有的稳定的、公用的数据库连接，那么我们怎么办呢？我们可以把变化的东西抽出来作为一个参数传入JdbcTemplate的方法中。但是变化的东西是一段代码，而且这段代码会用到JdbcTemplate中的变量。怎么办？那我们就用回调对象吧。在这个回调对象中定义一个操纵JdbcTemplate中变量的方法，我们去实现这个方法，就把变化的东西集中到这里了。然后我们再传入这个回调对象到JdbcTemplate，从而完成了调用。这可能是Template Method不需要继承的另一种实现方式。

#### springmvc执行流程

Http 请求 到 DispatcherServlet
(1) 客户端请求提交到 DispatcherServlet。
HandlerMapping 寻找处理器
(2) 由 DispatcherServlet 控制器查询一个或多个 HandlerMapping，找到处理请求的
Controller。
调用处理器 Controller
(3) DispatcherServlet 将请求提交到 Controller。

开始执行Handler（Controller)。在填充Handler的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作:

- HttpMessageConveter：将请求消息（如 Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息。
- 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等。
- 数据根式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。
- 数据验证：验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中。

Controller 调用业务逻辑处理后，返回 ModelAndView
(4)(5)调用业务处理和返回结果：Controller 调用业务逻辑处理后，返回 ModelAndView。
DispatcherServlet 查询 ModelAndView
(6)(7)处理视图映射并返回模型： DispatcherServlet 查询一个或多个 ViewResoler 视图解析器，
找到 ModelAndView 指定的视图。
ModelAndView 反馈浏览器 HTTP
(8) Http 响应：视图负责将结果显示到客户端。

#### @Autowired与@Resource的区别

@Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上。

@Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用

@Resource（这个注解属于J2EE的），默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名进行安装名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。

#### spring事务的传播属性

REQUIRED，SUPPORTS，MANDATORY，REQUIRES_NEW，NOT_SUPPORTED，NEVER，NESTED

**REQUIRED**  

强制要求要有一个物理事务。如果没有已经存在的事务，就专门打开一个事务用于当前范围

**REQUIRES_NEW**  

与REQUIRED相比，总是使用一个独立的物理事务用于每一个受影响的逻辑事务范围，从来不参与到一个已存在的外围事务范围。

**NESTED**  

使用同一个物理事务，带有多个保存点，可以回滚到这些保存点，可以认为是部分回滚，这样一个内部事务范围触发了一个回滚，外围事务能够继续这个物理事务，尽管有一些操作已经被回滚。典型地，它对应于JDBC的保存点，所以只对JDBC事务资源起作用。

**SUPPORTS**  

支持当前事务。如果当前有事务，就参与进来，如果没有，就以非事务的方式运行。这样的一个逻辑事务范围，它背后可能没有实际的物理事务，此时的事务也成为空事务。  

**NOT_SUPPORTED**  

不支持当前事务。总是以非事务方式运行。当前的事务会被挂起，并在适合的时候恢复。  

**MANDATORY**  

支持当前事务。如果当前没有事务存在，就抛出异常。  

**NEVER**  

不支持当前事务。如果当前有事务存在，就抛出异常。

#### spring事务的隔离级别

Serializable：最严格的级别，事务串行执行，资源消耗最大

REPEATABLE_READ：保证了一个事务不会修改已经由另一个事务读取但未提交（回滚）的数据。避免了“脏读取”和“不可重复读取”的情况，但是带来了更多的性能损失。

READ_COMMITTED:大多数主流数据库的默认事务等级，保证了一个事务不会读到另一个并行事务已修改但未提交的数据，避免了“脏读取”。该级别适用于大多数系统。

Read_Uncommitted：保证了读取过程中不会读取到非法数据。隔离级别在于处理多事务的并发问题。

### Mybatis

#### {}和${}的区别是什么？

{}是预编译处理，${}是字符串替换。

Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；

Mybatis在处理${}时，就是把${}替换成变量的值。

使用#{}可以有效的防止SQL注入，提高系统安全性。

#### Mybatis的编程步骤?

创建SqlSessionFactory 
通过SqlSessionFactory创建SqlSession 
通过sqlsession执行数据库操作 
调用session.commit()提交事务 
调用session.close()关闭会话

#### Mybatis在核心处理类叫什么?

SqlSession

#### Mybatis 是如何进行分页的？分页插件的原理是什么?

1）Mybatis 使用 RowBounds 对象进行分页，也可以直接编写 sql 实现分页，也可以使用
Mybatis 的分页插件。
2）分页插件的原理：实现 Mybatis 提供的接口，实现自定义插件，在插件的拦截方法内拦
截待执行的 sql，然后重写 sql。
举例：select * from student，拦截 sql 后重写为：select t.* from （select * from student）t
limit 0，10

#### 简述 Mybatis 的插件运行原理，以及如何编写一个插件？

1）Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、
Executor 这 4 种接口的插件，Mybatis 通过动态代理，为需要拦截的接口生成代理对象以实
现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是
InvocationHandler 的 invoke()方法，当然，只会拦截那些你指定需要拦截的方法。
2）实现 Mybatis 的 Interceptor 接口并复写 intercept()方法，然后在给插件编写注解，指定
要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件

#### Mybatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？

1）Mybatis 动态 sql 可以让我们在 Xml 映射文件内，以标签的形式编写动态 sql，完成逻辑
判断和动态拼接 sql 的功能。
2）Mybatis 提供了 9 种动态 sql 标签：
trim|where|set|foreach|if|choose|when|otherwise|bind。
3）其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼
接 sql，以此来完成动态 sql 的功能

#### resultType resultMap 的区别？

1）类的名字和数据库相同时，可以直接设置 resultType 参数为 Pojo 类
2）若不同，需要设置 resultMap 将结果名字和 Pojo 名字进行转换

### Java

#### Java一些字节字符区别

##### char,byte,short,int,long,float,double分别是几字节的

byte 8位 1个字节
char 16位  2个字节
short 16位 2个字节
int 32位   4个字节
float 32位 4个字节
double 64位8个字节
long 64位  8个字节

##### UTF-8和GBK的区别

GBK就是在保存你的帖子的时候，一个汉字占用两个字节。外国人看会出现乱码，此为我中华为自己汉字编码而形成之解决方案。

UTF8就是在保存你的帖子的时候，一个汉字占用3个字节。但是外国人看的话不会乱码，此为西人为了解决多字节字符而形成之解决方案。

#### Java基础

##### 面向对象

###### Java面向对象的特征

``封装``

封装，也就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。

``继承``

减少代码重复、臃肿，提高代码可维护性。

``多态``

多态就是同一个接口，使用不同的实现，而执行不同的操作。

###### 谈谈final、finally、finalize的区别

``final``

根据程序上下文环境，Java关键字final有“这是无法改变的”或者“终态的”含义，它可以修饰非抽象类、非抽象类成员方法和变量。你可能出于两种理解而需要阻止改变：设计或效率。

final类不能被继承，没有子类，final类中的方法默认是final的。

final方法不能被子类的方法覆盖，但可以被继承。

final成员变量表示常量，只能被赋值一次，赋值后值不再改变。

final不能用于修饰构造方法。

``finally``

finally是关键字，在异常处理中，try子句中执行需要运行的内容，catch子句用于捕获异常，finally子句表示不管是否发生异常，都会执行。finally可有可无。但是try...catch必须成对出现。

``finalize()``

finalize() 方法名，Object类的方法，Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象进行调用。finalize()方法是在垃圾收集器删除对象之前对这个对象调用的子类覆盖 finalize() 方法以整理系统资源或者执行其他清理操作。

###### Java 中的 ==, equals 与 hashCode 的区别与联系

``对于关系操作符 ==``

若操作数的类型是基本数据类型，则该关系操作符判断的是左右两边操作数的值是否相等

若操作数的类型是引用数据类型，则该关系操作符判断的是左右两边操作数的内存地址是否相同。也就是说，若此时返回true,则该操作符作用的一定是同一个对象。

``equals``

equals()是比较对象的内容

``hashCode``

是一个本地方法，返回的对象的地址值。

###### 重载与重写的区别

``重写方法的规则：``

1、参数列表必须完全与被重写的方法相同，否则不能称其为重写而是重载

2、返回的类型必须一直与被重写的方法的返回类型相同，否则不能称其为重写而是重载

3、访问修饰符的限制一定要大于被重写方法的访问修饰符（public>protected>default>private）

4、重写方法一定不能抛出新的检查异常或者比被重写方法申明更加宽泛的检查型异常。例如： 父类的一个方法申明了一个检查异常IOException，在重写这个方法是就不能抛出Exception,只能抛出IOException的子类异常，可以抛出非检查异常

``重载的规则：``

1、必须具有不同的参数列表；

2、可以有不责骂的返回类型，只要参数列表不同就可以了；

3、可以有不同的访问修饰符；

4、可以抛出不同的异常；

``重写与重载的区别在于：``

重写多态性起作用，对调用被重载过的方法可以大大减少代码的输入量，同一个方法名只要往里面传递不同的参数就可以拥有不同的功能或返回值

用好重写和重载可以设计一个结构清晰而简洁的类，可以说重写和重载在编写代码过程中的作用非同一般

### 多线程

#### 在java中守护线程和本地线程区别？

java中的线程分为两种：守护线程（Daemon）和用户线程（User）。

任何线程都可以设置为守护线程和用户线程，通过方法Thread.setDaemon(bool on)；true则把该线程设置为守护线程，反之则为用户线程。Thread.setDaemon()必须在Thread.start()之前调用，否则运行时会抛出异常。

两者的区别：

虚拟机（JVM）何时离开，Daemon是为其他线程提供服务，如果全部的User Thread已经撤离，Daemon 没有可服务的线程，JVM撤离。也可以理解为守护线程是JVM自动创建的线程（但不一定），用户线程是程序创建的线程；比如JVM的垃圾回收线程是一个守护线程，当所有线程已经撤离，不再产生垃圾，守护线程自然就没事可干了，当垃圾回收线程是Java虚拟机上仅剩的线程时，Java虚拟机会自动离开。

扩展：Thread Dump打印出来的线程信息，含有daemon字样的线程即为守护进程，可能会有：服务守护进程、编译守护进程、windows下的监听Ctrl+break的守护进程、Finalizer守护进程、引用处理守护进程、GC守护进程。

#### 线程与进程的区别？

进程是操作系统分配资源的最小单元，线程是操作系统调度的最小单元。

一个程序至少有一个进程,一个进程至少有一个线程。

#### 什么是多线程中的上下文切换？

多线程会共同使用一组计算机上的CPU，而线程数大于给程序分配的CPU数量时，为了让各个线程都有执行的机会，就需要轮转使用CPU。不同的线程切换使用CPU发生的切换数据等就是上下文切换。

#### 在Java中Executor和Executors的区别？

Executors 工具类的不同方法按照我们的需求创建了不同的线程池，来满足业务的需求。  

Executor 接口对象能执行我们的线程任务。  

ExecutorService接口继承了Executor接口并进行了扩展，提供了更多的方法我们能获得任务执行的状态并且可以获取任务的返回值。  

使用ThreadPoolExecutor 可以创建自定义线程池。  

Future 表示异步计算的结果，他提供了检查计算是否完成的方法，以等待计算的完成，并可以使用get()方法获取计算的结果。

#### 什么是Callable和Future?

Callable接口类似于Runnable，但是Runnable不会返回结果，并且无法抛出返回结果的异常，而Callable功能更强大一些，被线程执行后，可以返回值，这个返回值可以被Future拿到，也就是说Future可以拿到异步执行任务的返回值。  

可以认为是带有回调的Runnable。

Future接口表示异步任务，是还没有完成的任务给出的未来结果。所以说Callable用于产生结果，Future用于获取结果。

#### 什么是并发容器的实现？

何为同步容器：可以简单地理解为通过synchronized来实现同步的容器，如果有多个线程调用同步容器的方法，它们将会串行执行。比如Vector、Hashtable、Collections.synchronizedSet、synchronizedList等方法返回的容器。  

可以通过查看Vector，Hashtable等这些同步容器的实现代码，可以看到这些容器实现线程安全的方式就是将它们的状态封装起来，并在需要同步的方法上加上关键字synchronized。

并发容器使用了与同步容器完全不同的加锁策略来提供更高的并发性和伸缩性，例如在ConcurrentHashMap中采用了一种粒度更细的加锁机制，可以称为分段锁，在这种锁机制下，允许任意数量的读线程并发地访问map，并且执行读操作的线程和写操作的线程也可以并发的访问map，同时允许一定数量的写操作线程并发地修改map，所以它可以在并发环境下实现更高的吞吐量。

#### 为什么我们调用start()方法时会执行run()方法，为什么我们不能直接调用run()方法？

当你调用start()方法时你将创建新的线程，并且执行在run()方法里的代码。  

但是如果你直接调用run()方法，它不会创建新的线程也不会执行调用线程的代码，只会把run方法当作普通方法去执行。

#### Java中你怎样唤醒一个阻塞的线程？

wait、notify形式通过一个object作为信号，object的wait()方法是锁门的动作，notify()、notifyAll()是开门的动作，某一线程一旦关上门后其他线程都将阻塞，直到别的线程打开门。notify()准许阻塞的一个线程通过，notifyAll()允许所有线程通过。如下例子：主线程分别启动两个线程，随后通知子线程暂停等待，再逐个唤醒后线程抛异常退出。

#### 什么是多线程中的上下文切换？

在上下文切换过程中，CPU会停止处理当前运行的程序，并保存当前程序运行的具体位置以便之后继续运行。

从这个角度来看，上下文切换有点像我们同时阅读几本书，在来回切换书本的同时我们需要记住每本书当前读到的页码。在程序中，上下文切换过程中的“页码”信息是保存在进程控制块（PCB）中的。PCB还经常被称作“切换桢”，“页码”信息会一直保存到CPU的内存中，直到他们被再次使用。

上下文切换是存储和恢复CPU状态的过程，它使得线程执行能够从中断点恢复执行。上下文切换是多任务操作系统和多线程环境的基本特征。

#### notify()和notifyAll()有什么区别？

当一个线程进入wait之后，就必须等其他线程notify/notifyall,使用notifyall,可以唤醒所有处于wait状态的线程，使其重新进入锁的争夺队列中，而notify只能唤醒一个。

如果没把握，建议notifyAll，防止notigy因为信号丢失而造成程序异常。

#### 乐观锁和悲观锁的理解及如何实现？

悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如Java里面的同步原语synchronized关键字的实现也是悲观锁。  

乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

#### SynchronizedMap 和 ConcurrentHashMap有什么区别？

SynchronizedMap一次锁住整张表来保证线程安全，所以每次只能有一个线程来访为map。

ConcurrentHashMap使用分段锁来保证在多线程下的性能。ConcurrentHashMap中则是一次锁住一个桶。ConcurrentHashMap默认将hash表分为16个桶，诸如get,put,remove等常用操作只锁当前需要用到的桶。这样，原来只能一个线程进入，现在却能同时有16个写线程执行，并发性能的提升是显而易见的。  

另外ConcurrentHashMap使用了一种不同的迭代方式。在这种迭代方式中，当iterator被创建后集合再发生改变就不再是抛出ConcurrentModificationException，取而代之的是在改变时new新的数据从而不影响原有的数据 ，iterator完成后再将头指针替换为新的数据 ，这样iterator线程可以使用原来老的数据，而写线程也可以并发的完成改变。

#### 什么叫线程安全？servlet是线程安全吗?

线程安全是编程中的术语，指某个函数、函数库在多线程环境中被调用时，能够正确地处理多个线程之间的共享变量，使程序功能正确完成。

Servlet不是线程安全的，servlet是单实例多线程的，当多个线程同时访问同一个方法，是不能保证共享变量的线程安全性的。  

Struts2的action是多实例多线程的，是线程安全的，每个请求过来都会new一个新的action分配给这个请求，请求完成后销毁。  

SpringMVC的Controller是线程安全的吗？不是的，和Servlet类似的处理流程

Struts2好处是不用考虑线程安全问题；Servlet和SpringMVC需要考虑线程安全问题，但是性能可以提升不用处理太多的gc，可以使用ThreadLocal来处理多线程的问题。

### SpringBoot

#### Spring Boot有哪些优点？

减少开发，测试时间和努力。

使用JavaConfig有助于避免使用XML。

避免大量的Maven导入和各种版本冲突。

提供意见发展方法。

通过提供默认值快速开始开发。

没有单独的Web服务器需要。这意味着你不再需要启动Tomcat，Glassfish或其他任何东西。

需要更少的配置 因为没有web.xml文件。只需添加用@ Configuration注释的类，然后添加用@Bean注释的方法，Spring将自动加载对象并像以前一样对其进行管理。您甚至可以将@Autowired添加到bean方法中，以使Spring自动装入需要的依赖关系中。

基于环境的配置 使用这些属性，您可以将您正在使用的环境传递到应用程序：-Dspring.profiles.active = {enviornment}。在加载主应用程序属性文件后，Spring将在（application{environment} .properties）中加载后续的应用程序属性文件。

#### 什么是JavaConfig？

JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。

#### 如何重新加载Spring Boot上的更改，而无需重新启动服务器？

Spring Boot有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

#### Spring Boot中的监视器是什么？

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。
