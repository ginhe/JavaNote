## 回忆MyBatis

（1）首先导入相关依赖

```xml
<!--mybatis-->
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis</artifactId>
   <version>3.5.2</version>
</dependency>
<!--mysql连接-->
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <version>8.0.17</version>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-webmvc</artifactId>
   <version>5.1.10.RELEASE</version>
</dependency>

<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-jdbc</artifactId>
   <version>5.1.10.RELEASE</version>
</dependency>
<!--aspectJ AOP 织入器-->
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjweaver</artifactId>
   <version>1.9.4</version>
</dependency>
<!--mybatis-spring整合-->
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis-spring</artifactId>
   <version>2.0.2</version>
</dependency>
```

（2）实体类：

```java
package com.jnju.entity;

public class User {
    private int id;
    private String name;
    private String pwd;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pwd='" + pwd + '\'' +
                '}';
    }
}
```

（3）mybatis的配置文件mybatisConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>    <!--指定要配置别名的包，别名为类名，不区分大小写-->
        <package name="com.jnju.entity"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/justtest?serverTimezone=UTC&amp;seUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="123"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="com.jnju.dao"/>
    </mappers>
</configuration>
```

（4）UserMapper接口以及对应的映射文件

```java
public interface UserMapper {
    public List<User> selectUser();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jnju.dao.UserMapper">

    <select id="selectUser" resultType="User">
        select * from user
   </select>

</mapper>
```

（5）测试方法：

```java
    @Test
    public void selectUser() throws IOException {
        String configName = "mybatisConfig.xml";
        InputStream inputStream = Resources.getResourceAsStream(configName);
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = factory.openSession();

        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> usersList = userMapper.selectUser();
        for(User user : usersList) {
            System.out.println(user);
        }
         sqlSession.close();
    }
```



## MyBatis-Spring

MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中，其中两者的版本对应关系如下：

| **MyBatis-Spring** | **MyBatis** | **Spring** | **Spring Batch** | **Java** |
| ------------------ | ----------- | ---------- | ---------------- | -------- |
| 2.0                | 3.5+        | 5.0+       | 4.0+             | Java 8+  |
| 1.3                | 3.4+        | 3.2.2+     | 2.1+             | Java6+   |

要和 Spring 一起使用 MyBatis，需要在 Spring 应用上下文中定义至少两样东西：一个 SqlSessionFactory 和至少一个数据映射器类。

（1）在前面的MyBatis基础用法中，是通过 SqlSessionFactoryBuilder 来创建 SqlSessionFactory 的。而 MyBatis-Spring 是使用 SqlSessionFactoryBean来创建。此外该bean还有configLocation属性和mapperLocations，它们分别用于指定MyBatis的基础配置文件xml和sql语句的映射文件。

> SqlSessionFactoryBean 会创建它自有的 MyBatis 环境配置（Environment），并按要求设置自定义环境的值。

```xml
<!--配置SqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--关联Mybatis-->
        <property name="configLocation" value="classpath:mybatisConfig.xml"/>
        <property name="mapperLocations" value="classpath:com/jnju/dao/*.xml"/>
    </bean>
```

（2）SqlSessionTemplate 是 MyBatis-Spring 的核心。作为 SqlSession 的一个实现，它可以无缝代替原始MyBatis中的 SqlSession。此处我们使用 SqlSessionFactory 作为构造方法的参数来创建 SqlSessionTemplate 对象

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
     <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

（3）mybatisConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>    <!--指定要配置别名的包，别名为类名，不区分大小写-->
        <package name="com.jnju.entity"/>
    </typeAliases>

</configuration>
```

（4）UserDaoImpl以及它对应的bean标签

```java
public class UserDaoImpl implements UserMapper {
    //Spring管理sqlSession的创建
    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    public List<User> selectUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }

}
```

```xml
<bean id="userDao" class="com.jnju.dao.impl.UserDaoImpl">
    <property name="sqlSession" ref="sqlSession"/>
</bean>
```

（5）测试方法：

```java
@Test
public void selectUserBySpring() throws IOException {
    ApplicationContext context = new ClassPathXmlApplicationContext("Spring-Mybatis.xml");
    UserMapper userMapper = (UserMapper)context.getBean("userDao");

    List<User> usersList = userMapper.selectUser();
    for(User user : usersList) {
        System.out.println(user);
    }
}
```

> 此处也可以在开启注解扫描后，通过@Component("userDao")注解和@Autowired注解来省略userDao的bean标签。



### SqlSessionDaoSupport 抽象类

我们可以通过SqlSessionDaoSupport抽象类来获得SqlSession：此时我们需要设置其属性sqlSessionFactory值

在前面例子的基础上，我们修改UserDaoImpl：

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserMapper {

    public List<User> selectUser() {
        UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
        return mapper.selectUser();
    }

}
```

修改userDao的bean标签：

```xml
    <bean id="userDao" class="com.jnju.dao.impl.UserDaoImpl">
<!--        <property name="sqlSession" ref="sqlSession"/>-->
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>
```



## 事务

事务是把一系列的操作看作一个独立的工作单元，这些操作要么全部完成，要么全部不起作用。

先来看一段没有事务的数据库操作：

（1）UserMapper接口和它的实现类

```java
public interface UserMapper {
    List<User> selectUser();
    int addUser(User user);
    int deleteUser(int id);
}
```

其中selectUser方法负责添加用户和删除新添加的用户。

```java
public class UserDaoImpl extends SqlSessionDaoSupport implements UserMapper {

