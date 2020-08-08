# JVM JDK 和JRE
Java虚拟机JVM是运行Java字节码的虚拟机
Java程序从源代码到运行分为如下步骤：

![](https://user-gold-cdn.xitu.io/2020/5/2/171d2af6c23d1561?w=1327&h=248&f=png&s=59424)


**JDK和JRE**

- JRE是Java运行时环境，它的内部包含了Java虚拟机，Java类库。它是运行java程序所需要的软件环境，但是它不能用于创建新程序。


- JDK是一个软件开发工具包，它包含JRE，编译器（javac）和工具（java程序调试和分析的工具：jconsole，jvisualvm等）。它能创建和编译程序。

> 如果你需要运行java程序，只需安装JRE。如果你需要编写java程序，则需要安装JDK。

![](https://user-gold-cdn.xitu.io/2020/7/4/1731729e49165532?w=455&h=296&f=png&s=129283)



# 面向对象

## 什么是面向对象

面向对象编程——Object Oriented Programming，简称OOP，是一种程序设计思想。OOP把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间传递。

## 抽象类和接口
**抽象类**

一个含有抽象方法的类称之为抽象类，抽象类中含有无具体实现的方法，所以不能用抽象类创建对象。
```java
public  abstract class ClassName {
    abstract void fun();
}
```
在使用抽象类时应注意：

1. 抽象类可以拥有具体成员变量和具体成员方法。
2. 抽象类可以没有抽象方法，但是如果一个类已经声明成了抽象类，即使这个类中没有抽象方法，它也不能再实例化；如果一个类中有了一个抽象方法，那么这个类必须声明为抽象类。
3. 如果一个**非抽象类**继承了抽象类，则非抽象类必须实现抽象父类的所有抽象方法。否则它必须声明为抽象类

4. abstract不能和final同时修饰同一方法。用final修饰后，修饰类代表不可以继承，修饰方法则代表不可重写。

5. abstract不能与static修饰同一个方法，static修饰的方法可以用类名调用，而对于abstract修饰的方法没有具体的方法实现，所有不能直接调用。



**接口**

接口是抽象类的延伸，java为保证数据安全是不能多重继承的，一个只能继承一个父类，但是接口不同，一个类可以同时实现多个接口。
```java
public  interface InterfaceName {
    int MAX_SERVICE_TIME = 100;
    
    public abstract void method1(int a) throws Exception;
 
    void method2(int a) throws Exception;

    default void doSomething() {
		System.out.println("do something");
	}
    
    static void staticMethod() {
        System.out.println("接口中的静态方法");
    }

}

```


在使用接口应注意：

1. 接口的方法权限默认为public abstract，但你也可以声明为static或default，这样static方法只能通过接口名（而不是实现类的类名）调用；而default方法只能通过接口实现类的对象来调用。

2. 接口中定义的所有变量默认是**public static final**的，定义的时候必须赋值，

3. 类实现接口通过implements实现，实现接口的非抽象类必须要实现该接口的**所有**方法，抽象类可以不用实现。

4. 接口没有构造函数，而抽象类可以有构造器。

5. 不能使用new操作符实例化一个接口，你只能声明一个接口变量，该变量必须引用一个实现该接口的类的对象。

6. 接口中不能含有静态代码块以及静态方法，而抽象类可以有静态代码块和静态方法。


>**接口与抽象类在不同版本中的访问权限变化**
>
>抽象类在1.8以前，其方法的默认访问权限为protected；1.8后改为default
>接口在1.8以前，方法必须是public；1.8时可以使用default；1.9时可以是private



## 三个特征

（1）**封装**：将对象的属性和行为特征包装到一个类中，把实现细节隐藏起来，通过公用的方法来展现类对外提供的功能，提高了类的内聚性，降低了对象之间的耦合性，即“高内聚、低耦合”。
>一个软件是由多个子程序组装而成，而一个程序又由多个模块(方法)构成。
>
>内聚：每个模块尽可能独立完成自己的功能，不依赖于模块外部的代码。高内聚是指尽可能类的每个成员方法只完成一件事
>
>耦合：各个子程序之间的关系紧密程度。程序的关系越复杂则耦合度越高。低耦合则是指减少类内部，一个成员方法调用另一个成员方法

（2）**继承**：通过关键字extends将已存在的类的定义作为基础建立新类，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。

继承的特点有：

* 子类拥有父类非private的属性和方法
* 子类可以拥有自己属性和方法，即子类可以对父类进行扩展
* 子类可以用自己的方式实现父类的方法（方法重写）
* 构造函数不能被继承

（3）**多态**：将父对象设置成一个或更多子对象相等的技术，赋值之后，父对象就可以根据当前赋值给它的子对象的特性以不同的方式运作。简单的说：父类变量 引用 子类对象。

以下面代码为例。当父类变量 引用 子类对象时，在调用成员函数时，应该调用向子类的成员函数。

```java
public class A {  
/*
	new A().show(new D())
	a2.show(new D())
	b.show(new D())
*/
    public String show(D obj) {  
        return ("A and D");  
    }  
    
/*	
		a1.show(b)
		a1.show(c)
*/
    public String show(A obj) {  
        return ("A and A");  
    }   
  
}  
  
public class B extends A{  
//b.show(new C())
    public String show(B obj){  
        return ("B and B");  
    }  
//a2.show(new C())
    public String show(A obj){  
        return ("B and A");  
    }   
}  
  
public class C extends B{  }  
  
public class D extends B{  }  
  
public class Test {  
    public static void main(String[] args) {  
        A a1 = new A();  
        A a2 = new B();  //父类变量a2引用子类
        B b = new B();  
        C c = new C();  					
        D d = new D();  
/*
	a1->a2
	  ->b
	  	 ->c,d
*/
        System.out.println("1--" + a1.show(b));   //1--A and A		
        System.out.println("2--" + a1.show(c));   //2--A and A 
        System.out.println("3--" + a1.show(d));   //3--A and D           
        System.out.println("4--" + a2.show(b));   //4--B and A。	   类B的 show(A obj)   
        System.out.println("5--" + a2.show(c));   //5--B and A      类B的 show(A obj)
        System.out.println("6--" + a2.show(d));   //6--A and D      类A的  show(D obj)
        System.out.println("7--" + b.show(b));    //7--B and B      类B的 show(B obj)     
        System.out.println("8--" + b.show(c));    //8--B and B    	类B的 show(B obj)
        System.out.println("9--" + b.show(d));    //9--A and D      类A的 show(D obj)
    }  
}
```

指向子类的父类引用由于向上转型，它只能访问从父类继承来的方法和属性，而对于子类中存在而父类中不存在的方法，该引用是不能使用的。

**向上转型和向下转型**

（1）父类引用可以指向子类对象：Father f1 = new Son(); 但是子类引用不能指向父类对象：Son s1 = new Father();

​	（1-1）把子类对象直接赋值给父类引用叫向上转型，向上转型不用强制转换。

​	（1-2）把指向子类对象的父类引用赋值给子类引用叫向下转型，需要强制转换：Son s2 = (Son)f1l;

### 重写与重载

实现多态的方式有：重写和重载。

**重写**

如果子类中定义某方法与其父类有**相同的方法名，参数和返回类型**，且方法的内容作出不同的处理，则该方法覆盖继承来的方法。此外重写还有一些规定：

* 子类重写父类的函数的时候，返回值类型必须是父类函数的返回值类型或该返回值类型的子类，不能返回比父类更大的数据类型；
* 子类函数的访问修饰权限不能比父类对应的访问权限还要严格；
* 子类方法抛出的异常类型必须是父类抛出的异常类型或其子类型。

此外子类不能重写父类的静态方法，这是因为静态方法是指程序一运行就已经分配好了内存地址，并且地址是固定的。如果子类定义了相同名称的静态方法，只会新增一个内存地址，不会重写。

**重载**

方法重载是指在一个类里，一个方法与另一个方法的方法名相同，返回值类型可相同也可不同，但是参数类型、个数、顺序中至少有一个不同。

> 你可以重载main()方法，但是JVM启动类时仅使用public static void main(String[] args)。



# 访问权限控制符
![](https://user-gold-cdn.xitu.io/2020/5/2/171d2cc2b32efa40?w=1132&h=301&f=png&s=57432)
（1）若一个成员需要被外部包访问，则必须使用public

（2）若一个成员需要被本包下其他类所访问，则可以使用public 或protected，或者不写任何修饰符。

（3）default 只允许在同一包中访问



# 关键字

## static
static关键字可以修饰 变量，方法，语句块和内部类，一般static 关键字与 final 一起用于定义常量。

（1）**当static修饰变量**：类的所有实例共享一份静态变量，内存中也仅存一份。

（2）**当static修饰方法**：该方法必须实现且不能是抽象方法。静态方法中不能使用 this 和 super。静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法）

>**为什么this，super不能用在static方法中?**
>
>this表示的是这个类的当前实例，super表示的是父类的当前实例，而static是属于类的，this表示的是类对象，因此不能调用

（3）**当static修饰方法语句块**：类初始化时运行一次。

（4）**当static修饰内部类时**：非静态内部类是依赖于外部类的实例，而静态内部类无需依赖，此外静态内部类不能访问外部类的非静态变量和方法。

**初始化顺序**

* 先执行父类静态内容，子类静态内容；
* 执行父类非静态代码块，父类构造方法
* 执行子类非静态代码块，子类构造方法


``` java
public class Parent  
{  
    public Parent()  
    {  
        System.out.println("Parent>>>>>>>>>>>1");  
    }  
  
    {  
        System.out.println("Parent>>>>>>>>>>>2");  
    }  
    static  
    {  
        System.out.println("Parent>>>>>>>>>>>3");  
    }  
}  
```
```java
public class Child extends Parent  
{  
    public Child()  
    {  
        System.out.println("Child>>>>>>>>>>>1");  
    }  
  
    {  
        System.out.println("Child>>>>>>>>>>>2");  
    }  
    static  
    {  
        System.out.println("Child>>>>>>>>>>>3");  
    }  
  
    public static void main(String[] args)  
    {  
        new Child();  
        /*
        Parent>>>>>>>>>>>3
        Child>>>>>>>>>>>3
        Parent>>>>>>>>>>>2
        Parent>>>>>>>>>>>1
        Child>>>>>>>>>>>2
        Child>>>>>>>>>>>1
        */
    }  
}  
 
```


## final
（1）**当final作用于基本类型变量**：变量值不能变。

（2）**当final作用于引用变量**：引用变量存放的是内存地址，地址不能变即不能指向另一个对象，但地址指向的对象可变。

```java
        final StringBuffer stringBuffer = new StringBuffer("123");
//        stringBuffer = new StringBuffer("1");   报错
        stringBuffer.append("12");
```
（3）**当final作用于类**：该类无法被继承，且类中的方法会被隐式地指定为 final 方法。



## instance of

用于判断某一对象是否为某一类型或其子类。而getClass返回的是该对象实际指向的类型。

```java
    static class Fruit {}
    static class Apple extends Fruit {}
    static class Orange extends Fruit{}

    @Test
    public void justTest() {
       Fruit apple = new Apple();
        System.out.println(apple instanceof Fruit);     //true
        System.out.println(apple instanceof Apple);     //true
        System.out.println(apple instanceof Orange);    //false

        System.out.println(apple.getClass().equals(Fruit.class));       //false
        System.out.println(apple.getClass().equals(Apple.class));       //true

    }
```





# 内部类

内部类可以分为四类：普通内部类、静态内部类、匿名内部类、局部内部类。此处仅介绍普通内部类、静态内部类、匿名内部类。
## 普通内部类

```java
public class InnerClassTest {
    public InnerClassTest() {
        // 在外部类对象内部，直接通过 new InnerClass(); 创建内部类对象
        InnerClassA innerObj = new InnerClassA();
        //外部类可以访问内部类的所有访问权限的成员
    }

    public class InnerClassA {
        //内部类可以访问外部类的所有访问权限的成员
        
        /*
        	（1）普通内部类中不能定义 static 属性
          		static int field5 = 5; 
          		原因：
          			java变量的初始化顺序是： （静态变量、静态初始化块）>（变量、初始化块）>构造器。我们要加载内部类必须等到
          			外部类实例化后，JVM才能加载内部类的字节码。但是要初始化static变量就必须加载内部类的字节码，因此是不被允许的。
         	（2）但可以定义静态常量：static final int field5 = 5;  此处的field5为编译期常量
         	这是因为：JVM会把程序中所有编译期常量都初始化并放入常量池中，无需通过加载内部类即可初始化field5变量
        */
        
        
    }

    public static void main(String[] args) {
        //声明内部类对象：
        InnerClassTest outerObj = new InnerClassTest();
        InnerClassA innerObj = outerObj.new InnerClassA();
    }
}

```
## 静态内部类
静态内部类独立于外部类对象存在，因此静态内部类中无法访问外部类的非静态成员；而外部类可以访问静态内部类对象的所有访问权限的成员

```java
public class InnerClassTest {
	public int field1 = 1;
    
	public InnerClassTest() {
        // 创建静态内部类对象
        StaticClass innerObj = new StaticClass();
        //外部类可以访问静态内部类对象的所有访问权限的成员
    }
	
    static class StaticClass {
        // 静态内部类中可以定义 static 属性
        static int field5 = 5;

        //静态内部类中无法访问外部类的非静态成员
    }

    public static void main(String[] args) {
	    // 无需依赖外部类对象，直接创建内部类对象
        InnerClassTest.StaticClass staticClassObj = new  InnerClassTest.StaticClass();
    }
}

```

## 匿名内部类
创建一个接口/抽象类对象，并在其定义中实现接口方法，此时会创建一个匿名内部类对象。

**在抽象类对象使用匿名内部类**
```java
abstract class Person {
    public abstract void eat();
}
 
public class Demo {
    public static void main(String[] args) {
        Person p = new Person() {
            public void eat() {
                System.out.println("eat");
            }
        };
        p.eat();
    }
}
```
**在接口对象使用匿名内部类**

```java
public class Demo {
    public static void main(String[] args) {
        Runnable r = new Runnable() {
            public void run() {
                 System.out.print("呱");
            }
        };
        Thread t = new Thread(r);
        t.start();
    }
}
```



# 数据类型

## 基本数据类型
**字符与字节的概念**

字节是通过网络传输信息（或在硬盘或内存中存储信息）的单位，也是计算机用于计量存储容量和传输容量的一种计量单位，1个字节等于8位二进制。而字符是人们使用的符号，比如'1'， '中'， 'a'， '$'， '￥等等。

我们将一个字符映射成⼀个⼆进制数据的过程称为**解码**，⼀个⼆进制数据映射到⼀个字符的过程叫做**解码**。我们将某个字符范围的编码规则称为**字符集**。同一个字符集可以有多种比较规则。

> 如果解码和编码使用的字符集不同，将导致两者的结果出现错误。

常见的字符集有：

（1）ASCII字符集

一共128个字符，包括空格、标点符号、数字、⼤⼩写字⺟和⼀些不可⻅字符。使⽤1个字节来进⾏编码。比如：

- 'L' -> 01001100（⼗六进制：0x4C，⼗进制：76）
- 'M' -> 01001101（⼗六进制：0x4D，⼗进制：77）

（2）utf8字符集

它收录地球上能想到的所有字符，⽽且还在不断扩充。这种字符集兼容ASCII字符集，编码⼀个字符需要使⽤1～4个字节。

**基本数据类型**

Java有8种基本数据类型，分别是：


* byte： 1字节，范围为-128-127
* char：2字节。（字符串常量占若干个字节）
* short：2字节，范围为-32768-32767
* int：  4字节，范围为正负21亿
* float：4字节。变量值后面需添加后缀F或f，否则默认为double
* long：8字节
* double：8字节
* booolean：当作为单变量时与int相等；当作为数组时占用1字节



当占位数少的类型赋值给占位数多的类型，java自动使用隐式类型转换（如int型转为long型）
当把高级别的变量的值赋给低级别变量时，必须使用强制类型转换。比如int型转为byte型。但这可能存在精度的损失。

```java
       int a = 233;
       byte ch = (byte)a;
       System.out.println(ch);		//输出-23
```

233的的二进制表示为：24个0加上11101001，而byte只有8位，因此从最高位开始舍弃，截断后剩下：11101001，由于二进制最高位1表示负数，0表示正数，其相应的负数为-23。



### 浮点数精度损失

浮点数可能会出现精度损失，如下程序所示

```java
    @Test
    public void justTest() {
        double a =1;
        double b =0.99;

        System.out.println(a-b);
        if((a-b) == 0.01){
            System.out.println("1 - 0.99 == 0.01");
        }else{
            System.out.println("1 - 0.99 != 0.01");
        }
        /*
       			 0.010000000000000009
				 1 - 0.99 != 0.01
		*/
    }
```

**精度损失原理**

首先先解释两个问题：

（1）十进制整数如何转换为二进制数

以数字11为例：

    11/2=5  余   1      
    5/2=2   余   1
    2/2=1   余   0
    1/2=0   余   1
    0结束        
11的二进制为1011。所有的整数除以2最终一定会有得到0的，因此整数永远可以用二进制精确表示 ，但小数就不一定。

（2） 十进制小数如何转化为二进制数

以小数0.9为例：

```
//乘以2直到没有小数为止
0.9*2=1.8                 取整数部分 1
0.8(1.8的小数部分)*2=1.6    取整数部分 1
0.6*2=1.2   			  取整数部分 1
0.2*2=0.4                 取整数部分 0
0.4*2=0.8                 取整数部分 0
0.8*2=1.6                 取整数部分 1
0.6*2=1.2                 取整数部分 0
 .........      0.9二进制表示为(从上往下): 1100100100100......

```

你会发现小数乘以2永远不能消除小数部分，这样的算法将无限下去。而double有效数字有限，所以必定会有损失。

**解决方案**

对于两个浮点数的比较不能使用==，我们们可以使用java.math.BigDecimal来进行double之间的比较或运算。

```java
  
		BigDecimal f1 = new BigDecimal("0.0");
        BigDecimal pointOne = new BigDecimal("0.1");
        f1 = f1.add(pointOne);

        BigDecimal f2 = new BigDecimal("0.1");
        BigDecimal eleven = new BigDecimal("11");
        f2 = f2.multiply(eleven);

        System.out.println("f1 = " + f1);
        System.out.println("f2 = " + f2);

        if (f1.compareTo(f2) == 0)
            System.out.println("f1 and f2 are equal using BigDecimal\n");
        else
            System.out.println("f1 and f2 are not equal using BigDecimal\n");
```

需要注意的是：虽然BigDecimal可以将double作为参数：BigDecimal(double)。但并不推荐这种用法，最好的做法是用字符串。如果非要用double构造，你可以使用BigDecimal.valueOf(double)

```java
    @Test
    public void justTest() {
        float a=57.3f;
        BigDecimal decimalA=new BigDecimal(a);
        System.out.println(decimalA);		//57.299999237060546875

        double b=57.3;
        BigDecimal decimalB=new BigDecimal(b);
        System.out.println(decimalB);		//57.2999999999999971578290569595992565155029296875

        double c=57.3;
        BigDecimal decimalC=new BigDecimal(Double.toString(c));
        System.out.println(decimalC);		//57.3

        double d=57.3;
        BigDecimal decimalD=BigDecimal.valueOf(d);
        System.out.println(decimalD);		//57.3
    }
```

再来看一下BigDecimal的两个比较方法：

```java
//euquals方法先判断要比较的数据类型，然后比较两数的精度和值        
System.out.println(new BigDecimal("1.2").equals(new BigDecimal("1.20")));  //输出false
//compareTo方法会把精确度低的转为高精确度，然后再进行比较
System.out.println(new BigDecimal("1.2").compareTo(new BigDecimal("1.20")) == 0); //输出true
```



## String类
在JDK8中，String类使用char[]数组保存值，该数组使用final修饰，即一旦引用便不可修改。修改String值的方法实际是创建了一个新String对象。

**为什么是final修饰？**

（1）确保String不会在其子类改变语义（因为没有子类）

（2）线程安全性

（3）在下面我们会提到：对于形如**String str = “a”**的赋值语句，它实际是从字符串常量池中取值

```java
String str = “a”;
String str1 = “a“;
```

这两个变量都是指向堆内存的一个String对象，如果这个String对象被允许改变，则可能会出现逻辑错误，比如修改了变量a，却也把变量str1也修改了。


### new String(“a”)  和 “a”
JVM对于String的存储是放在String常量池，这个常量池存放着对String对象的引用。

**new String("a")**

JVM会在字符串常量池中找"a"字符串，若没有则创建字符串常量，然后放到常量池中。接着在堆内存中创建一个存储“a”的新String对象，并返回其对象引用地址。因此该过程创建了两个对象。

**String str = “a”**

先检查字符串常量池中有没有"a"，若没有则创建一个，然后str指向池中的对象；若有则直接指向。

```java
String a = new String("a");
String b = "a";
String c = "a";
System.out.println("a == b " + (a == b));    //false
System.out.println("a.equals(b) " + a.equals(b));   // true

System.out.println("b == c " + (b == c));      // true
System.out.println("b.equals(c) " + b.equals(c));      // true
```

### String str= "a" + "b" + "c" 一共创建了几个对象

答案是一个。java可以在编译时对字符串常量直接相加的表达式进行优化，而不必等到运行期再去进行加法运算处理。

### String、StringBuilder与StringBuffer区别

（1）String使用私有的常量char数组，因此不可变；其他二者均使用普通的char数组。

（2）线程安全性方面：

String由于不可变性，天生线程安全；

StringBuffer则由于使用了synchronized关键字同样线程安全；

StringBuilder则不保证线程安全。但由于锁的获取和释放会带来开销，所以StringBuilder效率更高。

（3）一些方法上：

equals方法：StringBuilder和StringBuffer均未重写该方法，默认通过==比较；而String是通过重写了该方法，如下所示

```java
private final char value[];

public boolean equals(Object anObject) {
        if (this == anObject) {								//(1)当前对象与比较对象是否为同一对象
            return true;
        }
    /*
    	若传入对象是String类型：
    	（2）判断长度是否相等
    	（3）按照数组value的每一位进行比较
    	否则返回false
    */
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                            return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

toString方法：String直接返回自身；StringBuilder返回一个新的String方法；StringBuffer添加了synchronized方法

```java
//StringBuffer:
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}

//StringBuilder：
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```



## 包装类
Java为每种基本数据类型分别设计了对应的类，即包装类。包装类对象一经创建，其内容（所封装的基本类型数据值）不可改变。

### 自动拆箱与自动装箱

自动装箱是Java自动将基本类型转换成对应的对象，比如调用Integer的`valueOf`方法将int的变量转换成Integer对象，反之调用Integer的intValue方法将Integer对象转换成int类型值，这个过程叫做自动拆箱。

- 进行 = 赋值操作（装箱或拆箱）
- 进行+，-，*，/混合运算 （拆箱）
- 进行>,<,==比较运算（拆箱）
- 调用equals进行比较（装箱）
- ArrayList,HashMap等集合类 添加基础类型数据时（装箱）



### 包装类的缓存机制

创建包装类有两种方式：

* 构造器方法（new）
* 自动装箱（Integer.valueOf）

两种创建方式的区别在于：

* 对于构造器方法，每次构建返回的将都会是一个新对象；
* 自动装箱会先判断，再决定返回的是一个新对象还是常量池中已存在的对象。

**一个示例**

```java
   	 int a = 100;
     Integer b = 100;
     System.out.println(a == b);		//b自动拆箱。输出true
     
     Integer c = 100;
     Integer d = 100;
     System.out.println(c == d);		//c，d通过valueOf方法装箱，生成两个Integer对象，输出true
       
     c = 200;
     d = 200;
     System.out.println(c == d);		//输出false
```

为什么第三个的输出是false，这是因为在自动装箱的过程中，它会先判断i值是否在-128和127之间，如果在-128和127之间则直接从IntegerCache.cache缓存中获取指定数字的包装类；不存在则new出一个新的包装类。因此上例中第二个输出是true，第三个输出是false。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

值得一提的是：只有double，float和boolean的自动装箱代码没有使用缓存，这三者每次都是new 新的对象，其它的6种基本类型都使用了缓存策略。



## 值传递和引用传递

**值传递**：方法接收的是调用者提供的值。Java总是采用值传递，即方法得到的是所有参数值的拷贝。

**引用传递**：方法接收的是调用者的引用。

在Java方法中，若方法参数是基本数据类型，则传递的值是基本类型的字面量值的拷贝，方法对参数的修改是无效的；但若是对象引用，则传递的是引用的对象在堆中地址值的拷贝，方法对引用对象的改变会被反应到对应的对象中

### 深拷贝和浅拷贝

**深拷贝**：对基本数据类型进行值传递，对引用数据类型进行引用传递。

**浅拷贝**：对基本数据类型进行值传递，对引用数据类型则创建一个新的对象，并复制其内容

![](https://user-gold-cdn.xitu.io/2020/5/3/171d86c504420d76?w=448&h=202&f=png&s=85709)

**如何实现拷贝？**

首先来看一下浅拷贝：

被复制的类需要实现Cloneable接口，然后覆盖clone()方法。

```java
public class JustTest implements Cloneable{
    private int num;

    public void setNum(int num) {this.num = num;}

    @Override
    public Object clone() throws CloneNotSupportedException {
        JustTest tmp = null;
        try {
            tmp = (JustTest)super.clone();
        }catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return tmp;
    }
	//省略toString方法.

    @Test
    public void name() throws CloneNotSupportedException {
        JustTest jt = new JustTest();
        jt.setNum(123);
        JustTest ano = (JustTest)jt.clone();

        System.out.println(jt);
        System.out.println(ano);
        jt.setNum(13);
        System.out.println("分割线---------------");
        System.out.println(jt);
        System.out.println(ano);
        System.out.println("jt == ano ? " + (jt == ano));
    }
    /*
    输出：
        JustTest{num=123}
        JustTest{num=123}
        分割线---------------
        JustTest{num=13}
        JustTest{num=123}
        jt == ano ? false
     */
}
```

此时如果我们将属性更改为引用类型

```java
public class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }
	//省略setter, toString方法
}
```

测试一下：

```java
    @Test
    public void name() throws CloneNotSupportedException {
        Person person = new Person("jnju");
        JustTest jt = new JustTest();
        jt.setPerson(person);
        JustTest ano = (JustTest)jt.clone();

        System.out.println(jt);
        System.out.println(ano);
        person.setName("666");
        System.out.println("分割线---------------");
        System.out.println(jt);
        System.out.println(ano);
		/*
			JustTest{person=Person{name='jnju'}}
            JustTest{person=Person{name='jnju'}}
            分割线---------------
            JustTest{person=Person{name='666'}}
            JustTest{person=Person{name='666'}}
		*/
    }
