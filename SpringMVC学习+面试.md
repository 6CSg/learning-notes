# 一、SpringMVC常用注解介绍

## 1.@Controller

controller主要负责处理前端控制器(DispatcherServlet )发过来的请求，经过业务逻辑层处理之后封装层一个model，并将其返回给view进行展示。@controller注解通常用于类上，如果结合Thymeleaf模板使用的话，会返回一个页面。如果是前后端分离的项目，则使用@RestController，表明返回的是json格式数据。

## 2.@RestController

@RestController注解里面包含了@Controller注解和@ResponseBody注解，@ResponseBody 注解是将返回的数据结构转换为 JSON 格式，所以说可以这么理解：**@RestController = @Controller + @ResponseBody** ，省了很多事，我们使用 @RestController 之后就不需要再使用 @Controller 了

## 3.@RequestMapping

@RequestMapping 是一个用来处理请求地址映射的注解，它可以用于类上，也可以用于方法上。用于类上的注解会将一个特定请求或者请求模式映射到一个控制器之上，表示类中的所有响应请求的方法都是以该地址作为父路径；方法的级别上注解表示进一步指定到处理方法的映射关系。

```java
@Controller
@RequestMapping("/test")
public class RequestMappingController {

	//此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }

}
```

**注：**

对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解：

- 处理get请求的映射-->@GetMapping

- 处理post请求的映射-->@PostMapping

- 处理put请求的映射-->@PutMapping

- 处理delete请求的映射-->@DeleteMapping

## 4.@PathVariable

@PathVariable 注解主要用来获取 URL 参数，Spring Boot 支持 Restfull 风格的 URL，比如一个 GET 请求携带一个参数 id，我们将 id 作为参数接收，可以使用 @PathVariable 注解。如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529114405729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTEzOTkyOQ==,size_16,color_FFFFFF,t_70)

这里需要注意一个问题，如果想要 URL 中占位符中的 id 值直接赋值到参数 id 中，需要保证 URL 中的参数和方法接收参数一致，否则将无法接收。如果不一致的话，其实也可以解决，需要用 @PathVariable 中的 value 属性来指定对应关系。如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529114430469.png)

## 5.@RequestParam

@PathValiable 是从 URL 模板中获取参数值， 即这种风格的 URL：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529114746903.png)
@RequestParam 是从 Request 里获取参数值，即这种风格的 URL：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529114803767.png)
对于@RequestParam 注解代码测试如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529114824646.png)

## 6.@RequestBody

注解实现接收http请求的json数据，将json转换为java对象。

RequestBody 注解用于接收前端传来的实体，接收参数也是对应的实体，比如前端通过 JSON 提交传来两个参数 username 和 password，此时我们需要在后端封装一个实体来接收。在传递的参数比较多的情况下，使用 @RequestBody 接收会非常方便。例如：

@PostMapping("/user")
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190529114850677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTEzOTkyOQ==,size_16,color_FFFFFF,t_70)
可以看出，**@RequestBody 注解用于 POST 请求上，接收 JSON 实体参数。**它和上面我们介绍的表单提交有点类似，只不过参数的格式不同，一个是 JSON 实体，一个是表单提交。在实际项目中根据具体场景和需要使用对应的注解即可。

## **7.@ExceptionHandler**

@ExceptionHander注解用于标注处理特定类型异常类所抛出异常的方法。当控制器中的方法抛出异常时，Spring会自动捕获异常，并将捕获的异常信息传递给被@ExceptionHandler标注的方法。

![image-20221117165124160](C:\Users\龚chang\AppData\Roaming\Typora\typora-user-images\image-20221117165124160.png)

## 8.@CrossOrigin

@CrossOrigin注解将为请求处理类或请求处理方法提供跨域调用支持。如果我们将此注解标注类，那么类中的所有方法都将获得支持跨域的能力。使用此注解的好处是可以微调跨域行为。使用此注解的示例如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/KX5qDhxsEwPBzcTicD4VcgVTUOHGc72CXDXk1ibI9yFpvYXSJa2bOcvYwR6Tpdoia3dPRcjk3YQUvdZ8lDgFoJuMw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

# 二、什么是MVC模式？

MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/java-guide-blog/image-20210809181452421.png" alt="img" style="zoom:80%;" />

**MVC的全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，是一种软件设计典范。**它是用一种业务逻辑、数据与界面显示分离的方法来组织代码，将众多的业务逻辑聚集到一个部件里面，在需要改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑，达到减少编码的时间。

- V即View视图是指**用户看到并与之交互的界面**。比如由html元素组成的网页界面，或者软件的客户端界面。MVC的好处之一在于它能为应用程序处理很多不同的视图。在视图中其实没有真正的处理发生，它只是作为一种输出数据并允许用户操纵的方式。
- M即model模型是指模型**表示业务规则**。在MVC的三个部件中，模型拥有最多的处理任务。被模型返回的数据是中立的，模型与数据格式无关，这样一个模型能为多个视图提供数据，由于应用于模型的代码只需写一次就可以被多个视图重用，所以减少了代码的重复性。
- C即controller控制器是指控制器**接受用户的输入并调用模型和视图去完成用户的需求**，控制器本身不输出任何东西和做任何处理。它只是接收请求并决定调用哪个模型构件去处理请求，然后再确定用哪个视图来显示返回的数据。

# 三、 Spring MVC 的核心组件有哪些？

记住了下面这些组件，也就记住了 SpringMVC 的工作原理。

- **`DispatcherServlet`** ：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应。
- **`HandlerMapping`** ：**处理器映射器**，根据 uri 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装。
- **`HandlerAdapter`** ：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`；
- **`Handler`** ：**请求处理器**，处理实际请求的处理器。
- **`ViewResolver`** ：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端



# 四、SpringMVC的执行流程？

![img](https://img-blog.csdnimg.cn/20200314193953559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDE3NjE2OQ==,size_16,color_FFFFFF,t_70)

1. 客户端（浏览器）发送请求， `DispatcherServlet`拦截请求。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping` 。`HandlerMapping` 根据 uri 去匹配查找能处理的 `Handler`（也就是我们平常说的 `Controller` 控制器） ，并会将请求涉及到的拦截器（`Inteceptor`）和 `Handler` 一起封装。
3. `DispatcherServlet` 调用 `HandlerAdapter`适配执行 `Handler` 。
4. `Handler` 完成对用户请求的处理后，会返回一个 `ModelAndView` 对象给`DispatcherServlet`，`ModelAndView` 顾名思义，包含了数据模型以及相应的视图的信息。`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
5. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
6. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
7. 把 `View` 返回给请求者（浏览器）

# 五、统一异常处理怎么做？

推荐使用注解的方式统一异常处理，具体会使用到 `@ControllerAdvice` + `@ExceptionHandler` 这两个注解 。

```java
@ControllerAdvice//将当前类标识为异常处理的组件
@ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(BaseException.class)
    public ResponseEntity<?> handleAppException(BaseException ex, HttpServletRequest request) {
      //......
    }

    @ExceptionHandler(value = ResourceNotFoundException.class)
    public ResponseEntity<ErrorReponse> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
      //......
    }
}
```

这种异常处理方式下，会给所有或者指定的 `Controller` 织入异常处理的逻辑（AOP），当 `Controller` 中的方法抛出异常的时候，由被`@ExceptionHandler` 注解修饰的方法进行处理。

`ExceptionHandlerMethodResolver` 中 `getMappedMethod` 方法决定了异常具体被哪个被 `@ExceptionHandler` 注解修饰的方法处理异常。

# 六、拦截器

## 1.SpringMVC中的拦截器有三个抽象方法：

- preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法

- postHandle：控制器方法执行之后执行postHandle()

- afterComplation：处理完视图和模型数据，渲染视图完毕之后执行afterComplation()


## 2、多个拦截器的执行顺序

- **若每个拦截器的preHandle()都返回true：**

此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的配置顺序有关，手动设置优先级：

preHandle()会按照配置的顺序执行，而postHandle()和afterComplation()会按照配置的反序执行

- **若某个拦截器的preHandle()返回了false：**

preHandle()返回false和它之前的拦截器的preHandle()都会执行，postHandle()都不执行，返回false的拦截器之前的拦截器的afterComplation()会执行