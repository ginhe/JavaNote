# Spring是什么

Spring是一个开源的轻量级框架，它的目标是使J2EE开发变得更容易。

它是一个由7 个定义良好的模块组成的封层架构，核心容器则是6个模块的核心，它定义了创建，配置和管理bean的方式。组成 Spring 框架的每个模块都可以单独存在，或者与其他一个或多个模块联合实现。

![](https://user-gold-cdn.xitu.io/2020/6/17/172c26572891ae71?w=1129&h=555&f=png&s=175475)

每个模块的功能如下：

- **核心容器：**它提供了Spring 框架的基本功能，其主要组件是BeanFactory，它是工厂模式的实现。BeanFactory 使用*控制反转*（IOC） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。
- **Spring上下文：**Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、校验和调度功能。
- **Spring AOP：**通过配置管理特性，Spring AOP 模块直接将面向切面的编程功能 , 集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理任何支持 AOP的对象。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖组件，就可以将声明性事务管理集成到应用程序中。
- **Spring DAO**：JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。
- **Spring ORM**：Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括 JDO、Hibernate 和 iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。
- **Spring Web 模块**：Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring 框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。
- **Spring MVC 框架**：MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

## 什么是IOC

先来看一个例子：

首先我们有一个处理数据库的dao层接口及其实现类：

```java
public interface UserDao {
    void addUser(String name, String pwd);
}
```

```java
public class UserDaoImpl implements UserDao {
   public void addUser(String name, String pwd) {
       //省略与数据库的交互
      System.out.println("成功添加数据[" + name + "," + pwd + "]");
  }
}
```

然后处理逻辑业务的service层接口及其实现类：

```java
public interface UserService {
   public void addUser(String uname, String uPassword);
}
```

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao = new UserDaoImpl();

    public void addUser(String name, String pwd){
        userDao.addUser(name, pwd);
    }
}
```

测试一下：

```java
@Test
public void test(){
   UserService service = new UserServiceImpl();
   service.addUser("root", "123");
}
```

这是我们在还没有Spring框架时的写法，这种写法存在一个依赖关系，即：UserServiceImpl依赖于UserDao接口的实现类，我们主动在UserServiceImpl里创建相关类并调用其方法。假设如果我们又有新的UserDao接口实现类

```java
public class UserDaoMySqlImpl implements UserDao {
   @Override
   public void addUser(String name, String pwd) {
       System.out.println("MySql数据库中成功添加用户[" + name + "," + pwd + "]");
  }
}
```

```java
public class UserDaoOracleImpl implements UserDao {
   @Override
   public void addUser(String uname, String uPassword)  {
       System.out.println("Oracle数据库中成功添加用户[" + name + "," + pwd + "]");
  }
}
```

那么此时我们就需要在UserServiceImpl根据需求来new出相关类。如果需求非常大，那么就需要添加大量代码。我们发现，UserServiceImpl与UserDao接口实现类的耦合度过高，导致相应的维护成本就越高。

```java
public class UserServiceImpl implements UserService {
   private UserDao userDao = new UserDaoOracleImpl();

   @Override
   public void addUser(String uname, String uPassword){
       userDao.addUser(String uname, String uPassword);
  }
}
```

> **什么是耦合和内聚**
>
> 耦合指模块之间的依赖关系。模块间的依赖越多，则表示耦合度越高，相应的维护成本就越高。
>
> 内聚指的是模块内功能之间的联系。模块内功能的联系越紧密，则表示内聚度越高，模块的职责也就越单一

因此我们需要降低耦合，提高内聚，也就是设计原则中的开闭原则和单一职责原则。工厂模式是解决程序间耦合的一种设计模式。可以把所有要创建的对象放在工厂的一个集合里，当需要使用这个对象的时候，直接从工厂里面取出来用就行。

**控制反转IoC**，是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度，**依赖注入DI**则是实现IoC的一种方法。IOC有一个负责创建依赖对象的容器，从从控制和反转两个词分两个方面来理解：

- IoC容器**控制**了对象要获取的外部资源（其它对象或数据等）
- IoC容器查找并注入依赖给对象，对象是被动接受而非主动创建。这就是反转的含义。

我们可以通过xml配置或注解的方式来实现IoC。

### IoC演示

**示例1**

编写一个实体类User

```java
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void sayHi(){
        System.out.println("Hello "+ name);
    }
}
```

右键resources下新建一个Spring的配置文件bean.xml：

![](https://user-gold-cdn.xitu.io/2020/6/17/172c28f51319cf19?w=594&h=138&f=png&s=13721)

```xml
<!--
   id 是bean的唯一标识符,要,如果没有配置id,name就是默认标识符
   如果配置id,又配置了name,那么name是别名,获取Bean时可以用别名获取
   name可以设置多个别名,可以用逗号,分号,空格隔开
   如果不配置id和name,可以根据applicationContext.getBean(.class)获取对象;
-->
    <bean id="person" class="com.jnju.entity.Person">
        <property name="name" value="Spring"/>
    </bean>
```

然后设置容器即可得到输出语句。

```java
@Test
public void testPerson() {
   BeanFactory  bf = new XmlBeanFactory( new ClassPathResource("bean.xml"));
   Person person = (Person) bf.getBean("person");
   person.sayHi();
 }
```

BeanFactory接口是最简单的容器，它主要的功能是为依赖注入 （DI） 提供支持。在 Spring 中，有大量对 BeanFactory 接口的实现。其中，最常被使用的是 **XmlBeanFactory** 类。这个容器从一个 XML 文件中读取配置元数据，由这些元数据来生成一个被配置化的系统或者应用。

在资源宝贵的移动设备或者基于 applet 的应用当中， BeanFactory 会被优先选择。否则，一般使用的是 ApplicationContext。

ApplicationContext 是 BeanFactory 的子接口，也被成为 Spring 上下文。它与和 BeanFactory 类似：加载配置文件中定义的 bean，将所有的 bean 集中在一起，当有请求的时候分配 bean。 此外它还增加了企业所需要的功能，比如，从属性文件中解析文本信息和将事件传递给所指定的监听器。

```java
@Test
public void testPerson() {
   ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");

   Person person = (Person) ac.getBean("person");
   person.sayHi();
 }