```

这也就认证了浅拷贝对于引用变量person只是复制其地址，并没有新开辟另一个空间。

**如何实现深拷贝**

我们可以对引用类Person实现Cloneable接口，然后覆盖clone()方法。

此外还可以将类型序列化，此时写到流中的对象是原始对象的一个拷贝，然后再利用反序列化获取该对象。

# Object类

Java中每个类都由Object类扩展而来，所有数组类型（对象数组和基本类型的数组）扩展了Object类。
## equals方法

该方法用于判断两个对象的内容是否相同，该方法遵循如下原则：

* 自反性：对于任何非空引用值X，X.equals(X)都应返回true
* 对称性：对于任何非空引用值X和Y，当且仅当Y.equals(X)返回true时，X.equals(Y)也应该返回true
* 传递性：对于任何非空引用值X，Y，Z，如果X.equals(Y)返回true，并且Y.equals(Z)返回true，那么X.equals(Z)应返回true
* 一致性：对于任何非空引用值X和Y，多次调用X.equals(Y)始终返回true或始终返回false

在我们自定义的类中，若不重写equals 方法，则它会默认调用Object 类的 equals方法，也就是用==运算符比较两个对象。
### == 和 equals()的区别

对基本类型使用 == ：比较两个的值是否相等；

对引用类型使用 == ：比较两个的引用是否相同；
而equals()方法比较的则是值是否相同：（String重写了equals方法，该方法比较值）

```java
String x = "string";
String y = "string";
String z = new String("string");
System.out.println(x == y); // true
System.out.println(x == z); // false    String()方法重新开辟内存空间
System.out.println(x.equals(y)); // true
```



## 空指针异常

Object的equals方法容易抛空指针异常，因此我们应该使用常量或确定有值的对象来调用 equals方法。

```java
// 不能使用一个值为null的引用类型变量来调用非静态方法，否则会抛出异常
String str = null;
if (str.equals("jnju")) {
  ...
} else {
  ..
}
```

上面的程序会抛出空指针异常。因此我们可以修改为：

```java
"jnju".equals(str);
```





## hashcode方法

如果重写了equals方法，必须重新定义hashCode方法。该方法返回对象的hash值。

**为什么重写了equals方法，需要重新定义hashCode方法**

在HashMap的添加元素操作中，需要通过hashCode方法来定位在元素要放入的位置，如果不重新定义hashCode方法，则会出现本应被认为相同的两个对象由于hash值不同而使得hashmap存了两个相同对象的情况。

**将对象作为HashMap的key的注意事项**

一定要注意重写equals和hashCode两个方法。hashmap在判断其是否存在key是通过传入key的hash值是否相等（通过==比较）以及key的值或者引用是否相等来判断的。

我们来看一下Hashmap对于key是否存在的判断方法：

```java
public boolean containsKey(Object key) {
     return getNode(hash(key), key) != null;
}
 final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && 		//此处是判断的条件
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

