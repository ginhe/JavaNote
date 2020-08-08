#  代理模式

AOP的底层机制是动态代理，因此我们需要先了解代理模式。代理模式又分为静态代理和动态代理。

**什么是代理模式**

在我们现实生活中的购物场景中，我们通常是从代理商那里购买产品，而不是从厂家那里购买。

![](https://user-gold-cdn.xitu.io/2020/6/18/172c6a4f3a3a8236?w=554&h=321&f=png&s=28240)

而代理模式的思想也是类似的，它分为几个角色：

- 抽象角色：一般是接口或抽象类
- 真实角色 : 被代理的角色
- 代理角色 : 代理真实角色 ; 它在代理真实角色的基础上 , 一般会做一些附属的操作。
- 客户：使用代理角色进行一些操作。

### 静态代理

以电影上映为例：电影是电影公司委托给电影院进行播放的，而电影院在提供播放电影的服务之外，还可以提供其他服务：例如售卖零食等。现在我们用代理模式来实现这个过程

（1）抽象角色：电影接口，它描述了它的功能——播放电影

```java
public interface Movie {
    void play();
}
```

（2）被代理类：实现电影接口的动作片电影：

```java
public class ActionMovie implements Movie {
    public void play() {
        System.out.println("正在播放动作电影《终结者》");
    }
}
```

（3）代理类：电影院：

```java
public class Cinema implements Movie{
    ActionMovie am;
    public Cinema(ActionMovie am) {
        this.am = am;
    }


    public void play() {
        playAdvertisement(true);
        am.play();
        playAdvertisement(false);
    }

    public void playAdvertisement(boolean isStart){
        if ( isStart ) {
            System.out.println("电影马上开始，开始售卖零食");
        } else {
            System.out.println("电影即将结束，开始关门");
        }
    }

}
```

（4）测试方法：

```java
@Test
public void testProxy() {
    Movie movie = new Cinema(new ActionMovie());
    movie.play();
}
```

可以发现，代理模式可以让代理类（电影院）在不修改被代理对象（动作片电影类）的基础上，加上一些新功能。而所谓的静态代理是指：代理类（电影院类）的.class文件在运行前就已经存在。

**静态代理的好处**

- 公共业务由代理完成，实现了业务的分工。并且公共业务发生扩展时更加方便和集中；
- 真实角色不需要关注公共事情。

**缺点**

如果真实对象增加，比如此处电影院还可以代理播放其他类型的电影，那么就要在代理类Cinema里原有的基础上添加代码，这使得工作量大大增加。



### 动态代理

在动态代理中，代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手动编写源码。

同样以电影院上映为例，保持电影接口，动作片电影类不变，修改一下代理类：

（1）代理类不再是实现Movie接口，而是继承一个InvocationHandler类，表明它是一个动态代理类

```java
public class Cinema implements InvocationHandler {
   Object movie;

    public Cinema(Object movie) {
        this.movie = movie;
    }

    public void playAdvertisement(boolean isStart){
        if ( isStart ) {
            System.out.println("即将开始播放，开始售卖零食");
        } else {
            System.out.println("即将结束播放，开始关门");
        }
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        playAdvertisement(true);
        method.invoke(movie, args);
        playAdvertisement(false);
        return null;
    }

}
```

（2）测试方法

```java
    @Test
    public void testProxy() {
        InvocationHandler movie = new Cinema(new ActionMovie());
        //代理类
        Movie movieProxy  = (Movie) Proxy.newProxyInstance(ActionMovie.class.getClassLoader(),
                ActionMovie.class.getInterfaces(), movie);
        movieProxy.play();
    }
```



#### **相关类**

上面的代码是通过Proxy类的静态方法newProxyInstance ()来动态创建代理类。

> Proxy提供了创建动态代理类和实例的静态方法，它同时也是由这些方法创建而来的所有动态代理类的父类。

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
//loader对象表示类加载器;interfaces对象表示代理接口;
```

**InvocationHandler类是什么**

InvocationHandler 是一个接口，每个代理的实例都有一个与之关联的InvocationHandler实现类（本例的电影院对象）。当代理的方法被调用时，代理便会通知和转发给内部的InvocationHandler实现类。其实现类可以通过接口方法**invoke**来调用被代理类（本例的动作电影类）的方法。

```java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
    //代理对象proxy；代理对象调用的方法method；调用方法的参数args
}
```



**扩展案例**

在原来案例的基础上，我们再增加需求：电影院还可以播放悬疑片：

```java
public class SuspenseMovie implements Movie {
    public void play() {
        System.out.println("正在播放《惊魂记》");
    }
}
```

测试方法：

```java
    @Test
    public void testProxy() {
        InvocationHandler actionMovie = new Cinema(new ActionMovie());
        InvocationHandler suspenseMovie = new Cinema(new SuspenseMovie());
        //代理类
        Movie actionMovieProxy   = (Movie) Proxy.newProxyInstance(ActionMovie.class.getClassLoader(),
                ActionMovie.class.getInterfaces(), actionMovie);
        Movie susMovieProxy = (Movie) Proxy.newProxyInstance(
                SuspenseMovie.class.getClassLoader(),
                SuspenseMovie.class.getInterfaces(), suspenseMovie);

        actionMovieProxy .play();
        susMovieProxy.play();
    }
