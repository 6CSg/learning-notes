# Spring——IOC（控制反转）

## 1.谈谈自己对于 Spring IoC 的了解？

**IoC（Inverse of Control:控制反转）** 是一种设计思想，而不是一个具体的技术实现。IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。不过， IoC 并非 Spring 特有，在其他语言中也有应用。

**为什么叫控制反转？**

- **控制** ：指的是对象创建（实例化、管理）的权力
- **反转** ：控制权交给外部环境（Spring 框架、IoC 容器）

## 2.什么是 Spring Bean？

简单来说，Bean 代指的就是那些被 **IoC 容器所管理的对象**。

我们需要告诉 IoC 容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。**配置元数据可以是 XML 文件、注解或者 Java 配置类。**

### 3.IoC和DI是什么关系

**控制反转是通过依赖注入实现的**，通俗来说就是**IoC是设计思想，DI是实现方式**。

DI—Dependency Injection，即依赖注入：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

我们来深入分析一下：

- **谁依赖于谁**？

当然是应用程序依赖于IoC容器；

- **为什么需要依赖**？

应用程序需要IoC容器来提供对象需要的外部资源；

- **谁注入谁**？

很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；

- **注入了什么**？

就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。

## 3. IOC 配置的三种方式

### I.XML配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-bean   s.xsd">
    <!--外部bean的操作-->
<!--先创建Service 和 Dao对象-->
    <bean id="usrService" class="com.csg.spring5.service.UserService">
        <!--给userService对象注入Dao属性（注入Dao对象）-->
        <!--ref映射UserDao的bean对象的id,即把外部bean注入-->
        <property name="userDao" ref="userDaoImpl"></property>
    </bean>
    <bean id="userDaoImpl" class="com.csg.spring5.dao.UserDaoImpl"></bean>
</beans>
```

- **优点**： 可以使用于任何场景，结构清晰，通俗易懂
- **缺点**： 配置繁琐，不易维护，枯燥无味，扩展性差

### II.JavaConfig

将类的创建交给我们配置的JavcConfig类来完成，Spring只负责维护和管理，采用纯Java创建方式。其本质上就是把在XML上的配置声明转移到Java配置类中

- **优点**：适用于任何场景，配置方便，因为是纯Java代码，扩展性高，十分灵活
- **缺点**：由于是采用Java类的方式，声明不明显，如果大量配置，可读性比较差

**举例**：

1. 创建一个配置类， 添加@Configuration注解声明为配置类
2. 创建方法，方法上加上@bean，该方法用于创建实例并返回，该实例创建后会交给spring管理，方法名建议与实例名相同（首字母小写）。注：实例类不需要加任何注解

```java
@Configuration
public class BeansConfig {

    /**
     * @return user dao
     */
    @Bean("userDao")
    public UserDaoImpl userDao() {
        return new UserDaoImpl();
    }

    /**
     * @return user service
     */
    @Bean("userService")
    public UserServiceImpl userService() {
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDao(userDao());
        return userService;
    }
}
```

### III.注解配置

通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，**前提是需要先配置Spring的注解扫描器@ComponentScan。**

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面

## 4.依赖注入的三种方式

**常用的注入方式主要有三种：**

- 构造方法注入（Construct注入）
- setter注入
- 基于注解的注入（接口注入）

### I.setter方式

- **在XML配置方式中**，property都是setter方式注入，比如下面的xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-bean   s.xsd">
    <!--外部bean的操作-->
<!--先创建Service 和 Dao对象-->
    <bean id="usrService" class="com.csg.spring5.service.UserService">
        <!--给userService对象注入Dao属性（注入Dao对象）-->
        <!--ref映射UserDao的bean对象的id,即把外部bean注入-->
        <property name="userDao" ref="userDaoImpl"></property>
    </bean>
    <bean id="userDaoImpl" class="com.csg.spring5.dao.UserDaoImpl"></bean>
</beans>
```

本质上包含两步：

1. 第一步，需要new UserServiceImpl()创建对象, 所以**需要默认构造函数**
2. 第二步，调用setUserDao()函数注入userDao的值, 所以需要setDept()函数

所以对应的service类是这样的：

```java
package com.csg.spring5.bean;

public class Emp {
    private String eName;
    private String gender;
    //员工属于某一个部门，用一个Dept对象来表示
    private Dept dept;

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    public void seteName(String eName) {
        this.eName = eName;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public void getEmp(){
        System.out.println("姓名："+eName);
        System.out.println("姓别："+gender);
        System.out.println("部门："+dept.getdName());
    }
}
```

### II.构造器注入