```

我们可以来测试一下：

（1）作为key的学生类

```java
public class Student {
    private Integer id;
    
    private String name;

   //省略构造方法，getter，setter
   
    @Override
    public boolean equals(Object o){
        return this.id.equals(((Student) o).getId());
    }

    @Override
    public int hashCode(){
        return this.id;
    }
    
}
```

（2）测试：

```java
   @Test
   public void test1(){
       Student s1 = new Student(2, "呱");
       Student s2 = new Student(2, "咕");
       HashMap<Student, Integer> map = new HashMap<>();

       map.put(s1, 1);
       if(!map.containsKey(s2)) {       //s2的id和s1的id相同，不应被进map里
           map.put(s2, 2);
       }

       System.out.println(map);		//{Student{id:2;name:呱}=1}
   }
```

如果你注释掉Student类中equals和hashCode两个方法中的任意一，那么map也会把s2放进 。



哈希码的通用约定如下：

* 在一个对象没有被改变的前提下，无论这个对象被调用多少次，hashCode 方法都会返回相同的整数值。此外对象的哈希码没必要在不同的程序中保持相同的值。
* 如果 2 个对象使用 equals 方法相同的话，则 hashcode 一定也是相同的
* 两个对象有相同的 hashcode 值，它们也不一定是相等的



# 异常

若某个方法无法按照正常途径完成任务，需要另一种途径退出方法。则可以抛出一个封装错误信息的对象。此时方法会立即退出且不返回任何值。

异常处理机制将代码执行交给异常处理器。Throwable是所有异常或错误的超类，其子类有Error和Exception。

**Exception和Error的区别**

**（1）Error**

程序无法处理的错误，表示Java运行时系统的内部错误和资源耗尽错误，例如Java 虚拟机运行错误，OutOfMemoryError等。这些异常发生时，JVM会选择线程终止。

**（2）Exception**

这种异常分为 运行时异常RuntimeException和非运行时异常IOException(编译异常)。

- 运行时异常：RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的。
- 非运行时异常：必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常



**Throwable 类常用方法**

public string getMessage()：返回异常发生时的简要描述

public string toString()：返回异常发生时的详细信息



## 异常处理

**try块**：捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。

**catch块**：处理 try 捕获到的异常。

**finally块**：无论是否捕获或处理异常，finally 块里的语句都会被执行。如果在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行。

>- 如果finally语句第一行出现了异常，则finally块不会执行；若在其他行则还会得到执行。
>
>
>- 此外如果try 语句和 finally 语句中都有 return 语句，则finally语句的返回值会覆盖掉try语句块的返回值。






# 泛型
**什么是泛型**

我们可能遇到多个模块的功能非常相似，只是处理的数据类型可能不同（可能是int，String或其他自定义类型），因此我们可以通过泛型来表示为一个通用的数据类，这样就不必为每个数据类型再重复的写功能。

以ArrayList为例，在声明其对象时，只需声明放入其中的元素是什么类型：

```java
ArrayList<String> list = new ArrayList<>();
arrayList.add(123);
arrayList.add("dasd");//报错
```
泛型是一种定义的模板，这样我们就可以定义放入各种不同类型的ArrayList对象。

**（1）泛型类**

一个泛型类是具有一个或多个类型变量的类。
```java 
public class Test {
    static class Pair<T> {				//若需要类中有多个类型：public class Pair<T, U>
        private T first;
        private T second;
        