```

ApplicationContext 包含 BeanFactory 所有的功能，一般情况下，相对于 BeanFactory，ApplicationContext 会更加优秀。

最常被使用的 ApplicationContext 接口实现：

- **FileSystemXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径。
- **ClassPathXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
- **WebXmlApplicationContext**：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

**示例2**

让我们再用IoC来演示dao-service的示例：

（1）修改UserServiceImpl类：

```java
public class UserServiceImpl implements UserService {
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void addUser(String name, String pwd){
        userDao.addUser(name, pwd);
    }
}
```

（2）在beans.xml添加bean标签：

```xml
    <bean id="MysqlImpl" class="com.jnju.dao.impl.UserDaoImpl"/>
    <bean id="OracleImpl" class="com.jnju.dao.impl.UserDaoOracleImpl"/>

    <bean id="ServiceImpl" class="com.jnju.service.impl.UserServiceImpl">
        <!--此处name的值是是set方法后面的那部分, 首字母小写-->
        <!--引用另外一个bean使用的是ref-->
        <property name="userDao" ref="OracleImpl"/>
    </bean>
```

（3）测试方法：

```java
    @Test
    public void testDaoService() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");

        UserServiceImpl serviceImpl = (UserServiceImpl) ac.getBean("ServiceImpl");
        serviceImpl.addUser("root", "123");
    }
```



### IoC创建对象的方式

**无参构造方法**

（1）在前面的Person类中添加无参方法：

```java
public Person() {
     System.out.println("Person无参构造方法");
 }
```

（2）测试方法不变，运行后会发现输出了无参方法的语句

![](https://user-gold-cdn.xitu.io/2020/6/17/172c2aa2b3d17b30?w=285&h=71&f=png&s=3914)

**数组注入**

```xml
<!-- String[] books -->
<property name="books">
   <array>
        <value>西游记</value>
        <value>红楼梦</value>
         <value>水浒传</value>
   </array>
</property>
```

**list注入**

```xml
<!-- List<String> hobbys --> 
<property name="hobbys">
     <list>
         <value>听歌</value>
         <value>看电影</value>
         <value>爬山</value>
     </list>
 </property>
```

**Map注入**

```java
<!-- Map<String,String> card  -->
<property name="card">
     <map>
         <entry key="中国邮政" value="456456456465456"/>
         <entry key="建设" value="1456682255511"/>
     </map>
 </property>
```

**set注入**

```java
<!-- Set<String> games -->
<property name="games">
     <set>
         <value>LOL</value>
         <value>BOB</value>
         <value>COC</value>
     </set>
</property>
```



**有参构造方法**

（1）Person类

```java
public class Person {
    private String name;

    public Person(String name) {
        System.out.println("Person有参构造方法");
        this.name = name;
    }

    public Person() {
        System.out.println("Person无参构造方法");
    }
    //get/set，sayHi方法不变
}
```

（2）beans.xml：

```xml
<bean id="person" class="com.jnju.entity.Person">
     <constructor-arg name="name" value="sejy"/>
<!--  根据下标设置  <constructor-arg index="0" value="sejy"/>-->
<!--   根据参数类型设置 <constructor-arg type="java.lang.String" value="sejy"/>-->
</bean>
```

（3）测试方法不变，输出：

![](https://user-gold-cdn.xitu.io/2020/6/17/172c2af65a372538?w=317&h=77&f=png&s=4083)



# Bean

构成应用程序的支柱并由Spring IoC容器管理的对象称为bean，所有的bean统一放在context的上下文（即ApplicationContext）中管理。

Bean标签的属性如下：

![](https://user-gold-cdn.xitu.io/2020/6/17/172c2b3f7408c25b?w=815&h=275&f=png&s=186266)

## Bean的作用域

Spring 框架支持Bean的五个作用域：singleton、prototype、request、session和global session。

![](https://user-gold-cdn.xitu.io/2020/6/17/172c2b54d49ebc07?w=839&h=152&f=png&s=103644)



其中singleton 是默认的作用域，即IOC容器只会存在一个共享的bean示例。我们可以通过Bean属性scope来指定作用域。

## Bean 生命周期

此处我们以管理者ApplicationContext说明一个Bean的生命周期获得：

1. Bean的建立， 由ApplicationContext读取Bean定义文件，并生成各个实例
2. Setter注入，执行Bean的属性依赖注入
3. BeanNameAware的setBeanName(), 如果实现该接口，则执行其setBeanName方法。若有Bean类实现了org.springframework.context.ApplicationContextAware接口，则执行其setApplicationContext()方法，然后再进行BeanPostProcessors的processBeforeInitialization()。
4. BeanPostProcessor的processBeforeInitialization()，如果有关联的processor，则在Bean初始化之前都会执行这个实例的processBeforeInitialization()方法
5. InitializingBean的afterPropertiesSet()，如果实现了该接口，则执行其afterPropertiesSet()方法
6. Bean定义文件中定义的init-method
7. BeanPostProcessors的processAfterInitialization()，如果有关联的processor，则在Bean初始化之前都会执行这个实例的processAfterInitialization()方法
8. DisposableBean的destroy()，在容器关闭时，如果Bean类实现了该接口，则执行它的destroy()方法
9. Bean定义文件中定义的destroy-method，在容器关闭时，可以在Bean定义文件中使用“destory-method”定义的方法



上述提到的init-method和destroy-method，我们可以在Bean标签的属性里定义：

```xml
 init-method="init" destroy-method="destroy"
