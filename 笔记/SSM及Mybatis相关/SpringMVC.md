# SpringMVC

工作原理



工作流程

1. 用户发送请求至前端控制器DispatcherServlet。

2. DispatcherServlet收到请求调用HandlerMapping处理器映射器(根据请求的url查找Controller)。

3. 处理器映射器找到具体的处理器(Controller)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4. DispatcherServlet调用HandlerAdapter处理器适配器。

5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。

6. Controller执行完成返回ModelAndView。

7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

9. ViewReslover解析后返回具体View。

10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。

11. DispatcherServlet响应用户。

### 使用

1. 创建web项目

2. **在web.xml中注册DispatherServlet前端控制器**

    设置资源路径为/(只会匹配servlet请求)，注意不要写成/\*(/\*会匹配jsp页面)。

    设置初始化参数为resources目录下的springmvc.xml配置文件，设置服务器开启后立即创建该servlet。作用是加载springmvc配置文件。

```xml
<!-- 注册前端控制器-->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    
    <!--设置初始化参数
	这里写的这个名字是有讲究的，如果我们的文件名为<servlet-name>-servlet.xml，则不用配置此项
	spring会自动查找web-INF下的此文件
         -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <!--读取类路径下的springmvc.xml-->
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!--服务器开启后立即创建-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>



<!-- 解决乱码问题-->
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

3. 配置springmvc.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--扫播包，以指定控制层@controller-->
    <context:component-scan base-package="com.origami.controller"/>
    <!--让spring MVC不处理静态资源 .css.js.html.mp3.mp4-->
    <mvc:default-servlet-handler/>
    <!--
    开启mvc的注解支持,这样我们可以使用注解
    而不用去配置DefaultAnnotationHandleMapper和AnnotationMethodHandleAdapter
    -->
    <mvc:annotation-driven/>

    <!--配置视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--设置前缀、后缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>


</beans>
```

>  注意 ：maven创建的项目在WEB-INF下没有lib，需要手动到项目结构的artifacts的选项下手动添加jar包。

@RequestMapping("/xxx")

* 表示调用此方法时要访问的路径，即servlet中的@webservlet
* 可用于类上、方法上(用于类上表示调用哪个类的哪个方法)
* 属性
  * path表示访问路径
  * value，只有一个值时，指代path属性，此时可省略
  * method，指定请求方式，post、get等，只有符合指定请求时才会执行方法
  * params，用于限定请求参数，params=“username”，表示必须有属性username才会执行方法，params=“money!100”表示属性money的值不能为100
  * headers，必须包含执行请求头才能执行方法
  * produces 指定

支持restful风格

restful风格既是没有?和&将请求参数分割，而是直接使用/将参数进行分割

##### URI

需要使用@ParhVariable注解

将占位符{xxx}中的值传递给@ParhVariable("xxx")后面的参数

```java
//若请求路径为；localhost:8080/项目名/test/1001/张三
@RequestMapping("/test/{id}/{name}")
public String test(@PathVariable("id")int id,@PathVariable("name")String name){
    xxx;
}
```

上述代码，将请求参数1001的值赋给占位符{id}，而@PathVariable("id")将id的值赋给参数int的id，name同理。

可以通过实现`WebMvcConfigurer`接口来自定义MVC的配置，其内部已有默认实现，可以只对自己想要自定义的配置进行配置。[官方文档][https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-customize]

```properties
addFormatters() : 格式化和转换器
addInterceptors() : 拦截器
addviewControllers() : 路径转发，指定url到指定页面
addResourceHandlers : 静态资源的处理器
configureViewResolvers() : 视图解析器
configureMessageConverters() : 信息转换器，如将http请求转换为request，将javabean转换为json格式发送给浏览器
```

```java
@Configuration
// 注意使用@EnableWebMvc会使springboot关于mvc的自动配置失效
public class WebConfig implements WebMvcConfigurer {
    
    // 拦截器 
    @override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**r") 
            .excludePathPatterns("/hello");
    }
    
    // 路由跳转
    @override
	public void addviewcontrollers(ViewcontrollerRegistry registry){
		registry.addviewcoptroller("/").setviewName("home");
    }
    
    // 静态资源过滤
    @override
    public void addResourceHandlers(ResourceHandlerRegistry registry){
        registry.addResourceHandler("/res/**")
            .addResourceLocations("classpath:/tx static/");
    }
                                                                     
                                                                     
}
```