        public Pair(T first, T second) {
            this.first = first;
            this.second = second;
        }
    //省略get/set方法
    public static <T> T getMiddle(T... a) { return a[a.length / 2];}
    public static void main(String[] args) {
        Pair<String> peron = new Pair<>("123", "dqdq");
        System.out.println(peron.getFirst());
    }
}
```
**（2）泛型接口**


```java
public interface Generator<T> {
    public T next();
}
```


**（3）泛型方法**

```java
public class Test {
    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }
    public static void main(String[] args) {
        System.out.println(Test.getMiddle("1", "2", "3"));
//        大多情况下，可以省略尖括号里的说明
        System.out.println(Test.<String>getMiddle("r", , "3", "fcqw",));
    }
}
```

## 类型变量的限定
有时候我们需要约束 类的对象或方法里的参数，如下所示，我们需要确保T继承了Comparable接口，这样才能重写排序规则。


```java
    public static <T extends Comparable>  T min(T[] a) {
         if(a == null || a.length == 0)     return null;
         T smallest = a[0];
         for(int i = 1; i < a.length; i++) {
             if(smallest.compareTo(a[i]) > 0)
                 smallest = a[i];
         }
        return smallest;
    }
```

一个类型变量可以有多个限定。限定类型用&分割，而逗号用来分割类型变量。不过需要注意的是：**限定中可以有多个接口超类型，但最多有一个类，且该类必须是限定列表中的第一个。**


```java
<T extends ArrayList & Runnable & Serializable> 
```

## 类型擦除和翻译泛型
### 类型擦除
无论什么时候定义一个泛型类型，都会自动提供一个相应的原始类型。若指定了限定类，则对应的原始类型就是第一个限定类型，否则是Object。

假设声明了一个泛型类
```java
public class Pair<T extends Comparable & Serializable> implements Serializable{
    private T first;
    private T second;
        