```

然后在类里加上相关方法：

```java
    public void init() {
        System.out.println("初始化后的一些列工作");
    }

    public void destroy() {
        System.out.println("清除User对象前的一些列工作");
    }
```



## bean继承

Bean标签可以通过属性parent来指定其父bean，不过其定义的bean之间的父子关系不会传承到类里，但继承的概念是一样的：即子bean的定义可以继承父定义的配置数据。

**示例**

（1）在原有的Person类的基础上，我们再添加Student类：

```java
public class Student {
    private String classNo;
    private String major;
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public void setClassNo(String classNo) {
        this.classNo = classNo;
    }


    public void setMajor(String major) {
        this.major = major;
    }

    @Override
    public String toString() {
        return "Student{" +
                "classNo='" + classNo + '\'' +
                ", major='" + major + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}
```

（2）修改beans.xml

```xml
    <bean id="person" class="com.jnju.entity.Person">
        <property name="name" value="sejy"/>
    </bean>

    <bean id="student" class="com.jnju.entity.Student" parent="person">
        <property name="classNo" value="1"/>
        <property name="major" value="cs"/>
    </bean>
```

（3）测试方法：

```java
    @Test
    public void testStudent() {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");

        Student student = (Student) ac.getBean("student");
        System.out.println(student);
    }
```

此时输出为：Student{classNo='1', major='cs', name='sejy'}



# BeanFactory

**BeanFactory**是一个接口。它定义了`getBean()`，`containsBean()`等管理Bean的通用方法。我们常见的容器都是它的具体实现。下图是其继承关系的uml图

<img src="https://user-gold-cdn.xitu.io/2020/7/10/17336e624a227362?w=1756&amp;h=745&amp;f=png&amp;s=285371" style="zoom: 200%;" />

BeanFactory的默认实现类是DefaultListableBeanFactory

```java
@SuppressWarnings("serial")
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
    
    //存储实例化后的bean。其中BeanDefinition类表示Bean的信息，比如这个 Bean 指向的是哪个类、是否是单例的、是否懒加载
    private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
    
    @Override
	public <T> T getBean(Class<T> requiredType) throws BeansException {
		return getBean(requiredType, (Object[]) null);
	}

	@SuppressWarnings("unchecked")
	@Override
	public <T> T getBean(Class<T> requiredType, @Nullable Object... args) throws BeansException {
		Assert.notNull(requiredType, "Required type must not be null");
		Object resolved = resolveBean(ResolvableType.forRawClass(requiredType), args, false);
		if (resolved == null) {
			throw new NoSuchBeanDefinitionException(requiredType);
		}
		return (T) resolved;
	}
}
```

不过**DefaultListableBeanFactory**只实现了BeanFactory接口5个getBean方法中的2个，其余3个实现放在其父类AbstractBeanFactory里。

```java
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {
 @Override
    public Object getBean(String name) throws BeansException {
        return doGetBean(name, null, null, false);
    }

    @Override
    public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
        return doGetBean(name, requiredType, null, false);
    }

    @Override
    public Object getBean(String name, Object... args) throws BeansException {
        return doGetBean(name, null, args, false);
    }

    public <T> T getBean(String name, Class<T> requiredType, Object... args) throws BeansException {
        return doGetBean(name, requiredType, args, false);
    }

    @SuppressWarnings("unchecked")
    protected <T> T doGetBean(
            final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
            throws BeansException {

        final String beanName = transformedBeanName(name);
        Object bean;

        // 根据beanName尝试从一二三级缓存中获取Bean。该方法和解决循环依赖有关
        Object sharedInstance = getSingleton(beanName);
        //如果是第一次进入该方法，sharedInstance肯定为null。
        if (sharedInstance != null && args == null) {
            //省略打印操作...
            //检测当前Bean是否为FactoryBean
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
        }

        else {
            //判断是否循环依赖
            if (isPrototypeCurrentlyInCreation(beanName)) {
                throw new BeanCurrentlyInCreationException(beanName);
            }

            // 获取父BeanFactory,一般情况下,父BeanFactory为null
            BeanFactory parentBeanFactory = getParentBeanFactory();
            if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
                // Not found -> check parent.
                String nameToLookup = originalBeanName(name);
                if (args != null) {
                    // Delegation to parent with explicit args.
                    return (T) parentBeanFactory.getBean(nameToLookup, args);
                }
                else {
                    // 若没有参数则交给标准的getBean方法
                    return parentBeanFactory.getBean(nameToLookup, requiredType);
                }
            }

            if (!typeCheckOnly) {
                //标记bean已被创建
                markBeanAsCreated(beanName);
            }

            try {
                //获取其父类Bean定义，子类合并父类公共属性
                final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
                checkMergedBeanDefinition(mbd, beanName, args);

                // 获取当前Bean依赖的Bean的名称 ,@DependsOn
                String[] dependsOn = mbd.getDependsOn();
                if (dependsOn != null) {
                    for (String dep : dependsOn) {
                        if (isDependent(beanName, dep)) {
                            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                                    "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
                        }
                        // 如果当前Bean依赖其他Bean,把被依赖Bean注册给当前Bean
                        registerDependentBean(dep, beanName);
                        getBean(dep);
                    }
                }

                // 单例Bean
                if (mbd.isSingleton()) {
                    sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
                        @Override
                        public Object getObject() throws BeansException {
                            try {
                                return createBean(beanName, mbd, args);
                            }
                            catch (BeansException ex) {
                                // 如果出错，则从一级缓存singletonObjects里移除掉，因为它可能存在于那个哈希表中
                                // 并删除所有对该Bean的临时引用
                                destroySingleton(beanName);
                                throw ex;
                            }
                        }
                    });
                    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                }

                else if (mbd.isPrototype()) {
                    // It's a prototype -> create a new instance.
                    Object prototypeInstance = null;
                    try {
                        beforePrototypeCreation(beanName);
                        prototypeInstance = createBean(beanName, mbd, args);
                    }
                    finally {
                        afterPrototypeCreation(beanName);
                    }
                    bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
                }

                else {
                    //如果既不是单例也不是prototye，则获取其Scope值。并创建对应的对象
                    String scopeName = mbd.getScope();
                    final Scope scope = this.scopes.get(scopeName);
                    if (scope == null) {
                        throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
                    }
                    try {
                        Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                            @Override
                            public Object getObject() throws BeansException {
                                beforePrototypeCreation(beanName);
                                try {
                                    return createBean(beanName, mbd, args);
                                }
                                finally {
                                    afterPrototypeCreation(beanName);
                                }
                            }
                        });
                        bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
                    }
                    catch (IllegalStateException ex) {
                        throw new BeanCreationException(beanName,
                                "Scope '" + scopeName + "' is not active for the current thread; consider " +
                                "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                                ex);
                    }
                }
            }
            catch (BeansException ex) {
                cleanupAfterBeanCreationFailure(beanName);
                throw ex;
            }
        }

        // 检查requiredType是否和bean实例类型匹配
        if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
            try {
                return getTypeConverter().convertIfNecessary(bean, requiredType);
            }
            catch (TypeMismatchException ex) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Failed to convert bean '" + name + "' to required type '" +
                            ClassUtils.getQualifiedName(requiredType) + "'", ex);
                }
                throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
            }
        }
        return (T) bean;
    }
}
```





# IOC流程简介（未完成）

我们首先从第一行代码开始：

```
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
```

## 初始化方法

```java
public class ClassPathXmlApplicationContext extends AbstractXmlApplicationContext {
    @Nullable
    private Resource[] configResources;
    