```

相比于只能代理某一角色的静态代理，动态代理可以代理某一类业务（多个类），比如本例中Cinema代理的是电影接口的实现类，它不会更改原有代码。



# AOP

## 什么是AOP

### **纵向抽取方式**

在以往我们编写的代码中，如果一段代码是重复的，则可以将重复代码抽取成一个方法，等需要使用的时候再继承那个类即可。比如一些监视性能的方法代码。但这种抽取成类方法的纵向抽取方式还是会出现重复代码，如下所示：

![](https://user-gold-cdn.xitu.io/2020/5/24/172445b032272efa?w=450&h=407&f=png&s=170574)

其中注释1，2为业务代码，斜体代码为方法性能监视代码，黑色粗体代码为事务开始和提交代码。业务代码夹杂在重复非业务性代码之中。这些重复非业务代码依附在业务类方法*`removeTopic`*，*`createForum`*中，无法将其转移到其他地方。整个方法的大致结构如下：

![](https://user-gold-cdn.xitu.io/2020/5/24/172445d98d7b6c2b?w=661&h=191&f=png&s=57327)

### **面向切面编程AOP**

针对上述问题的一种解决方案是AOP：通过横向抽取机制将分散在业务类方法里的重复非业务代码抽取到各个独立的模块：

![](https://user-gold-cdn.xitu.io/2020/5/24/17244742e716db44?w=476&h=313&f=png&s=83155)

从而降低业务逻辑各个部分之间的耦合度，提高程序的可重用性，同时提高了开发的效率。

>AOP的另一种直观图（图源自[狂神说Spring07：AOP就这么简单](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484138&idx=1&sn=9fb187c7a2f53cc465b50d18e6518fe9&chksm=ce610449f9168d5f159f175de5b7f5fb544633e044275e59cea58c071344368f1fee50a73ccc&scene=21#wechat_redirect))：
>
>![](https://user-gold-cdn.xitu.io/2020/6/18/172c7278abbbed9f?w=1162&h=643&f=png&s=401041)



## AOP的原理

Spring AOP使用两种代理机制：基于JDK的动态代理和基于CGLib的动态代理。此处我们以上面提到的逻辑方法作为例子。

### JDK的动态代理

（1）被代理类ForumServiceImpl，它负责实现业务代码。

![](https://user-gold-cdn.xitu.io/2020/5/24/17244e468fb2f605?w=953&h=615&f=png&s=489082)

（2）被移除的重复非业务的性能监视代码放在代理类PerformanceHandler中，该类负责性能监视的相关方法并通过method.invoke()语句中的Java反射机制间接调用了业务方法。

![](https://user-gold-cdn.xitu.io/2020/5/24/17244e602f112e44?w=959&h=377&f=png&s=311386)

（3）测试方法，它通过newProxyInstance ()方法来动态创建ForumService的代理类：

![](https://user-gold-cdn.xitu.io/2020/5/24/17244eacdd42891a?w=801&h=355&f=png&s=252786)



### CGLib动态代理

使用JDK创建代理有一个限制：它只能为接口创建代理实例，比如上例是为接口ForumService创建的。对于一些没有通过接口定制业务方法的类，CGLib可以帮助我们。它采用底层的字节码技术，为一个类创建子类，在子类中采用方法拦截的技术拦截所有父类方法的调用并插入横切逻辑。

如下所示，CglibProxt作为代理类 实现了org.springframework.cglib.proxy.MethodInterceptor接口，该接口是一个方法拦截器，代理类可以通过接口方法*`intercept`*来拦截被代理类方法的调用，并指定拦截方法的顺序中添加其他操作。

*`intercept`*方法的参数obj表示被代理类实例，method为被代理类方法的反射对象，args为方法的动态参数，proxy为被代理类实例。

代理类的属性Enhancer类对象主要用于为 被代理类生成一个动态子类，可以在被代理类方法执行前后中执行一些其他操作。

![](https://user-gold-cdn.xitu.io/2020/5/24/1724595dc55b5388?w=969&h=399&f=png&s=341080)

测试方法如下：

![](https://user-gold-cdn.xitu.io/2020/5/24/17245a25e52b32f5?w=782&h=133&f=png&s=94090)



**其他示例**

我们再以电影上映为例，尝试一下CGLib动态代理

（1）被代理类：

```java
public class ActionMovie{
    public void play() {
        System.out.println("正在播放电影《终结者》");
    }
}
```

（2）代理类

```java
public class MovieProxy implements MethodInterceptor {
    private Enhancer enhancer = new Enhancer();
    public Object getProxy(Class clazz) {
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }

    public void playAdvertisement(boolean isStart){
        if ( isStart ) {
            System.out.println("即将开始播放，开始售卖零食");
        } else {
            System.out.println("即将结束播放，准备关门");
        }
    }

    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        playAdvertisement(true);
        Object res = methodProxy.invokeSuper(o, objects);
        playAdvertisement(false);
        return  res;
    }
}
```

（3）测试方法：

```java
@Test
public void test(String[] args) {
        //生成被代理类ActionMovie的代理实例movieProxy
        MovieProxy movieProxy = new MovieProxy();
        ActionMovie actionMovie = (ActionMovie)movieProxy.getProxy(ActionMovie.class);
        actionMovie.play();
}
```

> 需要注意的是：由于CGHLib采用动态创建子类的方式生成代理对象，因此不能对被代理类中的final或private方法进行代理。



**两种方式的使用场景**

Spring AOP默认是使用JDK动态代理，如果被代理类没有接口则会使用CGLib代理。如果是单例类最好使用CGLib代理，否则最好使用JDK代理。这是因为JDK在创建代理对象时的性能要高于CGLib代理，而生成代理对象的运行性能却比CGLib的低。



## AOP的术语

**连接点**

连接点是程序执行的某个特定位置，如类开始初始化前，类初始化后，类的某个方法调用前/后，方法抛出异常后。一个类或一段程序代码拥有一些边界性质的特定点，这些特定点称为连接点。Spring仅支持方法的连接点，即仅能在方法调用前/后，方法抛出异常时的这些程序执行点插入其他操作。

**切点**

每一个程序类都拥有多个连接点，而AOP是通过“切点”定位到特定的连接点。以数据库查询为例，连接点相当于数据库中的记录，切点则相当于查询条件。一个切点可以匹配多个连接点。

**增强(Advice)**

增强描述了插入目标类连接点上的一段程序代码，此外它还拥有和连接点相关的信息，即执行点方位。Spring提供的增强接口都是带方位名的，如表示方法调用前的位置BeforeAdvice，表示方法返回后的位置AfterReturningAdvice，ThrowsAdvice等，

**织入**

织入是指将增强添加到目标类的具体连接点的过程，Spring采用的是动态代理织入方式，即在运行期为目标类添加增加生成子类。

**代理（Proxy）**

一个类被AOP织入增强后，就产生一个结果类，它一个结合原类和增强逻辑的代理类。根据不同的代理方式，代理类既可能是和原类具有相同接口的类，也可能是原类的子类。

**切面（Aspect）**

切面由切点和增强组成，它包括横切逻辑的定义和连接点的定义。Spring AOP就是负责实施切面的框架，它将切面定义的横切逻辑（比如前面提到的性能监视代码）织入切面所定义的连接点中。



## Spring的AOP支持

- 基于代理的经典SpringAOP；
- 基于XML的AOP；
- 基于注解的AOP；

此处只介绍后两种。

### 基于XML的AOP

该方式通过配置文件来定义切面、切入点及声明通知，而所有的切面和通知都必须定义在 < aop:config> 元素中。

> 在使用AOP织入前，需要导入依赖：
>
> ```xml
>     <!--aspectj支持-->
>     <dependency>
>       <groupId>org.aspectj</groupId>
>       <artifactId>aspectjweaver</artifactId>
>       <version>1.8.3</version>
>     </dependency>
> ```

**示例**

（1）service接口和实现类：

```java
public interface UserService {
     void search();
}
```

```java
@Service("UserService")
public class UserServiceImpl implements UserService {