- **在XML配置方式中**，`<constructor-arg>`是通过构造函数参数注入，比如下面的xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <constructor-arg name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```

本质上是new UserServiceImpl(userDao)创建对象, 所以对应的service类是这样的：

```java
/**
 * @author pdai
 */
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private final UserDaoImpl userDao;

    /**
     * init.
     * @param userDaoImpl user dao impl
     */
    public UserServiceImpl(UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return this.userDao.findUserList();
    }

}  
```

- **在注解和Java配置方式下**

```java
/**
 * @author pdai
 */
 @Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private final UserDaoImpl userDao;

    /**
     * init.
     * @param userDaoImpl user dao impl
     */
    @Autowired // 这里@Autowired也可以省略
    public UserServiceImpl(final UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return this.userDao.findUserList();
    }

}
```

### III.注解注入

以@Autowired（自动注入）注解注入为例，修饰符有三个属性：**Constructor，byType，byName**,**默认按照byType注入**。

- **constructor**：通过构造方法进行自动注入，spring会匹配与构造方法参数类型一致的bean进行注入，如果有一个多参数的构造方法，一个只有一个参数的构造方法，在容器中查找到多个匹配多参数构造方法的bean，那么spring会优先将bean注入到多参数的构造方法中。
- **byName**：被注入bean的id名必须与set方法后半截匹配，并且id名称的第一个单词首字母必须小写，这一点与手动set注入有点不同。
- **byType**：查找所有的set方法，将符合符合参数类型的bean注入。

## 5.@Autowired和@Resource注入有何区别？

### I.@Autowired

`Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入 Bean （接口的实现类）。

**这会有什么问题呢？** **当一个接口存在多个实现类的话**，`byType`这种方式就无法正确注入对象了，因为这个时候 Spring 会同时找到多个满足条件的选择，默认情况下它自己不知道选择哪一个。

这种情况下，注入方式会变为 `byName`（根据名称进行匹配），这个名称通常就是类名（首字母小写）。就比如说下面代码中的 `smsService` 就是我这里所说的名称。

```java
// smsService 就是我们上面所说的名称
@Autowired
private SmsService smsService;
```

**bean的name是写在XML中或@Service @Bean @Component上的**

```xml
<bean id="userDaoImpl" class="com.csg.spring5.dao.UserDaoImpl"></bean> <!-id就是bean的name->
```

```java
 @Bean(value = "basicAuthRequestInterceptor") //value默认就是类名首字母小写
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new BasicAuthRequestInterceptor("user", "password123");
    }
```

举个例子，`SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;

// 正确注入 SmsServiceImpl1 对象对应的 bean，虽然根据类型无法注入，但只会会根据名称注入，变量名smsServiceImpl会自动匹配到SmsServiceImpl1这个类名的首字母小写
@Autowired
private SmsService smsServiceImpl1;

// 正确注入  SmsServiceImpl1 对象对应的 bean
// value对应的smsServiceImpl1 就是我们上面所说的名称，因为它会自动匹配到SmsServiceImpl1这个类的默认名称
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

### II.@Resource

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。

`@Resource` 有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）。

```java
public @interface Resource {
    String name() default "";
    Class<?> type() default Object.class;
}
```

如果仅指定 `name` 属性则注入方式为`byName`，如果仅指定`type`属性则注入方式为`byType`，如果同时指定`name` 和`type`属性（不建议这么做）则注入方式为`byType`+`byName`。

```java
// 报错，byName 和 byType 都无法匹配到 bean
//byType匹配不到是因为有两个实现类，不知道要注入哪个类;
//byName匹配不到是因为该接口的实现类没有叫smsService的
@Resource
private SmsService smsService;

// 正确注入 SmsServiceImpl1 对象对应的 bean, name就是我们定义的变量名smsServiceImpl1
@Resource
private SmsService smsServiceImpl1;

// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）,name就是我们注解上声明的
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显示指定名称，`@Resource`可以通过 `name` 属性来显示指定名称，如果不显示指定，要注入的属性的变量名就会被当作名称。

指定名称后，如果Spring IOC容器中没有对应的组件bean抛出NoSuchBeanDefinitionException。也可以将@Autowired中required配置为false，如果配置为false之后，当没有找到相应bean的时候，系统不会抛异常。

## 6.Bean 的作用域有哪些?

Spring 中 Bean 的作用域通常有下面几种：

- **singleton** : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。
- **prototype** : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例。
- **request** （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
- **session** （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
- **application/global-session** （仅 Web 应用可用）： 每个 Web 应用在启动时创建一个 Bean（应用 Bean），，该 bean 仅在当前应用启动时间内有效。
- **websocket** （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean。

**可以在用@Scope注解指定Bean的作用域**：

```java
@Service
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class UserService {
    //创建UserDao属性，生成set方法
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add(){
        System.out.println("This is service add...");
        userDao.update();
    }
}
```

##  7.单例 Bean 的线程安全问题了解吗？

大部分时候我们并没有在项目中使用多线程，所以很少有人会关注这个问题。单例 Bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。

常见的有两种解决办法：