    public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
        this(new String[]{configLocation}, true, (ApplicationContext)null);
    }
    
    public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, @Nullable ApplicationContext parent) throws BeansException {
            
        super(parent);
         // 根据提供的路径，处理成配置文件数组(以分号、逗号、空格、tab、换行符分割)    
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();
        }
    }
}    
```

## refresh方法

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {
    
	public void refresh() throws BeansException, IllegalStateException {
        
        synchronized(this.startupShutdownMonitor) {			//加锁
            
            this.prepareRefresh();	// （1）准备工作，记录下容器的启动时间、标记“已启动”状态、处理配置文件中的占位符
            
            //(2)将配置文件解析成一个个的Bean，并注册到BeanFactory（此时Bean 还没有初始化）
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                //（3）调用BeanFactory的后处理器。它的运行时间是在bean  定义全部加载到容器之后，bean的创建之前。
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                //初始化国际化工具类MessageSource
                this.initMessageSource();
                //初始化事件广播器，用于事件的发布。
                this.initApplicationEventMulticaster();
                this.onRefresh();
                // 注册事件监听器
                this.registerListeners();
                //（4）实例化所有单例，非懒加载的bean
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var9) {
                
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
}
```

我们将refresh()方法分解几部分来讲解：

### **prepareRefresh方法**

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {	
    
	protected void prepareRefresh() {
        //记录启动时间
        this.startupDate = System.currentTimeMillis();
        this.closed.set(false);		//closed:false
        this.active.set(true);		//active：true
        
        if (this.logger.isDebugEnabled()) {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Refreshing " + this);
            } else {
                this.logger.debug("Refreshing " + this.getDisplayName());
            }
        }

        this.initPropertySources();
        //校验xml配置文件
        this.getEnvironment().validateRequiredProperties();
        if (this.earlyApplicationListeners == null) {
            this.earlyApplicationListeners = new LinkedHashSet(this.applicationListeners);
        } else {
            this.applicationListeners.clear();
            this.applicationListeners.addAll(this.earlyApplicationListeners);
        }

        this.earlyApplicationEvents = new LinkedHashSet();
    }
}
```



### obtainFreshBeanFactory方法

`refresh`方法的第二部分是**obtainFreshBeanFactory**方法。该方法会解析所有Spring配置文件（通常放在resources目录下），将所有 Spring 配置文件中的 bean 定义封装成 BeanDefinition，加载到 BeanFactory 中。如果它解析到开启注解扫描：

```xml
<context:component-scan base-package="" /> 
```

那么它会扫描base-package指定目录，并将该目录下使用指定注解的bean定义也封装成BeanDefinition。下面来看一下这个方法的源码：

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
 
        ///初始化BeanFactory,并进行XML文件读取，并将得到
        this.refreshBeanFactory();
        
        return this.getBeanFactory();
}
```

`refreshBeanFactory()`是一个抽象方法，它有两个实现方法：