     public Pair(T first, T second) {
        // 省略...
    }
}
```

那么其原始类型为：


```java
public class Pair implements Serializable{
    private Comparable first;
    private Comparable second;
        
     public Pair(Comparable first, Comparable second) {
        // 省略...
    }
}
```

若调换一下限定类型的位置，则其原始类型用Serializable替换T，而编译器会在必要时向Comparable插入强制类型转换。因此为了提高效率，应将标签接口（没有方法的接口）放在边界列表的末尾。



### 翻译泛型

假设有一泛型类：

```java
public  class Pair<T> {
         private T first;
         private T second;
         
         public Pair(T first, T second) {
             this.first = first;
             this.second = second;
         }
         
          public T getFirst() {
             return first;
         }
 }
```

当程序调用泛型方法时，若擦除了返回类型，则编译器会插入强制类型转换。如下所示，擦除getFirst方法的返回类型后，将返回Object类型，编译器自动插入Employee类型。

```java
Pair<Employee> buddies = ...;
Employee buddy = buddies.getFirst();
```
即编译器把getFirst方法的调用翻译成两条虚拟机指令：

* 调用原始方法Pair.getFirst。
* 对返回的Object类型强制转换为Employee类型。

如果泛型类Pair的成员变量也是public，则在使用到其成员变量时也是要强制类型转换的：


```java
Employee buddy = buddies.first;
```

**翻译泛型方法**

类型擦除会带来一定的麻烦，比如继承泛型类型的多态麻烦（子类没有覆盖父类的方法）。

在下面的程序中，程序员希望在SonPair类覆盖父类Pair的setFirst(T first)


```java
class SonPair extends Pair<String>{  
     public void setFirst(String fir){....}  
}
```

在前面的类型擦除的基础上，你可能会认为：Pair在编译阶段已被类型擦除为Object了。它的相关方法变成了setFirst(Object first)，因此setFirst(String first)无法覆盖父类的方法。

不过编译器会解决这样的问题：自动在SonPair中生成一个桥方法。


```Java
public void setFirst(Object first) {
    setFirst((String) first)
} 
```

我们都知道：**编译器不允许我们编写方法参数一样的多个方法**，但是JVM会用参数类型和返回类型来确定一个方法，一旦编译器通某种方式自己编译出方法签名一样的两个方法，JVM还是能够分清楚这些方法的，前提是需要返回类型不一样。

总结一下：

* 虚拟机中没有泛型，只有普通类和方法。
* 在编译阶段，所有泛型类的类型参数都会被Object或者它们的限定边界来替换(类型擦除)。
* 桥方法的合成是为了保持多态。




## 泛型的约束与局限性

（1）**不能用基本类型实例化参数**

没有类似于Pair< double>的定义，只有Pair< Double>。这是因为类型擦除后的Pair类可能含有Object类的成员变量，而Object不能存储类似double的基本数据类型。

（2）**运行时类型查询只适用于原始类型**

在使用instanceof来查询一个对象是否属于某个泛型类型时会报错，但在使用getClass方法时总会返回原始类型：

```java
Pair<String> sp = ...;
Pair<Employee> ep = ...
if(sp.getClass() == ep.getClass()) // 相等
```

**（3）不能创建参数化类型的数组**

不允许创建一个参数化类型的数组。

```
Pair<String>[] array = new Pair<String>[10];
```

这是因为：对于上述语句，类型擦除后array的类型为Pair[]，如果我们将他转换为Object[]：


```
Object[] objarray = array;
```
数组会记住它的元素类型，当试图传入其他类型的元素时，本应该会报错。

```
object[0] = "halo";// Error-component type is Pair
```

但对于泛型类型，擦除会使这种机制无效。因此不允许创建参数化类型数组。

不过，仅仅不允许new Pair<String>[10]，而声明类型为Pair<Strimg>[]的变量是被允许的。

**（4）不能实例化类型变量**

不能使用new T(...)或者new T[...]或T.class这样的表达式中的类型变量。 因此下例中的构造器是非法的。

```
 public Generic(){
     first = new T();
     second = new T();
}
```
**（5）泛型类型的继承规则**

无论E与T有什么联系，通常，Pair<E>与Pair<T>没有什么联系。



# 集合

## 概述

Java集合大致分为Conllection和Map接口：

<img src="https://user-gold-cdn.xitu.io/2020/7/18/1735fa8a4fbc02ef?w=942&amp;h=461&amp;f=png&amp;s=166242"  />

### Iterator 迭代器

Iterator 迭代器可以用于对集合进行遍历，如果在遍历过程中，集合元素被修改，会抛出 `ConcurrentModificationException` 异常。

```java
public interface Iterator<E> {
    //集合中是否还有元素
    boolean hasNext();
    //获得集合中的下一个元素
    E next();
    ......
}
```

**使用示例**

```java
Map<Integer, String> map = new HashMap();
map.put(1, "Java");
map.put(2, "C++");
map.put(3, "PHP");

