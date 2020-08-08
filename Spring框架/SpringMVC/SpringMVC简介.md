## 三层架构和MVC
服务端程序一般是基于两种形式，一种是客户端-服务器（C/S）结构，另一种是浏览器和服务器（B/S）结构，我们使用java基本是开发B/S架构的程序。该架构分称三层架构：

（1）表现层：即Web层。它负责接收客户端请求，向客户端响应结构，表现层一般采用MVC的设计模型。

MVC全名为模型视图控制器，它分为三个部分：
* Model数据模型：用于封装数据
* View：比如jsp，html用于展示数据的页面
* Controller：接收用户的请求。

（2）业务层：即service层。它负责处理具体业务逻辑

（3）持久层：即dao层，用来与数据库交互，进行增删改查操作。

早期的MVC模型是由Servlet（控制器Controller角色），jsp（视图层角色） 和 javaBean（模型Model角色）构成：当用户发起请求后，Servlet会根据请求调用相关的java bean，执行相关逻辑操作后，将需要显示的数据交给jsp去展示。



###  SpringMVC 是什么

SpringMVC 由于java实现MVC设计模型的请求驱动类型的轻量级Web框架。该框架将原本的模型层Model拆分为业务层service和数据访问层dao：

- dao层负责对数据库进行操作;
- service层可以通过Spring的声明式事务操作dao层，此外还可以访问NoSQL



## 第一个SpringMVC项目

（1）创建一个Maven项目，此时其项目结构为：