![](https://user-gold-cdn.xitu.io/2020/7/3/173150b934f664c6?w=805&h=136&f=png&s=20227)

此处的实现是在AbstractApplicationContext的子类**AbstractRefreshableApplicationContext**中。

```java
public abstract class AbstractRefreshableApplicationContext extends AbstractApplicationContext {
    
     protected final void refreshBeanFactory() throws BeansException {
         //如果beanFactory不为空，则清除BeanFactory和里面的实例
            if (this.hasBeanFactory()) {
                this.destroyBeans();
                this.closeBeanFactory();
            }

            try {
               //（1）重新创建的Bean工厂默认为DefaultListableBeanFactory
                DefaultListableBeanFactory beanFactory = this.createBeanFactory();
                beanFactory.setSerializationId(this.getId());
                
                //（2）设置BeanFactory 的两个配置属性：是否允许 Bean 覆盖、是否允许循环引用
                this.customizeBeanFactory(beanFactory);
                //（3）解析xml，并把xml中的标签封装称BeanDefinition对象
                this.loadBeanDefinitions(beanFactory);
                synchronized(this.beanFactoryMonitor) {
                    this.beanFactory = beanFactory;
                }
            } //...
        }
    
    	protected abstract void loadBeanDefinitions(DefaultListableBeanFactory beanFactory)
			throws BeansException, IOException;
}
```

其中（3）的loadBeanDefinitions是一个抽象方法，该方法有四个类实现。本例会跳进**AbstractXmlApplicationContext**类的loadBeanDefinitions方法。

![](https://user-gold-cdn.xitu.io/2020/7/10/1733916647f94a39?w=1494&h=139&f=png&s=32454)



```java
public abstract class AbstractXmlApplicationContext extends AbstractRefreshableConfigApplicationContext {
	
    protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
		}
	}
}
```



#### **解析xml文件**

解析xml文件的具体代码不会展示，只是大致说明下流程：

1. 将xml文件封装成Resource流文件后，然后转成InputSource流文件，再转成document对象。
2. 通过**DefaultBeanDefinitionDocumentReader**类来处理document对象，其中document对象是一个实现了Node接口的组成的树形结构数据。因此**DefaultBeanDefinitionDocumentReader**会通过document.getDocumentElement()方法获取文件的root节点。
3. 获取文件的root节点后，它将要解析的标签分为两类：默认标签（bean，import，alias,beans标签）和自定义标签（context，componet-scan）。
4. 解析完标签内容后，创建**GenericBeanDefinition**类对标签内容进行封装进BeanDefinition。

BeanDefinition接口用来描述一个Bean的信息：包括属性值，构造方法参数值和继承自它的类的更多信息。它的主要功能是允许BeanFactoryPostProcessor 能够检索并修改属性值和别的bean的元数据。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    
    //父类Bean名称
	void setParentName(@Nullable String parentName);
    
	@Nullable
	String getParentName();

    //当前bean的className
	void setBeanClassName(@Nullable String beanClassName);
	
	@Nullable
	String getBeanClassName();
    
	//Bean的作用域
	void setScope(@Nullable String scope);

	@Nullable
	String getScope();

	void setLazyInit(boolean lazyInit);
	
	boolean isLazyInit();
    //...
}  
```
被封装好的BeanDefinition会被DefaultListableBeanFactory注册进beanDefinitionMap。

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
    
    //key为bean的名称，value为bean的具体定义，仅仅是定义，而非实例化好的对象
    private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

    //bean的名称列表
    private volatile List<String> beanDefinitionNames = new ArrayList<>(256);
    
	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {
		//...

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
                //判断beanDefinition是否同时具有overrideMethod和factoryMethod两个属性
                //若是则抛出异常
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}
		//判断BeanDefinition是否注册过
		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {	
            //若已注册过且不允许被覆盖，则抛出异常BeanDefinitionOverrideException
            //否则输出一些日志，然后覆盖已注册的BeanDefinition
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
            //如果bean是否正在处于创建阶段
			if (hasBeanCreationStarted()) {
				// 那就使用CopyAndWrite思想注册新的BeanDefinition
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// 如果没有正在创建，那就无需锁直接注册
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
	}
}
```



### invokeBeanFactoryPostProcessors

`refresh`方法的第三部分是**invokeBeanFactoryPostProcessors**方法。该方法会实例化和调用BeanFactoryPostProcessor包括其子类 BeanDefinitionRegistryPostProcessor）。

BeanFactoryPostProcessor 接口是 Spring 初始化 BeanFactory 时对外暴露的扩展点，Spring IoC 容器允许 BeanFactoryPostProcessor 在容器实例化任何 bean 之前读取 bean 的定义，并可以修改它。

该方法的源码如下

```java
final class PostProcessorRegistrationDelegate {
    
	public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
        
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
}
```



### finishBeanFactoryInitialization

该方法负责完成BeanFactory里bean的初始化。源码如下：

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {

	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// 设置类型转换器
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}
        //...
        //实例化剩下所有的非懒加载单例
        beanFactory.preInstantiateSingletons();
}
```

我们只需关心最后一句：`preInstantiateSingletons`方法

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
		
	@Override
	public void preInstantiateSingletons() throws BeansException {
		//...
        //从ArrayList类型的beanDefinitionNames获取所有的beanName
        List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
        
        for (String beanName : beanNames) {
            //把父BeanDefinition里的属性拿到子BeanDefinition中
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
            //非抽象，单例，懒加载才会实例化
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                //判断是否是FactoryBean类型的bean
				if (isFactoryBean(beanName)) {
					//...
                }else {
                    //进入实例化过程
					getBean(beanName);
				}
            }
        }
    }
}
```

#### doGetBean

进入实例化过程`getBean`方法