Iterator<Map.Entry<Integer, String>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
  Map.Entry<Integer, String> entry = iterator.next();
  System.out.println(entry.getKey() + entry.getValue());
}
```



### List接口

List存储的元素是有序，可重复的。它又分为ArrayList，Vector和LinkedList。

其中ArrayList，Vector的底层都是Object[]数组，两者也存在不同之处：

- ArrayList 是 List 的主要实现类，适用于频繁的查找工作，线程不安全
- Vector 是 List 的古老实现类，它的方法使用了synchronized修饰符，因此线程安全但性能比前者低

如果是在多线程环境下，更推荐如下：

```java
ArrayList list = Collections.synchronizedList( new ArrayList() );
```

如果我们需要对一个集合使用自定义排序时，我们就要重写`compareTo()`方法或`compare()`方法。

```java
ArrayList<Integer> arrayList = new ArrayList<Integer>();
        arrayList.add(-1);
        arrayList.add(3);
        arrayList.add(3);
        arrayList.add(-5);
        arrayList.add(7);
        arrayList.add(4);
        arrayList.add(-9);
        arrayList.add(-7);
        System.out.println("原始数组:");
        System.out.println(arrayList);

        // void sort(List list),按自然排序的升序排序
        Collections.sort(arrayList);
        System.out.println("Collections.sort(arrayList):");
        System.out.println(arrayList);
        // 定制排序的用法
        Collections.sort(arrayList, new Comparator<Integer>() {

            /*
            	对于compareTo方法：若参数o1大于调用方法的对象o2，则返回小于0的数
            	对于compare方法：如果返回值小于0，则需要交换o1和o2的顺序。
            */
            
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2.compareTo(o1);		
            }
        });
        System.out.println("定制排序后：");
        System.out.println(arrayList);