![](https://user-gold-cdn.xitu.io/2020/6/1/1726eac003bfa614?w=475&h=326&f=png&s=14155)



（2）导入依赖

```xml
 <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <spring-framework.version>5.1.10.RELEASE</spring-framework.version>
  </properties>

  <dependencies>
  <!--spring相关-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring-framework.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring-framework.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring-framework.version}</version>
    </dependency>
    <!-- servlet -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>

    <!-- jsp -->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.1</version>
      <scope>provided</scope>
    </dependency>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
```

（3）在web.xml中配置相关控制器

```xml
 <display-name>Archetype Created Web Application</display-name>
 
  <!--  解决中文乱码的过滤器-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>

  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  
<!--  前端控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<!--    给DispatcherServlet类对象的属性contextConfigLocation传递值classpath
        这样就可以加载配置文件springmvc.xml
-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
<!--  一般来说Servlet是在发送请求时才创建，定义该标签则表示是在服务器启动后就创建  -->
    <load-on-startup>1</load-on-startup>
  </servlet>
<!--  所有请求都会被SpirngMVC拦截-->
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
 <!-- 此处/ 和/*的区别在于：在< url-pattern > / </ url-pattern >中，.jsp不会进入DispatcherServlet
	而< url-pattern > /* </ url-pattern > 会匹配 *.jsp，会出现返回 jsp视图 时再次进入spring的DispatcherServlet 类，导致找不到对应的controller而报404错。
-->
  </servlet-mapping>
```

（4）在resources文件夹下创建配置文件spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                 http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
                 http://www.springframework.org/schema/context
                 http://www.springframework.org/schema/context/spring-context-4.2.xsd
               http://www.springframework.org/schema/mvc
               http://www.springframework.org/schema/mvc/spring-mvc.xsd">

<!--    注解扫描。由IOC容器统一管理-->
    <context:component-scan base-package="com.jnju"/>
    <!--    开启SpringMVC框架注解的支持-->
    <mvc:annotation-driven/>

<!--    视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
<!--        视图文件所在目录-->
        <property name="prefix" value="/WEB-INF/views/"/>
<!--       视图文件的后缀名  -->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

（5）Controller

```java
@Controller
@RequestMapping(path = "/test")
public class HelloController {

    @RequestMapping(path = "/hello")
    public String sayHello(Model model) {
        model.addAttribute("msg", "halo SpringMVC");
        //加上配置文件spring-mvc.xml里的前缀WEB-INF/views变成WEB-INF/views/success.jsp
        return "success";	
    }
}
```

（5）首页index.jsp 以及显示成功页面success.jsp

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>首页</title>
</head>
<body>
    <h2>Hello World!</h2>
    <a href="/test/hello">跳转示例</a>
</body>
</html>
```

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>成功</title>
</head>
<body>
    <h3>成功跳转到此页面</h3>
    ${msg}

</html>
```

（6）配置Tomcat

![](https://user-gold-cdn.xitu.io/2020/6/19/172ca67ba13e7db9?w=943&h=787&f=png&s=48029)

![](https://user-gold-cdn.xitu.io/2020/6/19/172ca67c29116895?w=932&h=788&f=png&s=25442)



**运行效果**

![](https://user-gold-cdn.xitu.io/2020/6/19/172ca67a40491c3f?w=329&h=198&f=png&s=12629)

点击连接后：

![](https://user-gold-cdn.xitu.io/2020/6/19/172cab5d47f8fb67?w=330&h=192&f=png&s=14693)



### 示例流程

（1）当我们点击链接后，此时url变为http://localhost:8080/test/hello，其含义是请求位于服务器http://localhost:8080上的SpringMVC站点上的test/hello控制器。

（2）Spring的web框架围绕前端控制器**DispatcherServlet**而设计的，它的作用是将请求分发到不同的Controller上。此时它会调用处理器映射器HandlerMapping，HandlerMapping根据请求url查找Handler。

（3）HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器，比如例子中被查找控制器为：test/hello。

（4）HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler。

（6）Handler让具体的Controller执行请求。

（7）Controller将具体的执行信息返回给HandlerAdapter，比如ModelAndView。

（8）HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。

（9）DispatcherServlet调用视图解析器ViewResolver来解析HandlerAdapter传递的逻辑视图名。

（10）视图解析器将解析的逻辑视图名传给DispatcherServlet。

（11）DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图。

（12）最终视图呈现给用户



## 跳转页面

![](https://user-gold-cdn.xitu.io/2020/6/19/172cb2a94a30fc0e?w=327&h=269&f=png&s=10516)

在前面的例子中，页面跳转是通过返回success字符串来实现的，由于我们在springmvc.xml 中已经配置了视图名的前缀和后缀，因此前端控制器实际获得的视图名为WEB-INF/views/success.jsp。

通常情况下为了保证页面安全，我们会把网站相关的页面放在WEB-INF文件夹保护起来，因为放在 `WEB-INF`文件夹下的页面没有办法通过地址栏直接访问，只能通过后台的跳转来间接的访问。而将引导（欢迎）页面index放在webapp文件夹这一层，以便用户搜索。

### 带返回值的方法

方法的返回值除了返回一个可以被视图解析器解析的视图名以外，还可以返回 含有 redirect 或 forward 标签的字符串来实现重定向和转发。

#### 返回类型是字符串

（1）引导页面

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<%
    // 转发一个 "/login" 请求给后台的HelloController的toLogin方法
    request.getRequestDispatcher("/test/login").forward(request, response);
%>
```

（2）处理请求

```java
@Controller
@RequestMapping(path = "/test")
public class HelloController {
    @RequestMapping(path = "/hello")
    public String sayHello(Model model) {
        model.addAttribute("msg", "halo SpringMVC");
        return "success";
    }

    @RequestMapping(path = "/ForwardAndRedirect")
    public String testForwardAndRedirect(String uname, String pwd, Model model) throws Exception {
        if(uname != null && pwd != null) {
            if("admin".equals(uname) && "123".equals(pwd)) {
                return "redirect:/test/success";
            }else {
                return "forward:/test/login";
            }
        }else {
            return "forward:/test/login";
        }
    }

    @RequestMapping("/login")
    public String toLogin() {
        return "login";
    }

    @RequestMapping("/success")
    public String toSuccess() {
        return "success";
    }
}    
```

（3）login页面和success页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>登陆页面</title>
</head>
<body>
    <form action="/test/ForwardAndRedirect" method="post">
        用户名称：<input type="text" name="uname" ><br/>
        用户密码：<input type="password" name="pwd" ><br/>
        <input type="submit" value="提交">
    </form>
    ${msg}
</body>
</html>

```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>成功</title>
</head>
<body>
<h3>成功跳转到此页面</h3>
    ${msg}

</html>
```

**显示效果**

（1）index.jsp引导至登陆页面

![](https://user-gold-cdn.xitu.io/2020/6/19/172caf4e86a5f557?w=315&h=199&f=png&s=11921)

（2）提交后成功登陆

![](https://user-gold-cdn.xitu.io/2020/6/19/172caf4ee7bbfae1?w=342&h=153&f=png&s=13464)

（3）登陆失败后仍跳至登陆页面，多次提交失败时地址不变

![](https://user-gold-cdn.xitu.io/2020/6/19/172cb03bb84dbcb5?w=420&h=240&f=png&s=17405)

当我们在返回的视图名字符串前加上了 redirect 或者 forward ，是不会走视图解析器，而是直接转发或重定向到指定方法。此外由于重定向是客户端重新发送的请求，因此WEB-INF 目录下的页面仍然访问不到。

**为什么登陆成功是重定向到主页面？**

如果使用的是转发，由于转发时客户端只发送1次此请求，登陆成功后跳转到成功页面后，此时地址栏的url仍是提交的请求

![](https://user-gold-cdn.xitu.io/2020/6/19/172cafca5856a1a2?w=419&h=180&f=png&s=16029)

此时如果刷新页面，就会出现二次提交表单的请求，这不仅影响用户体验，还增大服务端的压力。而重定向到成功页面后，地址栏的url是http://localhost:8080/test/success，无论怎么刷新都可以。

**为什么登录失败是转发到登录页面**

在本例中，若登陆失败则会给出提示信息。如果使用重定向（客户端发送两次请求），则页面就无法获取对应的提示信息



#### 返回类型是ModelAndView

ModelAndView是Spring MVC为我们提供的一个类，它有两个方法：
```java
public ModelAndView addObject(Object attributeValue) {		//设置属性，以便在页面中展示数据
        this.getModelMap().addAttribute(attributeValue);
        return this;
}

public void setViewName(@Nullable String viewName) {		//跳转的视图
        this.view = viewName;
}
```



（1）在HelloController类里添加方法

```java
@RequestMapping("/ModelAndView")
public ModelAndView getList() {
    ModelAndView res = new ModelAndView();
    List<User> users = new ArrayList<User>();
    users.add(new User(1, "aaa", 11));
    users.add(new User(2, "bbb", 12));
    users.add(new User(3, "ccc", 33));

    res.addObject("users", users);
    res.setViewName("showlist");
    return res;
}
```

（2）跳转到的showlist页面

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>列表展示</title>
</head>
<body>
    <c:forEach items="${users}" var="user">
        姓名：<c:out value="${user.uname}"></c:out>
        年龄：<c:out value="${user.age}"></c:out>
    </c:forEach>
</body>
</html>
```

（3）发出的请求

```xml
<a href="/test/ModelAndView">测试返回类型是ModelAndView的页面展示</a>
```

**显示效果**

![](https://user-gold-cdn.xitu.io/2020/6/19/172cb15c22ac63dc?w=416&h=136&f=png&s=14321)

### 无返回值方法

我们可以通过HttpServletRequest 和 HttpServletResponse 来进行页面的跳转

```java
@RequestMapping(path = "/ForwardAndRedirect")
public void testForwardAndRedirect(String uname, String pwd, Model model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        if(uname != null && pwd != null) {
            if("admin".equals(uname) && "123".equals(pwd)) {
                model.addAttribute("msg", "呱");
                request.getRequestDispatcher("/WEB-INF/views/success.jsp").forward(request, response);
            }else {
                //重定向无法跳到WEB-INF里的页面
                System.out.println("url:" + request.getContextPath()); //输出为空
//                model.addAttribute("msg", "用户名或密码错误，请重新登陆");
                response.sendRedirect(request.getContextPath() + "/index.jsp");
            }
        }else {
            System.out.println("url:" + request.getContextPath());  //输出为空
//            model.addAttribute("msg", "用户名或密码为空，请重新登陆");
            response.sendRedirect("index.jsp");
        }
    }
```

**显示效果**

（1）登陆页面

![](https://user-gold-cdn.xitu.io/2020/6/19/172cb2766475d683?w=282&h=219&f=png&s=14597)

（2）登陆成功

![](https://user-gold-cdn.xitu.io/2020/6/19/172cb2851a930f42?w=427&h=157&f=png&s=15659)

（3）登陆失败

![](https://user-gold-cdn.xitu.io/2020/6/19/172cb28a17fbb494?w=345&h=193&f=png&s=15804)



## 常用注解

### @RequestMapping注解

该注解除了应用在方法，还可以应用在类上，此时该注解会作用于控制器类的所有处理器方法上，它会作为URL的第一级访问目录。

**使用示例1**

```java
@Controller
@RequestMapping(path = "/test")
public class HelloController {

    @RequestMapping(path = "/hello")
    public String sayHello() {
        System.out.println("halo SpringMVC");
        return "success";
    }
}
```

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>首页</title>
</head>
<body>
    <h2>Hello World!</h2>
    <a href="test/hello">跳转示例</a>
</body>
</html>
```

运行跳转后的URL：

![](https://user-gold-cdn.xitu.io/2020/6/1/1726f1244bf5c5ae?w=291&h=41&f=png&s=3257)

**属性**

（1）value：用于指定请求的URL，它和 path 属性的作用是一样的。它支持接收多个字符串，例如

```
@RequestMapping({"/hello", "/test"})
```

（2）method：指定请求的方式，例如RequestMethod.GET。

（3）params：用于指定限制请求参数的条件，它要求请求参数的key和value必须和指定的相同，否则不会执行逻辑方法。

**使用示例2**

```
//请求参数必须有 accountName并且money 不能是 100；
@RequestMapping(path = "/hello", params = {"accountName", "moeny!100" })
public String sayHello() {
        System.out.println("halo SpringMVC");
        return "success";
}
```

请求的链接：

```
<a href="test/hello?accountName=aaa&money>100">删除账户，金额 100</a>
```



### @RequestParam注解

它可以把类似下方的请求URL中指定名称的参数传给逻辑方法

http://localhost:8080/test/RequestParam?key1=val1&key2=val2

**使用示例**

```java
@RequestMapping(path = "/RequestParam")
public String testRequestParam(@RequestParam("uname") String uname,
                               @RequestParam(value = "age", required = false) Integer age) {
    System.out.println(uname + " is " + age + " years old");
    return "success";
}
```

**属性**

- value：请求URL中参数对应的名称
- required：请求参数中是否必须提供此参数。默认值为true，表示必须提供，如果不提供将报错。



### @RequestBody注解

该注解用于读取Request请求的body部分数据，主要用来接收前端传递给后端的json字符串中的数据。由于GET的请求方式没有请求体，所以使用@RequestBody接收数据时，前端不能使用GET方式提交数据。

**什么是JSON**

JSON是一种轻量级的数据交换格式，**JSON 键值对**是用来保存 JavaScript 对象的一种方式，它的写法是：键/值对组合中的键名写在前面并用双引号 "" 包裹，使用冒号 : 分隔，然后紧接着值：

```
{"name": "jnju"}
{"age": "3"}
{"sex": "男"}
```

> **JSON与JavaScript 的关系**
>
> JSON 是 JavaScript 对象的字符串表示法，它使用文本表示一个 JS 对象的信息，本质是一个字符串。
>
> ```
> var obj = {a: 'Hello', b: 'World'}; //这是一个对象，注意键名也是可以使用引号包裹的
> var json = '{"a": "Hello", "b": "World"}'; //这是一个 JSON 字符串，本质是一个字符串
> ```

> 在一个处理请求的方法参数里，@RequestBody注解最多只能存在一个

**使用示例**

（1）请求链接及JSON数据

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false"  %>
<html>
<head>
    <title>登陆页面</title>
    <script src="js/jquery.min.js"></script>
    <script>
        //页面加载，绑定点击事件
        $(function () {
            $("#btn").click(function () {
                //发送ajax请求
                $.ajax({
                    //编写json格式数据
                    url:"testReturn/testAjax",
                    contentType:"application/json;charset=UTF-8",
                    data:'{"id":11, "uname": "on1","age":"20"}',
                    dataType:"json",
                    type:"post",
                    success:function (data) {
                        //data是服务器响应的json的数据
                    }
                })
            });
        });
    </script>
</head>
<body>
    <button id="btn">发送ajax请求</button>
</body>
</html>
```

（2）处理方法

```java
@RequestMapping("/testAjax")
public void testAjax(@RequestBody String body) {
    System.out.println(body);
}
```

（3）控制台输出：

```
{"id":11, "uname": "on1","age":"20"}
```

此外还可以将自定义类User作为接收对象，SpringMVC会智能的将符合要求的数据装配进User对象中

```java
public void testAjax(@RequestBody User body) 
```



### @ResponseBody注解

该注解将controller的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到response对象的body区，通常用来返回JSON数据或者是XML数据。

该注解可以放在返回类型前面或方法上，它可以将返回值放在response体内

**使用示例**

（1）请求链接

```xml
<a href="test/ResponseBody">测试ResponseBody</a>
```

（2）处理方法：

```java
@RequestMapping(path = "/ResponseBody")
@ResponseBody
public String testResponseBody(HttpServletRequest request) {
    return "url:" + request.getRequestURI();
}
/* 或者写成这样：
    @RequestMapping(path = "/ResponseBody")
    public @ResponseBody String testResponseBody(HttpServletRequest request)
*/
```

（3）显示页面

![](https://user-gold-cdn.xitu.io/2020/6/1/1726f3ca795e1853?w=378&h=106&f=png&s=10408)



### @PathVariable注解

该注解用于获取请求URL中的占位符，例如 test/PathVariable/jnju 中的字符串jnju。

**注解属性**

value：指定URL中占位符名称

required：表示请求参数中是否必须提供占位符。

**使用示例**

（1）请求链接

```
<a href="test/PathVariable/jnju">测试PathVariable</a><br>
```

（2）处理请求

```java
//如果方法参数名与占位符名相同，则可以去点value属性，此处就可以去掉。
@RequestMapping(path = "/PathVariable/{uname}")
public String testPathVariable(@PathVariable("uname") String uname) {
    System.out.println(uname);
    return "success";
}
```



### @ModelAttribute注解

#### **注释在方法上**

被@ModelAttribute注释的方法会在此控制层类的每个方法执行前被执行，且该方法的返回值会被绑定到Model对象。

**使用示例**

（1）请求连接

```jsp
<a href="test/ModelAttribute?id=11">测试ModelAttribute</a><br>
```

（2）处理请求

```java
@RequestMapping(path = "/ModelAttribute")
public String testModelAttribute(User user) {
    System.out.println(user);
    user.setUname("咕");
    System.out.println(user);
    return "success";
}
//  当注解指定属性名，比如@ModelAttribute("myUser")，
//则除了等价于model.addAttribute("user",user)，同时还等价于model.addAttribute("myUser",user);
@ModelAttribute			
public User getUserByID(Model model) {
    System.out.println("getUserByID方法执行");
    User user = new User(11, "3gu", 22);
    return user; //等价于model.addAttribute("user",user);
}
```

（3）跳转到的success页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>成功</title>
</head>
<body>
<h3>成功跳转到此页面</h3>
    ${user.id}
    ${user.uname}
    ${user.age}

</html>
```

控制台输出：

```
getUserByID方法执行
User{id=11, uname='3gu', age=22}
User{id=11, uname='咕', age=22}
```

点击链接后：

![](https://user-gold-cdn.xitu.io/2020/6/19/172cba2f1ac76f1f?w=431&h=201&f=png&s=16793)

**@ModelAttribute和@RequestMapping注解同一个方法**

此时@RequestMapping注解的属性value值一方面作为请求的路由部分，另一方面则表示要跳转的视图名一部分；此时方法的返回值不再是跳转的视图名，而是作为键值对里的value放在model里，键值对的key值则是@ModelAttribute指定的属性名。

**示例**

（1）请求链接

```
<a href="test/ModelAttribute2">测试ModelAttribute2</a><br>
```

（2）处理请求

```java
@Controller
@RequestMapping(path = "/test")
public class HelloController {
/*
    此时要跳转的页面url是http://localhost:8080/test/ModelAttribute2
    res-success作为键值对被放在model里
*/
    @RequestMapping(path = "/ModelAttribute2")
    @ModelAttribute("res")
    public String testModelAttribute2() {
        return "success";
    }
}
```

视图结构：

![](https://user-gold-cdn.xitu.io/2020/6/19/172cbafe26d5bb7a?w=338&h=318&f=png&s=12825)



点击链接后页面显示：

![](https://user-gold-cdn.xitu.io/2020/6/6/172873d8c4630ba1?w=398&h=175&f=png&s=14471)



#### 注释在方法参数上

（1）请求连接

```
<a href="test/ModelAttribute3">测试ModelAttribute3</a><br>
```

（2）处理请求

```java
@ModelAttribute("userKey")
public User getUserByID(Model model) {
     System.out.println("getUserByID方法执行");
     User user = new User(11, "3gu", 22);
     model.addAttribute("newStr", "3gu");
     
     return user; //等价于model.addAttribute("user",user)和model.addAttribute("userKey",user)
}
    
@RequestMapping(path = "/ModelAttribute3")
public String testModelAttribute3(@ModelAttribute("userKey") User user, @ModelAttribute("newStr") String str) {
     System.out.println(user);
     System.out.println(str);
     return "success";
}
```

控制台输出：

```
getUserByID方法执行
User{id=11, uname='3gu', age=22}
3gu
```