### 获取参数

##### 接收普通字段

请求方法的形参与表单传递的参数名一致时，可以自动接受到参数。

```java
//localhost:8080/addForm?username=xxx&password=xxx&age=xxx
@RequestMapping("/addForm")
public String addUser(String username, String password, int age) {
    System.out.println(username+"---"+password+"---"+age);
    return "add";
}
```

当提交的表单项和处理方法的参数名不一致时，可以使用注解==**@RequestParam**==注解

@RequestParam(value="xxx")属性将表单传递的xxx参数的值赋给注解后的形参

可以修改注解属性的requied为false，表示如果有指定参数，则赋值给形参，没有则不赋值但是不报错

```java
//localhost:8080/addForm?username=xxx&password=xxx&age=xxx
@RequestMapping("/addForm")
//将username的值赋给name
public String addUser(@Requestparam("username") string name){
    system. out. println(name);
    return "hello";
}
```

> 推荐无论参数名是否一致都使用@RequestParam注解，以明确表示出这是要从前端接受的数据。

##### 接受bean类型

bean

```java
public class User {
    private String username;
    private String password;
    private int age;
    //代码
｝
```
请求方法

```java
//localhost:8080/addUser?username=xxx&password=xxx&age=xxx
@RequestMapping("/addUser")
    public String addUser(User user) {
        System.out.println(user);
        //代码
    }
```

即当表单项的属性与bean对象的属性名称对应时，可以直接将表单想直接封装到bean对象中。

##### 级联bean

```java
public class User {
    private String username;
    private String password;
    private int age;
    private Adress adress;
    ...
｝
    
public class Adress {
    private String province;
    private String city;
    ...
｝
```

表单

```html
<input type="text" name="username"/>
<input type="text" name="password"/>
<input type="text" name="age"/>
<input type="text" name="adress.province"/>
<input type="text" name="adress.city"/>
```

此时也可以直接放装进User对象，且Adress也被封装。

##### servlet原生Api

直接在方法的参数上加上想要获取的api即可

```java
@RequestMapping("/addUser")
public String addUser(User user,HttpServletRequest req) {
    System.out.println(user);
    //可以直接使用req
    req.getParameter("xxx");
    //代码
}
```

##### 向域对象中存取

因为我们没有requset和session对象，所以不能像servelt一样直接存值，可以先用上面获取原生api然后获取requset对象和session对象再存取，也可以使用ModleAndView。

```java
public ModelAndView param（）{
    ModelAndView mav=new ModelAndView();
    mav.addObject("username"，"root");//往requset作用域中放值	 
    mav.setViewName（"success"};//跳转到指定页面
    return mav;
}
```

##### 接收集合

```java
public class User {
    private String username;
    private String password;
 
    private List<User> userList;
    private Map<String,User> userMap;
    //代码
｝
```

```html
<input type="text" name="username"/>
<input type="text" name="password"/>

<input type="text" name="userList[0].username"/>
<input type="text" name="userList[0].password"/>
<input type="text" name="userMap['key1'].username"/>
<input type="text" name="userMap['key2'].password"/>
```

即可将表单中的属性封装到ist和map中。

### 乱码

之前我们遇到乱码问题，可以在过滤器中配置全局编码，在SprinfMVC中，已经内置了一个过滤器，对编码进行设置，我们只需要在web.xml中添加此过滤器即可。

> 注意 :==过滤器路径设置为/\*==，/为表示只过滤servlet请求，/\*表示过滤所有，包括jsp页面等。

```xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### JSON

以键值对的形式

##### 常用json

* gson(谷歌)
  * 直接创建默认设置的Gson对象，或者创建GsonBuilder对象，进行自定义设置，然后使用builder的create()方法创建Gson对象。
  * 常用方法：
    * fromJson(json,type): 将json对象转换为指定对象
    * toJson(object) : 将指定对象转换为json
  * 还有创建GsonBuilder设置Date类型格式及输出标准格式及从文件读取等。

* jackson(springMVC默认使用，需要导入jar)
  * 主要使用ObjectMapper类
  * **==writeValueAsString(Objcet)==** :　将对象转换为json字符串
  * ==**readValue(String, Class)**==： 从json字符串读取并转换为指定的对象

### 注解

==@Controller==

*   用于类上，使指定为成为控制器

*   @RestController  
    *   rest风格，由@Controller和@ResponseBody组成
    *   不返回页面，直接返回json字符串

==@RequestMapping("/hello")==

*   路径映射
*   @GetMapping,@PostMapping,@PutMapping
    *   rest风格

==@Requestparam==

* 用于形参前，将表单传递的参数的值赋给注解后的形参

  * value表示将表单哪个属性赋值给对应形参
  * requied，默认取值true，表示此参数是必须的，表单必须提交，为false表示若表单有提交xxx值，则赋值给形参，没有则不赋值但是不报错。
  * default，可选，没有传递值时的默认值。

  ```java
  //localhost:8080/addForm?username=xxx&password=xxx&age=xxx
  @RequestMapping("/addForm")
  //将表单中username的值赋给name
  public String addUser(@Requestparam("username") String name){
      system. out. println(name);
      return "hello";
  }
  ```

==@RequestBody==

* 将前端发送的json格式的字符串转换为Java对象

* 用于形参前，获取请求体，以key=value&key=value形式，==**必须是post请求**==

  * requied，是否必须，若为true，get请求报错，为false，get请求为null

  ```java
  //localhost:8080/addForm?username=xxx&password=xxx&age=xxx
  @RequestMapping("/addForm")
  //获取post请求的请求体内容
  public String addUser(@RequestBody User user){
      system. out. println(user);
      return "hello";
  }
  ```

==@ResponseBody==

* 用在返回值上或方法上，将返回的类型转换为json，传给前端，而不是返回页面

* springMVC默认使用jackson转换json，需要添加jackson的依赖

  ```java
  @RequestMapping("/addForm")
  @ResponseBody
  public User addUser(@RequestBody String body){
      User user = new User();
      user.setXxx();
      ...
          
      return user;
  }
  ```

  此时，页面会收到json格式的user对象

==@PathVariable==

* 将路径上的占位符传递给参数

    ```java
    @Controller 
    @RequestMapping("/owners/{a}") 
    public class RelativePathUriTemplateController { 
    
        @RequestMapping("/pets/{b}") 
        public void findPet(@PathVariable("a") String a,@PathVariable String b, Model model) {     
            // implementation omitted 
        } 
    }
    ```

@RequeusetHeader

* 用于指定请求必须有某一header
  * value：提供消息头名称
  * required：是否必须有此消息头
  * 开发中一般不用

@CookieValue

* 用于获取指定的cookie

  * value，cookie的键
  * requied，cookie是否必须

  ```java
  @RequestMapping(value="/testCookieValue")
  public String testCookieValue(@CookieValue(value="JSESSTONID")String cookieValue){
  	System.out.print1n(cookieValue);
      return"success";
  }
  ```

@SessionAttribute

* 用于类上，将类中方法上modle中存入的值放到session域中，其他方法可以通过ModleMap的t方法获取 session中的值，也可以删除sessin中的值。

  * value，键

  ```java
  @Controller
  @ModleAttributes(value="msg")
  public class Test{
      
      public String set(ModleMap map){
          map.addAttribute("msg","xxx");
      }
      
      public String set(ModleMap map){
          map.get("msg");
      }
  }
  ```

@ModleAttribute

* 用于方法上或形参上，会先于其他方法执行，将数据保存到域中，然后执行其他方法时，可以获取到该属性

==@ControllerAdvice==

*   用于类上，使指定的Controller成为异常处理的Controller
*   与@ExceptionHandle配合使用
*   @RestControllerAdvice

==@ExceptionHandle==

*   用于标注了@ControllerAdvice类的方法上，发生指定异常时执行

### 返回值

##### String

* 返回值字符串指定页面
* 使用==forword:==关键字，请求转发到指定页面
* 使用 ==redirect:==关键字，重定向向指定页面(框架已默认加了项目名)

##### void

会默认跳转到请求路径.jsp路径

##### ModleAndView

与String类似，方法内设置返回的页面，返回ModleAndView

### 文件上传

##### 前提

* form表单的enctype取值必须是：multipart/form-data(
  * 默认值是：application/x-www-form-urlencoded
  * enctype：是表单请求正文的类型
* method属性取值必须是Post
* 提供一个文件选择域`<input type="file"/>`

##### 使用

普通的上传我们需要使用commons-fileupload组件一步一步获取每个部分，springMVC给我们创建了一个上传文件解析器(==CommonsMultipartResolver==)，当我们的上传时，前段控制器会调用上传解析器来为我们解决上传问题。需要commons-fileupload上传组件。

先要在SpringMVC的配置文件中配置文件解析器

```xml
<!--配置文件解析器对象,要求id名称必须是multipartResolver-->
<!--内部属性可自定义配置，如最大文件大小等-->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxuploadSize"value="10485760"/>
</bean>
```

接收方法

```java
@RequestMapping("/upload")
public String upload(HttpServletRequest req, MultipartFile upload) throws IOException {
    //获取uploads文件夹中所在的真实路径
    String path = req.getSession().getServletContext().getRealPath("/uploads/");
    //获取上传文件名
    String filename = upload.getOriginalFilename();
    //防止文件名重复
    filename = UUID.randomUUID().toString().replace("-", "") + filename;

    //文件保存到uploads文件夹下
    upload.transferTo(new File(path, filename));
    return "add";
}
```

> 注意 ： 方法参数MultipartFile的参数名应该与文件上传时`<input type="file" name="upload"/>`中的name属性的名字一样。

##### 跨服务器上传

需要添加jar包

```xml
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-core</artifactId>
    <version>1.18.1</version>
