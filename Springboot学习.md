# 一、了解自动配置原理

# 1、SpringBoot特点

没有 Spring Boot 的情况下，如果我们需要引入第三方依赖，需要手动配置，非常麻烦。但是，Spring Boot 中，我们直接引入一个 starter 即可。比如你想要在项目中使用 redis 的话，直接在项目中引入对应的 starter 即可。

## 1.1、依赖管理

- **父项目做依赖管理**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

他的父项目
 <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
  </parent>
```

- **开发导入starter场景启动器**

  - 见到很多 spring-boot-starter-* ： *就某种场景

  - 只要引入starter，这个场景的所有常规需要的依赖我们都自动引入

  - SpringBoot所有支持的场景 https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter

  - 见到的  *-spring-boot-starter： 第三方为我们提供的简化开发的场景启动器。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
---------------这是以下所有场景启动器的底层依赖---------------------
    这些都是场景启动器
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
    </dependency>
```

- **需关注版本号，自动版本仲裁**
  1. 引入依赖默认都可以不写版本
  2. 引入非版本仲裁的jar，要写版本号

- **可以修改默认版本号**

```xml
1、查看spring-boot-dependencies里面规定当前依赖的版本 用的 key。
2、在当前项目里面重写配置
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>    
</properties>
```

## 1.2、自动配置

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration //允许在上下文中注册额外的 bean 或导入其他配置类
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
 …………   
}
```

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@SpringBootConfiguration`：允许在上下文中注册额外的 bean 或导入其他配置类
- `@ComponentScan`：扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。如下图所示，容器中将排除`TypeExcludeFilter`和`AutoConfigurationExcludeFilter`。

## 1.3、条件注入

<img src="https://cdn.nlark.com/yuque/0/2020/png/1354552/1602835786727-28b6f936-62f5-4fd6-a6c5-ae690bd1e31d.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_17%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10" alt="image.png" style="zoom: 80%;" />

- `@ConditionalOnBean`：要求bean存在时，才会创建这个bean；
- `@ConditionalOnMissingBean`：要求bean不存在时，才会创建这个bean；
- `@ConditionalOnClass`：要求class存在时，才会创建这个bean；
- `@ConditionalOnMissingClass`：要求class不存在时，才会创建这个bean；
- `@ConditionalOnProperty`：主要是根据配置参数（写在配置文件中），来决定是否需要创建这个bean，这样就给了我们一个根据配置来控制Bean的选择的手段了
- `@ConditionalOnExpression`：相比较前面的Bean，Class是否存在，配置参数是否存在或者有某个值而言，这个依赖SPEL表达式的，就显得更加的高级了；**其主要就是执行Spel表达式**，根据返回的true/false来判断是否满足条件

```java
 @Bean
    @ConditionalOnExpression("#{'true'.equals(environment['conditional.express'])}")
    public ExpressTrueBean expressTrueBean() {
        return new ExpressTrueBean("express true");
    }
```

对应的配置如下

```properties
conditional.express=true
```