```java
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {

	@Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}
	
	//本方法提及到的缓存内容会在循环依赖的章节讲述
    protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
        //处理两者情况：1. 将别名转化成真的beanName；2. 把FactoryBean的前缀"&"给去了
        String beanName = this.transformedBeanName(name);
        //尝试从缓存里取出beanName对应的bean
        Object sharedInstance = this.getSingleton(beanName);	//getSingleton(beanName, true)
        Object bean;
        if (sharedInstance != null && args == null) {
			//...
            //检测当前Bean是否为FactoryBean
            bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
        } else {		//缓存中不存在bean。第一次创建bean会走到这里
            //...
			// 获取父BeanFactory,一般情况下父BeanFactory为null。
            BeanFactory parentBeanFactory = this.getParentBeanFactory();
            //如果存在父BeanFactory且在beanDefinitionMap中没有beanName，那就先去父级容器去查找
            if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
                String nameToLookup = this.originalBeanName(name);
                if (parentBeanFactory instanceof AbstractBeanFactory) {
                    return ((AbstractBeanFactory)parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
                }
				
                return parentBeanFactory.getBean(nameToLookup);
            }
		// 创建的Bean是否需要进行类型验证,一般情况下都不需要
            if (!typeCheckOnly) {
                //标记bean已被创建
                this.markBeanAsCreated(beanName);
            }

            try {
                RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
                this.checkMergedBeanDefinition(mbd, beanName, args);
                //处理依赖的bean
                String[] dependsOn = mbd.getDependsOn();
                String[] var11;
                if (dependsOn != null) {		
                    //若存在依赖的bean则首先递归实例化依赖的bean
                    var11 = dependsOn;
                    int var12 = dependsOn.length;

                    for(int var13 = 0; var13 < var12; ++var13) {
                        String dep = var11[var13];
                       //...
                        this.registerDependentBean(dep, beanName);

                        try {
                            this.getBean(dep);
                        } //...
                    }
                }
				//单例模式下的实例化bean
                if (mbd.isSingleton()) {		//getSingleton(beanName, singletonFactory)
                    sharedInstance = this.getSingleton(beanName, () -> {
                        try {
                            //创建bean
                            return this.createBean(beanName, mbd, args);
                        } catch (BeansException var5) {
                            //从缓存里删除bean
                            this.destroySingleton(beanName);
                            throw var5;
                        }
                    });
                    bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                } 
                //省略对原型模型bean和自定义模型bean的处理
            } 
        }

        //省略对类型是否符合bean的实际类型requiredType的判断
    }

}
```

#### doCreateBean

`doGetBean`方法会进入`createBean()`方法里，然后再进入`doCreateBean`方法

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
		implements AutowireCapableBeanFactory {
		
	@Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		//...
		try {
            
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			//...
			return beanInstance;
            //...
		}
	}
    
    protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args) throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
            //（1）创建bean实例并包装成BeanWrapper对象
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		final Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
			mbd.resolvedTargetType = beanType;
		}

		// MergedBeanDefinitionPostProcessor的应用（如：AutowiredAnnotationBeanPostProcessor）
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
                    //对类里面的注解进行包装
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		//是否需要提前曝光earlySingletonExposure。这段内容会在循环依赖讲解
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
            //（2）属性装配
			populateBean(beanName, mbd, instanceWrapper);
            //（3）实例化操作，比如将原生对象变成代理对象
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		//...
		//循环依赖检查，判断是否需要抛出异常
		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					//...
				}
			}
		}

}
```

先来看一下（1）中的**createBeanInstance**方法：

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    	//反射拿到class对象
		Class<?> beanClass = resolveBeanClass(mbd, beanName);
		//判断class是否为空以及访问权限是否为public
		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		//...
		//若有工厂方法则通过工厂方法创建
		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}
		/*
		由于一个类可能有多个构造函数，所以需要根据配置文件中配置的参数或者传入的参数确定最终调用的构造函数
		判断过程会比较消耗性能，因此Spring会将解析、确定好的构造函数缓存到BeanDefinition中的           
		     resolvedConstructorOrFactoryMethod字段中。
		在下次创建相同bean的时候，直接从RootBeanDefinition中的属性resolvedConstructorOrFactoryMethod缓存的值获取，避         免再次解析。
		*/
		boolean resolved = false;
		boolean autowireNecessary = false;
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
                    //已经解析过class的构造器
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
		if (resolved) {
            //使用已经解析过class的构造器。
			if (autowireNecessary) {
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
                //默认构造器
				return instantiateBean(beanName, mbd);
			}
		}

		//寻根据参数解析来确定构造函数
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    //解析的构造器不为空 || 注入类型为构造函数自动注入 || bean定义中有构造器参数 || 传入参数不为空
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
            //构造器自动注入
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
			return autowireConstructor(beanName, mbd, ctors, null);
		}

		//默认构造器
		return instantiateBean(beanName, mbd);
	}

```

（2）属性装配方法**populateBean**





# 循环依赖

首先需要说明 Spring遇到对象的循环依赖有三种情况：

- 构造器的循环：无法处理，抛出BeanCurrentlylnCreationException异常
- 单例模式下的set循环依赖：通过`三级缓存`处理
- 非单例模式下的循环：无法处理

我们以两个类为例：

```java
@Component
public class A {
  private B b;
  public void setB(B b) {
    this.b = b;
  }
}
@Component
public class B {
  private A a;
  public void setA(A a) {
    this.a = a;
  }
}
```

**循环依赖的流程**

首先我们先说明一下pring创建Bean步骤：

（1）实例化，即new了一个对象，但还未注入属性值

```java
instanceWrapper = createBeanInstance(beanName, mbd, args);
final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
```

（2）属性注入，即填充属性

```
populateBean(beanName, mbd, instanceWrapper);
```

（3）初始化。执行aware接口中的方法和初始化方法，完成`AOP`代理

其次再说明循环依赖的过程：

1. 当Spring 实例化类A后，判断A是否允许提前暴露，若允许则放入三级缓存（一个哈希表）
2. 在属性注入时发现有属性B需要注入，则会触发类B 的实例化（不完全的初始化），同样判断是否提前暴露，若允许则放入三级缓存
3. 同样在属性注入时又触发A的实例化，先后从一二级缓存里取，最后在三级缓存里取到
4. 把从三级缓存里取到的部分完成的A实例放入二级缓存，并删掉三级缓存里的A
5. 把A注入B的属性里，B完成实例化。然后把B放入一级缓存里，删掉B在二，三级缓存
6. 把B注入给A，此时A完成实例化。然后把A放入一级缓存里，删掉A在二，三级缓存

**源码**

我们首先从getBean("a")开始，该方法分为两个作用：

- 创建一个新bean
- 从缓存里获取已被创建的对象。