</dependency>
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-client</artifactId>
    <version>1.18.1</version>
</dependency>
```

用于上传的tomcat的配置文件

```xml
<init-param>
    <param-name>readonly</param-name>
    <param-value>false</param-value>
</init-param>
```

```java
@RequestMapping("/fileupload")
public String fileuoload3(MultipartFile upload) throws Exception {
    // 定义上传文件服务器路径
    String path = "http://localhost:9090/uploads/";

    // 说明上传文件项
    // 获取上传文件的名称
    String filename = upload.getOriginalFilename();
    // 把文件的名称设置唯一值，uuid
    String uuid = UUID.randomUUID().toString().replace("-", "");
    filename = uuid+"_"+filename;

    // 创建客户端的对象
    Client client = Client.create();

    // 和图片服务器进行连接
    WebResource webResource = client.resource(path + filename);

    // 上传文件
    webResource.put(upload.getBytes());

    return "success";
}

```

### 全局异常处理

出现问题向上抛，直到抛给前端控制器，前端控制器调用异常处理器处理异常。

* 前后端分离

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(SQLException.class)
    public RespBean sqlException(SQLException e) {
        if (e instanceof SQLIntegrityConstraintViolationException) {
            return RespBean.error("该数据有关联数据，操作失败!");
        }
        return RespBean.error("数据库异常，操作失败!");
    }
    
    @ExceptionHandler(AccessDeniesException.class)
    public RespBean accessDeniesException(AccessDeniesException e) {
        return RespBean.error("权限不足");
    }
}
```

* 使用模版引擎

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(SQLException.class)
    public String sqlException(SQLException e) {
      
        return "forword:/404.html";
    }
    
    @ExceptionHandler(AccessDeniesException.class)
    public RespBean accessDeniesException(AccessDeniesException e) {
        return "forword:/403.html";
    }
}
```

### 拦截器

拦截器类似于Javaweb中的过滤器，但拦截器是SpringMVC中的特有的，只有在SpringMVC中可以使用，且它只拦截请求，而不像过滤器一样可以拦截任何资源。 

实现HandleInterceptor接口

添加到xml配置文件中

加AOP实现切面增强