1. 在 Bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，大部分 Bean 实际都是无状态（没有实例变量）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。

不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了。

## 8.IOC容器初始化流程

![img](https://pdai.tech/_images/spring/springframework/spring-framework-ioc-source-9.png)

1. Spring IoC容器对Bean定义资源的载入是从refresh()函数开始的，refresh()是一个模板方法，refresh()方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。
2. 通过 ResourceLoader 来完成资源文件位置的定位
3. 通过 BeanDefinitionReader来完成定义信息的解析和 Bean 信息的注册, 往往使用的是XmlBeanDefinitionReader 来解析 bean 的 xml 定义文件 - 实际的处理过程是委托给 BeanDefinitionParserDelegate 来完成的，从而得到 bean 的定义信息，这些信息在 Spring 中使用 BeanDefinition 对象来表示 
4. 容器解析得到 BeanDefinition 以后，需要把它在 IOC 容器中注册，这由 IOC 实现 BeanDefinitionRegistry 接口来实现。注册过程就是在 IOC 容器内部维护的一个beanDefinitionMap来**保存得到的 BeanDefinition 的过程**。这个 beanDefinitionMap是 IoC 容器**持有 bean 信息的场所**，以后对 bean 的操作都是围绕这个beanDefinitionMap来实现的，beanDefinitionMap本质上是一个`ConcurrentHashMap<String, Object>`；并且BeanDefinition接口中包含了这个类的Class信息以及是否是单例等；

## **9、Spring Bean的生命周期**

Spring Bean的生命周期只有**四个阶段**：**实例化 Instantiation --> 属性赋值 Populate --> 初始化 Initialization --> 销毁 Destruction**

![img](https://img-blog.csdnimg.cn/img_convert/84341632e9df3625a91c3e2a1437ee65.png)

**（1）实例化Bean：**

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。

对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean

**（2）设置对象属性（依赖注入）：**

实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成属性设置与依赖注入。

**（3）处理Aware接口：**

**Spring会检测该对象是否实现了xxxAware接口**，通过Aware类型的接口，可以让我们拿到Spring容器的一些资源：

- 如果这个Bean实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，传入Bean的名字；
- 如果这个Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
- 如果这个Bean实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。
- 如果这个Bean实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

**（4）BeanPostProcessor前置处理：**

如果想对Bean进**行一些自定义的前置处理**，那么可以让**Bean实现了BeanPostProcessor接口**，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

**（5）InitializingBean：**

如果Bean**实现了InitializingBean接口**，**执行afeterPropertiesSet()方**法。

**（6）init-method：**

如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法。

**（7）BeanPostProcessor后置处理：**

如果这个Bean**实现了BeanPostProcessor接口**，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

**（8）DisposableBean：**

当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

**（9）destroy-method：**

最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

## 10.Spring如何解决循环依赖问题

**1、什么是循环依赖：**

类与类之间的依赖关系形成了闭环，就会导致循环依赖问题的产生。

```java
public class ClassA {
	private ClassB classB;
 
	public ClassB getClassB() {
		return classB;
	}
 
	public void setClassB(ClassB classB) {
		this.classB = classB;
	}
}
public class ClassB {
	private ClassA classA;
 
	public ClassA getClassA() {
		return classA;
	}
 
	public void setClassA(ClassA classA) {
		this.classA = classA;
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
	<bean id="classA" class="ioc.cd.ClassA">
		<property name="classB" ref="classB"></property>
	</bean>
	<bean id="classB" class="ioc.cd.ClassB">
		<property name="classA" ref="classA"></property>
	</bean>
</beans>
```

```java
	@Test
	public void test() throws Exception {
		// 创建IoC容器，并进行初始化
		String resource = "spring/spring-ioc-circular-dependency.xml";
		ApplicationContext context = new ClassPathXmlApplicationContext(resource);
		// 获取ClassA的实例（此时会发生循环依赖）
		ClassA classA = (ClassA) context.getBean(ClassA.class);
	}
```

<img src="https://img-blog.csdnimg.cn/img_convert/d9f570aa0dce404d7da35ecf42e9ddb5.png" alt="img" style="zoom: 80%;" />

**Spring通过三级缓存来解决单例模式下的循环依赖问题：**

```java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 
/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
```

**1、一级缓存：Map<String, Object> singletonObjects：**

（1）第一级缓存的作用：

- 用于存储单例模式下创建的Bean实例**（已经创建完毕）**。
- 该缓存是对外使用的，指的就是使用Spring框架的程序员。

（2）存储什么数据？

- K：bean的名称
- V：bean的实例对象（有代理对象则指的是代理对象，已经创建完毕）

**2、第二级缓存：Map<String, Object> earlySingletonObjects：**

（1）第二级缓存的作用：

- 用于存储单例模式下创建的Bean实例（**该Bean被提前暴露的引用，该Bean还在创建中**）。
- 该**缓存是对内使用的**，指的就是Spring框架内部逻辑使用该缓存。
- 为了解决第一个classA引用最终如何替换为代理对象的问题（如果有代理对象）

**3、第三级缓存：Map<String, ObjectFactory<?>> singletonFactories：**

（1）第三级缓存的作用：

- 通过ObjectFactory对象来存储单例模式下提前暴露的Bean实例的引用（正在创建中）。
- 该**缓存是对内使用的**，指的就是Spring框架内部逻辑使用该缓存。
- 此缓存是解决循环依赖最大的功臣

（2）存储什么数据？

- K：bean的名称
- V：ObjectFactory，该对象持有提前暴露的bean的引用

![img](https://img-blog.csdnimg.cn/img_convert/449a13de9e3f9e05cd888ac5b27f1ad0.png)

“A对象setter依赖B对象，B对象setter依赖A对象”，A首先完成了初始化的第一步，而且将本身提早曝光到singletonFactories中，此时进行初始化的第二步，发现本身依赖对象B，此时就尝试去get(B)，发现B尚未被create，因此走create流程，B在初始化第一步的时候发现本身依赖了对象A，因而尝试get(A)，尝试一级缓存singletonObjects(确定没有，由于A还没初始化彻底)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，因为A经过ObjectFactory将本身提早曝光了，因此B可以经过ObjectFactory.getObject拿到A对象(半成品)，B拿到A对象后顺利完成了初始化阶段一、二、三，彻底初始化以后将本身放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成本身的初始化阶段二、三，最终A也完成了初始化，进去了一级缓存singletonObjects中，并且更加幸运的是，因为B拿到了A的对象引用，因此B如今hold住的A对象完成了初始化。

### I.Spring为什么不能解决构造器的循环依赖？

构造器注入形成的循环依赖： 也就是beanB需要在beanA的构造函数中完成初始化，beanA也需要在beanB的构造函数中完成初始化，这种情况的结果就是**两个bean都不能完成初始化，循环依赖难以解决**。

Spring解决循环依赖主要是依赖三级缓存，但是的**在调用构造方法之前还未将其放入三级缓存之中**，因此后续的依赖调用构造方法的时候并不能从三级缓存中获取到依赖的Bean，因此不能解决。

### II.Spring为什么不能解决prototype作用域循环依赖？

这种循环依赖同样无法解决，因为spring不会缓存‘prototype’作用域的bean，而spring中循环依赖的解决正是通过缓存来实现的。

### III.Spring为什么不能解决多例的循环依赖？

多实例Bean是每次调用一次getBean都会执行一次构造方法并且给属性赋值，根本没有三级缓存，因此不能解决循环依赖。

### IV.如何解决构造方法注入循环依赖问题？

Spring可以自动解决单例模式下通过setter()方法进行依赖注入产生的循环依赖问题。而对于通过构造方法进行依赖注入时产生的循环依赖问题没办法自动解决，那针对这种情况，我们可以使用**@Lazy注解**来解决。

也就是说，对于类A和类B都是通过构造器注入的情况，可以在A或者B的构造函数的形参上加个@Lazy注解实现延迟加载。@Lazy实现原理是，当实例化对象时，如果发现参数或者属性**有@Lazy注解修饰，那么就不直接创建所依赖的对象了，而是使用动态代理创建一个代理类。**

## 11.BeanFactory 和 ApplicationContext有什么区别？

BeanFactory和ApplicationContext是Spring的**两大核心接口，都可以当做Spring的容器**。其中**ApplicationContext是BeanFactory的子接口。**

**依赖关系：**

**BeanFactory：是Spring里面最底层的接口**，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。

ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

- 继承MessageSource，因此支持国际化。

- 统一的资源文件访问方式。
- 提供在监听器中注册bean的事件。
- 同时加载多个配置文件。
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

**加载方式：**

- BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

- ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

- 相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。


**创建方式：**

- BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。


**注册方式：**

- BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

# 二、Spring——AOP（面向切面编程）

## 1.AOP定义

AOP，一般称为面向切面，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，**这个模块被命名为“切面”（Aspect）**，减少系统中的重复代码，降低了模块间的耦合度，提高系统的可维护性。可用于权限认证、日志、事务处理。

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。**静态代理的代表为AspectJ；动态代理则以Spring AOP为代表**。

## 2、Spring AOP 和 AspectJ

### :amphora: Spring AOP 和 AspectJ代理的区别

**AspectJ是静态代理**，也称为编译时增强，**AOP框架会在编译阶段生成AOP代理类，并将AspectJ(切面)织入到Java字节码中**，运行的时候就是增强之后的AOP对象。

**Spring AOP使用的动态代理**，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在**内存中临时为方法生成一个AOP对象**，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

**Spring AOP 已经集成了 AspectJ** ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，**当切面太多的话，最好选择 AspectJ** ，它比 Spring AOP 快很多。

### :scroll:**Spring AOP和AspectJ是什么关系**？

1. AspectJ是更强的AOP框架，是实际意义的**AOP标准**；
2. Spring为何不写类似AspectJ的框架？ Spring AOP使用纯Java实现, 它不需要专门的编译过程, 它一个**重要的原则就是无侵入性（non-invasiveness）**; Spring 小组完全有能力写类似的框架，只是Spring AOP从来没有打算通过提供一种全面的AOP解决方案来与AspectJ竞争。Spring的开发小组相信无论是基于代理（proxy-based）的框架如Spring AOP或者是成熟的框架如AspectJ都是很有价值的，他们之间应该是**互补而不是竞争的关系**。
3. Spring小组喜欢@AspectJ注解风格更胜于Spring XML配置； 所以**在Spring 2.0使用了和AspectJ 5一样的注解，并使用AspectJ来做切入点解析和匹配**。**但是，AOP在运行时仍旧是纯的Spring AOP，并不依赖于AspectJ的编译器或者织入器（weaver）**。
4. Spring 2.5对AspectJ的支持：在一些环境下，增加了对AspectJ的装载时编织支持，同时提供了一个新的bean切入点。

### :airplane:Spring AOP中代理的方式

Spring AOP中的动态代理主要有两种方式，**JDK动态代理和CGLIB动态代理**：

 ① JDK动态代理只提供接口的代理，不支持类的代理，要求被代理类实现接口。JDK动态代理的核心是InvocationHandler接口和Proxy类，在获取代理对象时，使用Proxy类来动态创建目标类的代理类（即最终真正的代理类，这个类继承自Proxy并实现了我们定义的接口），当代理对象调用真实对象的方法时， InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；

 InvocationHandler 的 invoke(Object  proxy,Method  method,Object[] args)：proxy是最终生成的代理对象;  method 是被代理目标实例的某个具体方法;  args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。

 ② **如果被代理类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。**CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此**如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的**。

![SpringAOPProcess](https://img-blog.csdnimg.cn/img_convert/230ae587a322d6e4d09510161987d346.jpeg)

### :octopus: AOP中的术语

| 术语              |                             含义                             |
| :---------------- | :----------------------------------------------------------: |
| 目标(Target)      |                         被通知的对象                         |
| 代理(Proxy)       |             向目标对象应用通知之后创建的代理对象             |
| 连接点(JoinPoint) |         目标对象的所属类中，定义的所有方法均为连接点         |
| 切入点(Pointcut)  | 被切面拦截 / 增强的连接点（切入点一定是连接点，连接点不一定是切入点） |
| 通知(Advice)      | 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情  |
| 切面(Aspect)      |                切入点(Pointcut)+通知(Advice)                 |
| Weaving(织入)     |       将通知应用到目标对象，进而生成代理对象的过程动作       |

### :canoe: AspectJ 定义的通知类型有哪些？

- **Before**（前置通知）：目标对象的方法调用之前触发
- **After** （后置通知）：目标对象的方法调用之后触发
- **AfterReturning**（返回通知）：目标对象的方法调用完成，在返回结果值之后触发
- **AfterThrowing**（异常通知） ：目标对象的方法运行中抛出 / 触发异常后触发。AfterReturning 和 AfterThrowing 两者互斥。如果方法调用成功无异常，则会有返回值；如果方法抛出了异常，则不会有返回值。
- **Around**： （环绕通知）编程式控制目标对象的方法调用。环绕通知是所有通知类型中可操作范围最大的一种，因为它可以直接拿到目标对象，以及要执行的方法，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法

![img](https://img-blog.csdnimg.cn/2020120700443256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

### :department_store: AspectJ注解方式

基于XML的声明式AspectJ存在一些不足，需要在Spring配置文件配置大量的代码信息，为了解决这个问题，**Spring 使用了@AspectJ框架为AOP的实现提供了一套注解。**

| 注解名称        | 解释                                                         |
| --------------- | ------------------------------------------------------------ |
| @Aspect         | 用来定义一个切面。(标记在类上)                               |
| @pointcut       | 用于定义切入点表达式。在使用时还需要定义一个包含名字和任意参数的方法签名来表示切入点名称，这个方法签名就是一个返回值为void，且方法体为空的普通方法。 |
| @Before         | 用于定义前置通知，相当于BeforeAdvice。在使用时，通常需要指定一个value属性值，该属性值用于指定一个切入点表达式(可以是已有的切入点，也可以直接定义切入点表达式)。 |
| @AfterReturning | 用于定义后置通知，相当于AfterReturningAdvice。在使用时可以指定pointcut / value和returning属性，其中pointcut / value这两个属性的作用一样，都用于指定切入点表达式。 |
| @Around         | 用于定义环绕通知，相当于MethodInterceptor。在使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。 |
| @After-Throwing | 用于定义异常通知来处理程序中未处理的异常，相当于ThrowAdvice。在使用时可指定pointcut / value和throwing属性。其中pointcut/value用于指定切入点表达式，而throwing属性值用于指定-一个形参名来表示Advice方法中可定义与此同名的形参，该形参可用于访问目标方法抛出的异常。 |
| @After          | 用于定义最终final 通知，不管是否异常，该通知都会执行。使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。 |

### :passenger_ship:AspectJ注解方式实现AOP

**1.开启@Aspect注解扫描**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
<!--开启注解扫描-->
    <context:component-scan base-package="com.csg.spring.aopannotation"></context:component-scan>
    <!--开启AspectsJ生成代理对象,去类中找带有@Aspects注解的对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

2.定义目标：

```java
//被增强类
@Component
public class User {
    public void add(){
        System.out.println("add....");
    }
}
```

**3.定义切面：**

```java
@Component
@Aspect
@Order(1)//()中值越小，优先级越高
public class PersonProxy {
//设置了优先级，如果是前置通知，则优先级高的先执行，如果是后置通知，则优先级高的后执行
    @Before(value = "execution(* com.csg.spring.aopannotation.User.add(..))")
    public void before(){
        System.out.println("Person before.....");
    }
}

//增强类
@Component
@Aspect//该注解表示生成代理对象
@Order(2)
public class UserProxy {
    //抽取相同的切入点，后续的通知可复用用于该切入点
    @Pointcut(value = "execution(* com.csg.spring.aopannotation.User.add(..))")
    public void sameEntryPoint(){

    }

    //前置通知
    //@Before注解表示该方法作为前置通知
    //@Before(value = "execution(* com.csg.spring.aopannotation.User.add(..))")//*代表任意修饰符列表，空格代表省略返回类型
    @Before(value = "sameEntryPoint()")
    public void before(){
        System.out.println(getClass() +"before.....");
    }
    //返回通知，表示在在返回值之后执行
    @AfterReturning(value = "sameEntryPoint()")
    public void afterReturning(){
        System.out.println("aferReturning...");
    }
    //最终通知,表示在执行被增强类中的方法执行之后执行
    @After(value = "sameEntryPoint()")
    public void after(){
        System.out.println("after...");
    }
    //表示在异常抛出前执行
    @AfterThrowing(value = "sameEntryPoint()")
    public void AfterThrowing(){
        System.out.println("AfterThrowing...");
    }

    @Around(value = "sameEntryPoint()")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前");
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后");
    }
}
```

**4、运行测试**

```java
@Test
    public void aopAno(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
        User user = context.getBean("user", User.class);
        user.add();
    }
Person before.....
环绕之前
before.....
add....
环绕之后
after...
aferReturning...
```

### :alarm_clock:AOP使用的相关问题

#### I.切入点的声明规则

Spring AOP 用户可能会经常使用 execution切入点指示符。执行表达式的格式如下：

```java
execution（modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern（param-pattern） throws-pattern?）
execution([权限修饰符][返回类型][类限定名][方法名]([参数列表]))。
```

例如：

- 对com.csg.dao.UserDao类中的show方法进行增强：execution(public void com.csg.dao.UserDao.show(..))

- 对com.csg.dao.UserDao类中的所有方法进行增强：execution(* com.csg.dao.UserDao.*(..))

- 对com.csg.dao包及其子包中的类的所有方法进行增强：execution(* com.csg.dao.*.*(..))

![img](https://pdai.tech/_images/spring/springframework/spring-framework-aop-7.png)

#### II.Spring AOP还是完全用AspectJ？

以下Spring官方的回答：（总结来说就是 **Spring AOP更易用，AspectJ更强大**）。

- Spring AOP比完全使用AspectJ更加简单， 因为它不需要引入AspectJ的编译器／织入器到你开发和构建过程中。 如果你**仅仅需要在Spring bean上通知执行操作，那么Spring AOP是合适的选择**。
- 如果你需要通知domain对象或其它没有在Spring容器中管理的任意对象，那么你需要使用AspectJ。
- 如果你想通知除了简单的方法执行之外的连接点（如：调用连接点、字段get或set的连接点等等）， 也需要使用AspectJ。

| Spring AOP                                       | AspectJ                                                      |
| ------------------------------------------------ | ------------------------------------------------------------ |
| 在纯 Java 中实现                                 | 使用 Java 编程语言的扩展实现                                 |
| 不需要单独的编译过程                             | 除非设置 LTW，否则需要 AspectJ 编译器 (ajc)                  |
| 只能使用运行时织入                               | 运行时织入不可用。支持编译时、编译后和加载时织入             |
| 功能不强-仅支持方法级编织                        | 更强大 - 可以编织字段、方法、构造函数、静态初始值设定项、最终类/方法等......。 |
| 只能在由 Spring 容器管理的 bean 上实现           | 可以在所有域对象上实现                                       |
| 仅支持方法执行切入点                             | 支持所有切入点                                               |
| 代理是由目标对象创建的, 并且切面应用在这些代理上 | 在执行应用程序之前 (在运行时) 前, 各方面直接在代码中进行织入 |
| 比 AspectJ 慢多了                                | 更好的性能                                                   |
| 易于学习和应用                                   | 相对于 Spring AOP 来说更复杂                                 |

# 三、Spring事务

## **1、Spring事务的实现方式和实现原理：**

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。Spring只提供**统一事务管理接口，具体实现都是由各数据库自己实现**，数据库事务的提交和回滚是通过 redo log 和 undo log实现的。Spring会在事务开始时，根据当前环境中**设置的隔离级别，调整数据库隔离级别，由此保持一致。**

## 2.Spring事务的种类：

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate。

```java
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {

                    // ....  业务代码
                } catch (Exception e){
                    //回滚
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}
```

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

**声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码**，只需在配置文件中做相关的事务规则声明或通过**@Transactional注解的方式**，便可以将事务规则应用到业务逻辑中，减少业务代码的污染。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

 Propagation propagation() default Propagation.REQUIRED;
 Isolation isolation() default Isolation.DEFAULT;
  int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
  boolean readOnly() default false;

}
```

- @Transactional 注解中的 propagation 对应 TransactionDefinition 中的 getPropagationBehavior，默认值为 Propagation.REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED)。
- @Transactional 注解中的 isolation 对应 TransactionDefinition 中的 getIsolationLevel，默认值为 DEFAULT(TransactionDefinition.ISOLATION_DEFAULT)。
- @Transactional 注解中的 timeout 对应 TransactionDefinition 中的 getTimeout，默认值为TransactionDefinition.TIMEOUT_DEFAULT。
- @Transactional 注解中的 readOnly 对应 TransactionDefinition 中的 isReadOnly，默认值为 false。

### 事务的回滚策略：

**回滚策略rollbackFor **，用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。默认情况下，事务只在**出现运行时异常（Runtime Exception）时回滚，以及 Error，出现检查异常（checked exception，需要主动捕获处理或者向上抛出）时不回滚**。

如果你想要回滚特定的异常类型的话，可以这样设置：

```java
@Transactional(rollbackFor= MyException.class)
```

## 3.说一下Spring的事务传播行为

事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时事务如何传播。

```java
public void methodA(){
    methodB();
    //doSomething
 }

 @Transaction(Propagation=XXX)
 public void methodB(){
    //doSomething
 }
```

代码中`methodA()`方法嵌套调用了`methodB()`方法，`methodB()`的事务传播行为由`@Transaction(Propagation=XXX)`设置决定。这里需要注意的是`methodA()`并没有开启事务，某一个事务传播行为修饰的方法并不是必须要在开启事务的外围方法中调用。

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为，**传播行为自己配置**。

① PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。

② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。

③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

④ PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。

⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。

⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

------

*(示例1)*根据场景举栗子,我们在testMain和testB上声明事务，设置传播行为REQUIRED，伪代码如下：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void testMain(){
    A(a1);  //调用A入参a1
    testB();    //调用testB
}
@Transactional(propagation = Propagation.REQUIRED)
public void testB(){
    B(b1);  //调用B入参b1
    throw Exception;     //发生异常抛出
    B(b2);  //调用B入参b2
}
```

该场景下执行testMain方法结果如何呢？

数据库没有插入新的数据，数据库还是保持着执行testMain方法之前的状态，没有发生改变。testMain上声明了事务，在执行testB方法时就加入了testMain的事务（**当前存在事务，则加入这个事务**），在执行testB方法抛出异常后事务会发生回滚，又testMain和testB使用的同一个事务，所以事务回滚后testMain和testB中的操作都会回滚，也就使得数据库仍然保持初始状态

*(示例2)*根据场景再举一个栗子,我们只在testB上声明事务，设置传播行为REQUIRED，伪代码如下：

```java
public void testMain(){
    A(a1);  //调用A入参a1
    testB();    //调用testB
}
@Transactional(propagation = Propagation.REQUIRED)
public void testB(){
    B(b1);  //调用B入参b1
    throw Exception;     //发生异常抛出
    B(b2);  //调用B入参b2
}
```

这时的执行结果又如何呢？

数据a1存储成功，数据b1和b2没有存储。由于testMain没有声明事务，testB有声明事务且传播行为是REQUIRED，所以在执行testB时会自己新建一个事务（**如果当前没有事务，则自己新建一个事务**），testB抛出异常则只有testB中的操作发生了回滚，也就是b1的存储会发生回滚，但a1数据不会回滚，所以最终a1数据存储成功，b1和b2数据没有存储

## 4.说一下 spring 的事务隔离？

spring 有五大隔离级别，**默认值为 ISOLATION_DEFAULT（使用数据库的设置），**其他四个隔离级别和数据库的隔离级别一致：

- ISOLATION_DEFAULT：用底层数据库的设置隔离级别，数据库设置的是什么我就用什么；
- ISOLATION_READ_UNCOMMITTED：未提交读，最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）；
- ISOLATION_READ_COMMITTED：提交读，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），SQL server 的默认级别；
- ISOLATION_REPEATABLE_READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别；
- ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

## 5.@Transaction失效场景

#### 1、数据库引擎不支持事务

这里以 MySQL 为例，其 MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎，一般要支持事务都会使用 InnoDB。

#### 2、没有被 Spring 管理

如下面例子所示：

```typescript
// @Service
public class OrderServiceImpl implements OrderService {

    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

如果此时把 `@Service` 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。

#### 3、方法不是 public 的

`@Transactional` 只能用于 public 的方法上，否则事务不会失效，如果要用在非 public 方法上，可以开启 `AspectJ` 代理模式。

#### 4、同一类中的自身方法调用

```java
@Service
public class OrderServiceImpl implements OrderService {

    public void update(Order order) {
        updateOrder(order);
    }
    
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

因为spring声明式事物是基于AOP实现的，是使用动态代理来达到事物管理的目的，当前类调用的方法上面加@Transactional 这个是没有任何作用的，因为调用这个方法的是this，没有经过 Spring 的代理类。

**解决方法：**

1. **再声明一个service，将内部调用改为外部调用。**

```java
@Service
public class MyService {
    
	@Resource
    OrderService orderServiceImpl;
    
    public void update(Order order) {
        orderServiceImpl.updateOrder(order);
    }
}

@Service
public class OrderServiceImpl implements OrderService {
    
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
    
}
```

2) **使用编程式事务**

![img](https://pic2.zhimg.com/v2-c91b12ceefa661fff0b7cfbeb490cae1_r.jpg)

#### 5、方法不是Public

Spring官方文档中有明确的说明，@Transactional 只有修饰public方法时才会生效，修饰protected、private或者包内可见的方法均不会生效。

解决方法：@Transactional 修饰public方法即可，或者**@Transactional 修饰类，需要事务生效的方法都为public**。

#### 6、事物传播性问题

![img](https://pic3.zhimg.com/v2-f45aae70a11a3096f8cb035176d7379a_r.jpg)

- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行

**当传播行为设置了PROPAGATION_NOT_SUPPORTED，PROPAGATION_NEVER，PROPAGATION_SUPPORTS这三种时，就有可能存在事物不生效。**

#### 7、使用cglib代理

要注意的是**单纯使用cglib代理是不会出现事务失效的情况**，只有当接口层使用声明式事务同时使用了cglib代理才会出现事务失效的情况。如下图为接口层使用声明式事务示例。

![img](https://pic1.zhimg.com/80/v2-c6bab2b8105a858a6c485d7024e96d10_720w.webp)

解决方法：**cglib代理和接口层声明式事务二者不同时使用**。

#### 8、rollbackFor异常指定错误

![img](https://pic3.zhimg.com/80/v2-2246993b27d6cdda440179aed0fc36aa_720w.webp)

例如rollbaclFor指定的是DataFormatException，但程序没有抛DataFormatException，因此事务不会回滚。若没有指定回滚异常，默认的回滚异常是RuntimeException ，如果出现其他异常那么就不会回滚事物。

解决方法：明确需要回滚的异常，正确设置rollbackFor的异常class。

#### 9、异常被catch住了

代码中手动catch了异常，然后又未抛出来，此时事务就不生效了。

解决方法：要么不catch需要回滚的异常，要么catch之后再抛出。

![img](https://pic1.zhimg.com/80/v2-0969f3bebfb20063b81e1f686fc2a818_720w.webp)

# 四、Spring 框架中都用到了哪些设计模式？

1. 工厂模式：Spring使用工厂模式，通过BeanFactory和ApplicationContext来创建对象

2. 单例模式：Bean默认为单例模式

3. 策略模式：例如Resource的实现类，针对不同的资源文件，实现了不同方式的资源获取策略

4. 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术

5. 模板方法：可以将相同部分的代码放在父类中，而将不同的代码放入不同的子类中，用来解决代码重复的问题。比如RestTemplate, JmsTemplate, JpaTemplate

6. 适配器模式：Spring AOP的增强或通知（Advice）使用到了适配器模式，Spring MVC中也是用到了适配器模式适配Controller

7. 观察者模式：Spring事件驱动模型就是观察者模式的一个经典应用。

8. 桥接模式：可以根据客户的需求能够动态切换不同的数据源。比如我们的项目需要连接多个数据库，客户在每次访问中根据需要会去访问不同的数据库