此时该方法的作用是第一个，它会在`doGetBean()`里调用`getSingleton(beanName, true)`尝试从缓存里获取bean，但获得的是null。然后会调用`getSingleton(beanName,singletonFactory)`。

```java
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {

   public Object getBean(String name) throws BeansException {
        return this.doGetBean(name, (Class)null, (Object[])null, false);
   }
    //doGetBean方法会调用两个不同的getSingleton方法
    protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
        String beanName = this.transformedBeanName(name);
        //尝试从缓存里取出beanName对应的bean
        Object sharedInstance = this.getSingleton(beanName);	//第一个getSingleton方法
        Object bean;
        if (sharedInstance != null && args == null) {
			//...
            //检测当前Bean是否为FactoryBean
            bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, (RootBeanDefinition)null);
        } else {		//缓存中不存在bean。第一次创建bean会走到这里
            //...
			// 获取父BeanFactory,一般情况下父BeanFactory为null。
            BeanFactory parentBeanFactory = this.getParentBeanFactory();
            //如果存在父BeanFactory且在beanDefinitionMap中没有beanName，那就先去父级容器去查找
            if (parentBeanFactory != null && !this.containsBeanDefinition(beanName)) {
                String nameToLookup = this.originalBeanName(name);
                if (parentBeanFactory instanceof AbstractBeanFactory) {
                    return ((AbstractBeanFactory)parentBeanFactory).doGetBean(nameToLookup, requiredType, args, typeCheckOnly);
                }
				
                return parentBeanFactory.getBean(nameToLookup);
            }
		// 创建的Bean是否需要进行类型验证,一般情况下都不需要
            if (!typeCheckOnly) {
                //标记bean已被创建
                this.markBeanAsCreated(beanName);
            }

            try {
                RootBeanDefinition mbd = this.getMergedLocalBeanDefinition(beanName);
                this.checkMergedBeanDefinition(mbd, beanName, args);
                //处理依赖的bean
                String[] dependsOn = mbd.getDependsOn();
                String[] var11;
                if (dependsOn != null) {		
                    //若存在依赖的bean则递归实例化依赖的bean
                    var11 = dependsOn;
                    int var12 = dependsOn.length;

                    for(int var13 = 0; var13 < var12; ++var13) {
                        String dep = var11[var13];
                       //...
                        this.registerDependentBean(dep, beanName);

                        try {
                            this.getBean(dep);
                        } //...
                    }
                }
				//单例模式下的实例化bean
                if (mbd.isSingleton()) {		//第二个getSingleton
                    sharedInstance = this.getSingleton(beanName, () -> {
                        try {
                            //创建bean
                            return this.createBean(beanName, mbd, args);
                        } catch (BeansException var5) {
                            //从缓存里删除bean
                            this.destroySingleton(beanName);
                            throw var5;
                        }
                    });
                    bean = this.getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
                } 
                //省略对原型模型bean和自定义模型bean的处理
            } 
        }

        //省略对 类型是否符合bean的实际类型requiredType的判断
    }
}
```

在getSingleton(beanName,singletonFactory)方法里通过`singletonFactory.getObject();`里转至`createBean`方法创建一个bean

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    
    //存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用。我们称为一级缓存
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);
    //存放已实例化，但尚未填充属性和初始化的bean，我们称这类bean为早期引用。该二级缓存用于解决循环依赖
    private final Map<String, Object> earlySingletonObjects = new HashMap(16);
    //存放进入实例化阶段的Bea的工厂，主要作用是提前暴露这些bean
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);
    //spring在创建一个单例对象前，会将其放入singletonsCurrentlyInCreation。创建完后从里面移除
    private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap(16));
    
	@Nullable
    public Object getSingleton(String beanName) {
        return this.getSingleton(beanName, true);
    }

    //第一个
	@Nullable
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        //...
        //从singletonObjects获取已初始化的bean。
        Object singletonObject = this.singletonObjects.get(beanName);
        
        //如果不存在已初始化的bean，判断要查找的bean是否正在创建中
        if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
            synchronized(this.singletonObjects) {
                
                //尝试从二级缓存earlySingletonObjects里取出bean
                singletonObject = this.earlySingletonObjects.get(beanName);
                
                //如果二级缓存中不存在这个bean并且允许提前曝光，则尝试三级缓存singletonFactory缓存中获取bean
                if (singletonObject == null && allowEarlyReference) {
                  ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                    if (singletonFactory != null) {
                        //此处同样没有成功获取到bean，因此本方法返回null
                        singletonObject = singletonFactory.getObject();
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }

        return singletonObject;
    }
    //第二个
    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "Bean name must not be null");
        synchronized(this.singletonObjects) {
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
               //...
				//（1）在单例对象创建前做标记，将beanName放入singletonsCurrentlyInCreation集合里
                //表示这个单例bean正在被创建。如果同一个单例bean被多次创建会抛出异常
                this.beforeSingletonCreation(beanName);
                boolean newSingleton = false;
                //...
                try {
                    //创建一个bean
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                }//...
                finally {
                    //...
					//创建完成后将对应的beanName从singletonsCurrentlyInCreation移除
                    this.afterSingletonCreation(beanName);
                }

                if (newSingleton) {
                    //创建完后的bean放入一级缓存里
                    this.addSingleton(beanName, singletonObject);
                }
            }

            return singletonObject;
        }
    }
	
    protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
                //将beanName-singletonFactory工厂作为映射添加到三级缓存里
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}
}
```

在上面代码（1）中的的` singletonObject = singletonFactory.getObject();`方法里，该方法最终会调用`createBean`方法，这个方法又会调用doCreateBean方法

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {
    
    @Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
        //...
        try {
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			//...
			return beanInstance;
		}
    }

     protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
            BeanWrapper instanceWrapper = null;
            //...
			//（1）实例化一个bean
            Object bean = instanceWrapper.getWrappedInstance();
            Class<?> beanType = instanceWrapper.getWrappedClass();
         //是否需要提前曝光earlySingletonExposure的条件是：单例&允许循环依赖&当前的bean正在创建中
         //allowCircularReferences表示是否允许循环依赖
         //isSingletonCurrentlyInCreation判断是否在创建beanName
         boolean earlySingletonExposure = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
        if (earlySingletonExposure) {
            //...
            //（2）将创建完的bean添加到三级缓存
			 this.addSingletonFactory(beanName, () -> {
                return this.getEarlyBeanReference(beanName, mbd, bean);
            });
        }
         Object exposedObject = bean;
         try {
             //（3）属性注入
            this.populateBean(beanName, mbd, instanceWrapper);
             //（4）初始化
            exposedObject = this.initializeBean(beanName, exposedObject, mbd);
        } 
     }
    protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
        //...
	}
}
```