    public List<User> selectUser() {
        User user = new User("dad", "123");
        UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
        mapper.addUser(user);
        mapper.deleteUser(6);
        return mapper.selectUser();
    }

    public int addUser(User user) {
        UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
        return mapper.addUser(user);
    }

    public int deleteUser(int id) {
        UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
        return mapper.deleteUser(id);
    }
}
```

（2）映射文件。此处我们故意把delete语句写错

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jnju.dao.UserMapper">

    <select id="selectUser" resultType="User">
        select * from user
   </select>

    <insert id="addUser" parameterType="User">
        insert into user (name,pwd) values (#{name},#{pwd})
    </insert>
<!--  delete的sql语句写错  -->
    <delete id="deleteUser" parameterType="int">
        deletes from user where id = #{id}
    </delete>

</mapper>
```

（3）测试方法

```java
@Test
public void testTranscation() {
    ApplicationContext context = new ClassPathXmlApplicationContext("Spring-Mybatis.xml");
    UserMapper userMapper = (UserMapper)context.getBean("userDao");

    List<User> usersList = userMapper.selectUser();
    for(User user : usersList) {
        System.out.println(user);
    }
}
```

运行会发现控制台报出sql语句的问题，并且数据库成功添加新用户。这与我们所希望的一系列操作要全部成功或失败的目标是不一致的。因此我们需要事务。

### Spring中的事务管理

Spring为我们提供了事务管理，它支持编程式事务管理和声明式的事务管理：

**编程式事务管理**

- 将事务管理代码嵌到业务方法中来控制事务的提交和回滚
- 缺点：必须在每个事务操作业务逻辑中包含额外的事务管理代码

**声明式事务管理**

- 一般情况下比编程式事务好用。

- 将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。

- 将事务管理作为横切关注点，通过aop方法模块化。Spring中通过Spring AOP框架支持声明式事务管理。

  

**示例**

（1）首先我们引入头文件：

```xml
xmlns:tx="http://www.springframework.org/schema/tx"
<!-- xsi:schemaLocation 部分->
http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx.xsd">
```

（2）引入事务管理器。Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 

```xml
<!-- 配置声明式事务-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
```

（3）配置事务通知

```xml
<!--配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
   <tx:attributes>
       <!--配置哪些方法使用什么样的事务,配置事务的传播特性-->
       <tx:method name="add" propagation="REQUIRED"/>
       <tx:method name="delete" propagation="REQUIRED"/>
       <tx:method name="update" propagation="REQUIRED"/>
       <tx:method name="search*" propagation="REQUIRED"/>
       <tx:method name="get" read-only="true"/>
       <tx:method name="*" propagation="REQUIRED"/>
   </tx:attributes>
</tx:advice>
```

其中**事务传播特性**是多个事务方法相互调用时，事务如何在这些方法间传播。spring支持7种事务传播行为：

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是Spring默认且常见的选择。
- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。
- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作

（4）导入AOP头文件并配置AOP织入事务

```xml
 <!-- xsi:schemaLocation 部分->
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd"
```

```xml
<aop:config>
    <aop:pointcut id="txPoincut" expression="execution(* com.jnju.dao.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoincut"/>
</aop:config>
```

再次进行测试就会发现数据库没有执行添加操作。