    public void search() {
        // 可以通过 int a = 2 / 0; 语句来模拟异常，此时后置通知方法不会执行。
        System.out.println("业务操作：查询用户");
    }
}
```

（2）增强类：一个模拟记录的日志类

```java
@Component
public class MyLog  {
    //  前置通知
    public void beforePrintLog() {
        System.out.println("前置通知");
    }
    //  后置通知，在切入点方法正常执行后执行，它和异常通知只能执行一个
    public void afterPrintLog() {
        System.out.println("后置通知");
    }
    //   异常通知
    public void ThrowingPrintLog() {
        System.out.println("异常通知");
    }
    //    最终通知，无论切入点方法是否正常执行它都会在其后面执行
    public void finallyPrintLog() {
        System.out.println("最终通知");
    }
    //环绕通知里需要我们手动执行切入点方法search
    public Object aroundPrintLog(ProceedingJoinPoint pjp) {
        Object res = null;
        try {
            Object[] args = pjp.getArgs();  //得到切入点方法的参数
            
            System.out.println("环绕通知里的前置通知");
            res = pjp.proceed(args);    //调用切入点方法search
            
            System.out.println("环绕通知里的后置通知");
            return res;
        }catch (Throwable t) {
            System.out.println("环绕通知里的异常通知");
            throw new RuntimeException(t);
        }finally {
            System.out.println("环绕通知里的最终通知");
        }
    }
}
```

（3）配置文件aop.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">

    <context:component-scan base-package="com.jnju"/>

    <!--
切入点表达式：
    使用到的关键字为： execution
    标志表达式写法为：访问修饰符 返回值 包名.类名.方法名(参数列表)。
    例如 "execution(public void com.on1.service.impl.AccountServiceImpl.saveAccount())
   全通配写法：
        （1）访问修饰符可以省略；
        （2）返回值可以使用通配符*代替，即任意返回值都可以。此例是* com.on1.service.impl.AccountServiceImpl.saveAccount()；
        （3）包名也可使用通配符*，有几级包就要写几个*。此例是* *.*.*.*.AccountServiceImpl.saveAccount()
        此外包名还可以使用..表示当前包及其子包。此例是 * *..AccountServiceImpl.saveAccount()
        （4）类名和方法名都可以使用*来实现通配。此例是 * *..*.*()，此处表示的是无参方法
        （5）对于参数列表，若是基本类型则直接写名称，比如int，如果是引用类型则写包名.类名，比如java.lang.String。
        以另一个方法updateAccount(int accountID)为例，则为：* *..*.*(int)
        此外参数列表也可通过*表示有参数，其类型是任意的。
   实际开发中切入点表达式的通常写法：切换到某一类的所有方法，比如 * com.on1.service.impl.*.*(..)
-->
    <aop:config>
        <!-- 配置切入点表达式，此标签写在aop:aspect标签内部则只能是当前切面使用，写在外部则是所有切面可用-->
        <aop:pointcut id="pt1" expression="execution(* com.jnju.service.impl.*.*(..))"/>
        <!-- 配置切面-->
        <aop:aspect id="logAdvice" ref="myLog">
            <aop:before method="beforePrintLog" pointcut-ref="pt1"/>
            <aop:after-returning method="afterPrintLog" pointcut-ref="pt1"/>
            <aop:after-throwing method="ThrowingPrintLog" pointcut-ref="pt1"/>
            <aop:after method="finallyPrintLog" pointcut-ref="pt1"/>
           <aop:around method="aroundPrintLog" pointcut-ref="pt1"/>
        </aop:aspect>
    </aop:config>
</beans>
```