在上面的doCreateBean()方法里，a完成实例化并添加进三级缓存，准备执行（3）的属性注入，发现a依赖b，因此Spring又会去`getBean(b)`，然后反射调用setter方法完成属性注入。

`getBean(b)`之后的步骤与a一样，同样在执行（3）的属性注入，发现b依赖a，此时又会再去`getBean(a)`，但此时的a不再是去新建而是从三级缓存里取出

```java
    //第一个
	@Nullable
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        //...
        //从singletonObjects获取已初始化的bean。
        Object singletonObject = this.singletonObjects.get(beanName);
        
        //如果不存在已初始化的bean，判断要查找的bean是否正在创建中
        if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
            synchronized(this.singletonObjects) {
                
                //尝试从二级缓存earlySingletonObjects里取出bean
                singletonObject = this.earlySingletonObjects.get(beanName);
                
                //如果二级缓存中不存在这个bean并且允许提前曝光，则尝试在三级缓存singletonFactory缓存中获取bean
                if (singletonObject == null && allowEarlyReference) {
                  ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                    if (singletonFactory != null) {
                        //（1）调用getEarlyBeanReference获取提前暴露的值
                        singletonObject = singletonFactory.getObject();
                        //放入二级缓存earlySingletonObjects
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        //从三级缓存里移除该bean
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }

        return singletonObject;
    }
```

来看一下（1）处提到的getEarlyBeanReference方法

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
               SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                //（1）调用后置处理器的getEarlyBeanReference
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

如果此处的A不是AOP代理，那么该方法只是返回在实例化阶段创建的对象a。也就是等价于下面的代码：

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    return exposedObject;
}
```

那么如果A是AOP代理的情况，那就代表着我们希望从容器里获取的A是其代理对象而不是A本身。因此在（1）处的ibp对象调用的是AnnotationAwareAspectJAutoProxyCreator的getEarlyBeanReference方法

```java
public Object getEarlyBeanReference(Object bean, String beanName) {
    Object cacheKey = getCacheKey(bean.getClass(), beanName);
    this.earlyProxyReferences.put(cacheKey, bean);
    // 如果需要代理，返回一个代理对象，不需要代理，直接返回当前传入的这个bean对象
    return wrapIfNecessary(bean, beanName, cacheKey);
}
```

## 总结

Spring通过三级缓存解决了循环依赖：一级缓存存放已初始完的bean，二级缓存是早期曝光对象，三级缓存是早期曝光对象工厂。

（1）当A，B两类发生循环引用时，在A完成实例化后，就使用实例化后的对象a去创建一个对象工厂，并将beanName-singletonFactory作为映射添加到三级缓存中。如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，否则是A实例化对象。

（2）当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取。

（3）首先获取到三级缓存中的工厂singletonFactory，调用其方法getObject获取对应的bean对象a，然后注入B中。紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。

（4）B创建完后，会将b再注入到A中，此时A再完成它的整个生命周期。至此，循环依赖结束



>**为什么要使用三级缓存？二级缓存能解决循环依赖吗？**
>
>首先说明Spring在结合AOP和Bean的生命周期的一个设计观念：Spring通过后置处理器AnnotationAwareAspectJAutoProxyCreator来结合AOP与Bean声明周期，它会在后置处理的`postProcessAfterInitialization`方法对初始化的Bean完成AOP代理。如果此时出现循环依赖，只能先给Bean创建代理；若没有出现则让Bean在生命周期的最后异步完成代理而不是实例化后就代理。
>
>在了解了这个观念后，就可以说明使用三级缓存的原因：三级缓存使用工厂的意义在于去延迟实例化阶段生成对象的代理。也就是说：只有在发生循环依赖时，才去提前生成代理对象。否则的话只会创建一个工厂放到三级缓存里，不会提前创建对象。
>
>如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则
>
>**为什么构造器无法处理循环？**
>
>Spring容器会将每一个正在初始化的 Bean 标识符放在一个**“当前创建Bean池”**中，Bean标识符在创建过程中将一直保持在这个池中，初始化完后会从池中移除。如果在创建Bean的过程中发现自己已在池中，就会抛出BeanCurrentlyInCreationException。
>
>当我们正在初始化单例A时，发现其依赖B，于是将A放在池中，然后创建B，又发现B依赖于A，但由于A已在池中，因此会报错。



# 参考资料

[狂神说Spring03：依赖注入（DI）](https://mp.weixin.qq.com/s?__biz=Mzg2NTAzMTExNg==&mid=2247484109&idx=1&sn=a3bc263536e84c93b9eb862cfa4da319&chksm=ce61046ef9168d78c02be9a0edcc44f1f6ea17fb16e04b733052385a700f381b0e6bb243d47b&scene=21#wechat_redirect)

[Spring ioc流程](https://my.oschina.net/huangguangsheng?tab=newest&catalogId=3608918)

[Spring循环依赖](https://www.cnblogs.com/daimzh/p/13256413.html)