/*

原始数组:
[-1, 3, 3, -5, 7, 4, -9, -7]
Collections.sort(arrayList):
[-9, -7, -5, -1, 3, 3, 4, 7]
定制排序后：
[7, 4, 3, 3, -1, -5, -7, -9]

*/
```

如果要比较的对象是自定义类，则需要实现Comparable接口

```java
public  class Person implements Comparable<Person> {
    @Override
    public int compareTo(Person o) {
        ...
    }
}
```


LinkedList底层使用了双向链表。

#### 常用方法

**ArrayList**

```java
        ArrayList<Integer> alist = new ArrayList<>();
        alist.add(32);
		alist.add(0,23);		//(1)在下标0处插入数字

		//(2)遍历方式
        for(Iterator it  = alist.iterator(); it.hasNext(); ) {
            System.out.println(it.next());
        }

        for(Integer tmp:alist){
            System.out.println(tmp);
        }
		for(int i = 0;i < alist.size(); i ++){
    		System.out.println(list.get(i));
		}	
		alist.clear();		//(3)删除所有元素

		//(4) alist.contains(23);
		//(5)alist.get(0);
		//(6)alist.indexOf(23) 				返回指定元素第一次出现的列表中的索引，若不存在则返回-1
		//(7)alist.lastIndexOf(23)			返回指定元素最后一次出现的列表中的索引
		//(8)int[] arr = alist.toArray() 	返回包含所有元素的数组
		//(9)boolean addAll(Collection c)	将指定集合中的所有元素按指定集合的迭代器返回的顺序附加到此列表的末尾。
		//(10)alist.remove(int index)		删除指定下标的元素
```

**LinkedList**

```java
        LinkedList<Integer> alist = new LinkedList<>();
        alist.add(32);
        for(Iterator it  = alist.iterator(); it.hasNext(); ) {
            System.out.println(it.next());
        }

        for(Integer tmp:alist){
            System.out.println(tmp);
        }
//        boolean add(E e)					在链表后添加一个元素，如果成功，返回true，否则返回false；
//        void addFirst(E e)				在链表头部插入一个元素；
//        addLast(E e)						在链表尾部添加一个元素；
//        void add(int index, E element)	在指定位置插入一个元素。

//        E remove()				移除链表中第一个元素；
//        boolean remove(Object o)	移除链表中指定的元素；
//        E remove(int index)		移除链表中指定位置的元素；
//        E removeFirst()			移除链表中第一个元素，与remove类似；
//        E removeLast()			移除链表中最后一个元素；

//        E get(int index)            按照下边获取元素；
//        E getFirst()                获取第一个元素；
//        E getLast()                 获取第二个元素；
//        E poll()	               	 删除并返回第一个元素。
//        boolean contains(Object o)	判断是否含有某一元素。
```



### Set接口

Set存储的元素是无序的、不可重复的，它有3个主要实现类：HashSet、LinkedHashSet 和 TreeSet

- HashSet 是 Set 接口的主要实现类 ，它的底层是 HashMap，线程不安全的，可以存储 null 值；
- LinkedHashSet 是 HashSet 的子类，能够按照添加的顺序遍历；
- TreeSet 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。




### Map接口

Map使用键值对key-value存储。其中Key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。

此处我们主要介绍HashMap 和 Hashtable

（1）**线程安全性**： HashMap 是非线程安全的，HashTable 是线程安全的,因为 HashTable 内部的方法基本都经过`synchronized` 修饰。不过如果要保证线程安全，还是使用 ConcurrentHashMap ；

（2）**效率**： 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。此外HashTable 基本被淘汰。

（3）**对Null key和 Null value 的支持：** HashMap 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；HashTable 不允许有 null 键和 null 值，否则会抛出 NullPointerException。

（4）**容量大小：** ① 创建时如果不指定容量初始值，Hashtable 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。HashMap 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍；②创建时如果给定了容量初始值，那么 Hashtable 会直接使用给定的大小，而 HashMap 会将其扩充为 2 的幂次方大小；

**遍历HashMap**

```java
//(1)
for (Integer key : map.keySet()) {
	System.out.println("Key = " + key);
}

for (Integer value : map.values()) {
	System.out.println("Value = " + value);
}

//(2)
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
	System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}
```





## Arrays.asList()

（1）该方法是泛型方法，传入的对象必须是对象数组。

```java
int[] myArray = { 1, 2, 3 };
List myList = Arrays.asList(myArray);
System.out.println(myList.size());	//1
System.out.println(myList.get(0));	//数组myArray地址值
System.out.println(myList.get(1));	//报错：ArrayIndexOutOfBoundsException
```

当传入一个原生数据类型数组时，`Arrays.asList()` 返回的是数组myArray本身。因此我们需要传入包装类。

（2）在使用该方法把数组转为集合后，不能再使用修改集合的方法，比如add/remove/clear，否则会抛出`UnsupportedOperationException`异常。

```java
        String[] strArray = new String[] {"sdad", "adssd"};
        List list = Arrays.asList(strArray);