（4）测试方法

```java
    @Test
    public void testAOP() {
        ApplicationContext context = new ClassPathXmlApplicationContext("aop.xml");
        UserService userService = (UserService) context.getBean("UserService");

        userService.search();
    }
```

输出：

```
前置通知
环绕通知里的前置通知
业务操作：查询用户
环绕通知里的后置通知
环绕通知里的最终通知
最终通知
后置通知
```

最后再给出xml文件里的相关标签：

![](https://user-gold-cdn.xitu.io/2020/5/25/1724b4036b759c41?w=813&h=790&f=png&s=129095)



### 基于注解的AOP

在开始之前需要开启注解扫描：

```xml
<aop:aspectj-autoproxy/>
```

例子同样不变，只需修改增强类MyLog

```java

@Component
@Aspect
public class MyLog  {

    @Pointcut("execution(* com.jnju.service.impl.*.*(..))")
    private void pt1(){}        //切入点表达式

    //  前置通知
    @Before("pt1()")
    public void beforePrintLog() {
        System.out.println("前置通知");
    }
    //  后置通知，在切入点方法正常执行后执行，它和异常通知只能执行一个
    @AfterReturning("pt1()")
    public void afterPrintLog() {
        System.out.println("后置通知");
    }
    //   异常通知
    @AfterThrowing("pt1()")
    public void ThrowingPrintLog() {
        System.out.println("异常通知");
    }
    //    最终通知，无论切入点方法是否正常执行它都会在其后面执行
    @After("pt1()")
    public void finallyPrintLog() {
        System.out.println("最终通知");
    }
    @Around("pt1()")
    public Object aroundPrintLog(ProceedingJoinPoint pjp) {
        Object res = null;
        try {
            Object[] args = pjp.getArgs();  //得到切入点方法的参数
            // 如果环绕通知操作（此处的输出语句）写在调用方法前，则表示是前置通知
            System.out.println("环绕通知里的前置通知");
            res = pjp.proceed(args);    //调用切入点方法
            // 如果环绕通知操作（此处的输出语句）写在调用方法之后，则表示是后置通知
            System.out.println("环绕通知里的后置通知");
            return res;
        }catch (Throwable t) {
            // 如果环绕通知操作（此处的输出语句）写在捕捉异常，则表示是异常通知
            System.out.println("环绕通知里的异常通知");
            throw new RuntimeException(t);
        }finally {
            // 如果环绕通知操作（此处的输出语句）写在finally语句，则表示是最终通知
            System.out.println("环绕通知里的最终通知");
        }
    }
}
```

输出不变。



# 参考资料

《精通Spring4.x 企业应用开发实战》