//        list.add("ewq");
```

这是因为该方法返回的是Arrays内部类，它并没有实现集合的修改方法。Arrays.asList()体现的是适配器模式，只是转换接口，后台的数据仍是数组。

那么我们该如何正确的将数组转为ArrayList？

- List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
- 

## 常用方法

```java
Arrays.fill(new int[2], 0);
```



# 通配符

## 数组的协变
在了解通配符之前，先讲解一下Java数组的协变。


```Java
public class Test {
    static class Fruit {}
    static class Apple extends Fruit {}
    static class Orange extends Fruit{}
    static class BadApple extends Apple{}
    
    public static void main(String[] args) {
      Fruit[] fruits = new Apple[10];
      fruits[0] = new Apple();
      fruits[1] = new BadApple();
        
      try {
          fruits[0] = new Fruit();
      }catch(Exception e) { System.out.println(e); }

      try {
            fruits[0] = new Orange(); 
        } catch(Exception e) { System.out.println(e); }
    /*
    输出：
        java.lang.ArrayStoreException: Ano.Test$Fruit
        java.lang.ArrayStoreException: Ano.Test$Orange
     */
    }
}

```

Apple类是Fruit的子类，一个 Apple 对象也是一种 Fruit 对象，所以一个 Apple 数组也是一种 Fruit 的数组。这称作数组的协变。

尽管 Apple[] 可以 “向上转型” 为 Fruit[]，但数组元素的实际类型还是 Apple，我们只能向数组中放入 Apple或者 Apple 的子类。虽然上面的代码是可以通过编译器的，但在运行期间，JVM能知道数组的实际类型是 Apple[]，因此当放入Fruit类对象和Orange类对象时，就会抛出异常。

泛型设计的目的之一是为了使这种运行期间的错误在编译器就可以发现，但如果我们使用泛型来代替数组，如下所示，就会报错。尽管 Apple 是 Fruit 的子类型，但是 ArrayList<Apple> 不是 ArrayList<Fruit> 的子类型，泛型不支持协变。

```java
// Compile Error: incompatible types:
ArrayList<Fruit> flist = new ArrayList<Apple>();
```

## 使用通配符
如果我们希望建立类似ArrayList< Apple> -> ArrayList < Fruit> 的向上转型的关系，我们可以使用通配符。

### 上限定通配符
来看一下 <? extends Fruit> 形式的通配符：


```java
public class GenericsAndCovariance {
    public static void main(String[] args) {
        List<? extends Fruit> flist = new ArrayList<Apple>();
        // 如下添加方式都会报错：
        // flist.add(new Apple());
        // flist.add(new Fruit());
        // flist.add(new Object());
        flist.add(null); // Legal but uninteresting
        Fruit f = flist.get(0);
    }
}
```
此处的通配符代表一种特定类型，但flist没有指定，只知道是Fruit的子类型，即Fruit是它的上边界。我们不知道flist到底可以装入什么类型，因此也就不能安全的添加一个对象。

因此当我们做了泛型的向上转型 (List<? extends Fruit> flist = new ArrayList< Apple>())，我们也就失去了向这个 List 添加任何对象的能力，即使是 Object 也不行。

>不过如果我们调用某个返回Fruit的方法，如flist.get(0)，那它是安全的，因为无论flist实际装的是什么类型，它肯定可以转型为Fruit。


看上去我们好像不能调用任何接受参数的方法，但实际不是：


```java
    public static void main(String[] args) throws Exception{
        List<? extends Fruit> flist = Arrays.asList(new Apple());
        Apple a = (Apple)flist.get(0);  
        flist.contains(new Apple());
        flist.indexOf(new Apple());
    }
}
```

由上可知，我们可以通过Arrays类返回一个装入Fruit子类Apple的数组，而flist可以调用 contains 和 indexOf 方法，它们都接受了一个 Apple 对象做参数。

Arrays.asList源码如下。它接收一个泛型类型的参数。
```java
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```

而contains方法和 indexOf方法 接受的是一个 Object 类型的参数


```java
public boolean contains(Object o)
public int indexOf(Object o)
public boolean add(E e)
```


这就是为什么我们通过add方法添加元素的原因：当我们指定flist中可以装入的泛型参数为 <? extends Fruit> ，add()方法的参数变成 <? extends Fruit>，编译器无法判断这个参数接收的是Fruit的哪种类型，因此不会接收任何类型。

而contains方法和 indexOf方法参数的类型没有涉及到通配符，所以编译器允许调用这两个方法。


因此如果我们编写的某些方法不允许参数类型是通配符时，这些方法的参数应该用类型参数，如add(E e)


### 下边界限定通配符

同样的例子，下面的方法参数是 < List<? super Apple>，它表示apples装入的是Apple类的父类对象。我们无法知道实际类型是什么。

我们被允许向apples装入Apple或Apple的子类，因为Apple或其子类肯定可以向上转型为 Apple；但我们无法装入Fruit，因为这可能不安全。
```java
    static void writeTo(List<? super Apple> apples) {
        apples.add(new Apple());
        apples.add(new BadApple());
        // apples.add(new Fruit()); // Error
    }
```


在了解子类型边界和超类型边界后，我们就可以知道如何向泛型类型写入和读取：


```Java
  public static <T> void copy(List<? super T> dest, List<? extends T> src)  {
      for (int i = 0; i < src.size(); i++) 
        dest.set(i, src.get(i)); 
        /*
            我们要从src对象读数据，因此使用上边界限定通配符
            要向dest对象写入数据，因此使用下边界限定通配符
        */
  } 
```


### 无边界通配符
无边界通配符的使用形式为：List<?>，即没有任何限定，无法知道具体是哪种类型，也就无法添加任何对象。


```Java
    public static void main(String[] args) throws Exception{
        List<?> flist = new ArrayList<>();
        //flist.add(new Apple());   //  报错
        //flist.add(new Fruit());   //  报错
        //flist.add(new Object());    //报错
    }
```

如果是List flist，即没有传入泛型参数，表示 flist 持有元素的类型是 Object，因此可以添加任何类型的对象。


```Java
    public static void main(String[] args) throws Exception{
        List flist = new ArrayList<>();
        flist.add(new Apple());   
        flist.add(new Fruit());   
        flist.add(new Object());    
    }
```

***
# 参考资料

[JavaGuide](https://github.com/Snailclimb/JavaGuide)

[Java 泛型总结（三）：通配符的使用](https://segmentfault.com/a/1190000005337789)

[Java基础总结](https://blog.nowcoder.net/n/8f0280724e074093a7e7b5951098c2bc)

[double精度丢失问题](https://blog.csdn.net/tomcat_2014/article/details/51453988)

《Java核心技术卷一》