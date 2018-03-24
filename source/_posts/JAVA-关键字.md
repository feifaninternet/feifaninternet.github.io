------------------
title: JAVA 关键字
------------------
date: 2018-03-19 16:19:17
tags: JAVA 关键字
categories: JAVA

### 总览
#### 1.访问控制 ####
   ```
    private    protected    public
   ```
#### 2.类,方法和变量修饰符 ####
   ```
    abstract    class    extends    final    implements    interface    native    
    new    static    strictfp    synchronized    transient    volatile
   ```
#### 3.程序控制 ####
   ```
    break    continue    return    do    while    if    
    else    for    instanceof    switch    case    default
   ```
#### 4.异常处理 ####
   ```
    try    catch    throw    throws
   ```
#### 5.基本类型 ####
   ```
   boolean    byte    char    double    float    int    long    short    null    true    false
   ```
#### 6.变量引用 ####
   ```
   super    this    void
   ```

### 详细解释

### 1.访问控制

#### 1.1 private 私有的
&nbsp;&nbsp;private 修饰符可用于类，方法或字段，这些被 private 修饰的类，方法或者字段只能在该类中引用，在类的外部或者子类是不可见的，所有类成员的默认访问范围都是 package 访问。
#### 1.2 protected 受保护的
&nbsp;&nbsp;&nbsp;&nbsp;protected 关键字修饰的对象在该对象所在的包以外的任何地方是不可以通过该类对象的引用来调用它的 protected 方法和属性的。但是在某类中定义一个 protected 方法，在该类所在的包中是可以访问该类的子类中的 protected 方法的，即使该子类的方法是 protected 修饰的。(对于构造函数，不存在继承问题，此时为super()调用问题)
#### 1.3 public 公共的
&nbsp;&nbsp;public 关键字，作用如其名。

### 2.类、方法和变量修饰符

#### 2.1 abstract 声明抽象
&nbsp;&nbsp;abstract 关键字可以修饰类或方法。   
&nbsp;&nbsp;abstract 类可以扩展（增加子类）,但不能直接实例化。   
&nbsp;&nbsp;abstract 方法不在声明它的类中实现，但必须在某个子类中重写。   
&nbsp;&nbsp;采用 abstract 方法的类本来就是抽象类，并且必须声明为 abstract。   
&nbsp;&nbsp;仅当 abstract 类的子类实现其超类的所有 abstract 方法时，才能实例化 abstract 类的子类。这种类称为具体类，以区别于 abstract 类。   
&nbsp;&nbsp;abstract 关键字不能应用于 static、private 或 final 方法，因为这些方法不能被重写，因此，不能在子类中实现,其中 final 类还不能有子类。
#### 2.2 class 类
&nbsp;&nbsp;类是相关变量和/或方法的集合。类是面向对象的程序设计方法的基本构造单位。每个对象都是类的一个实例。要使用类，通常使用 new 操作符将类的对象实例化，然后调用类的方法来访问类的功能。
#### 2.3 extends 继承、扩展
&nbsp;&nbsp;extends 关键字用在 class 或 interface 声明中，用于指示所声明的类或接口是其所继承的类或接口的子类。子类继承父类的所有 public 和 protected 变量和方法。子类可以重写父类的任何非 final 方法。一个类只能扩展一个其他类。
#### 2.4 final 最终、不可改变
&nbsp;&nbsp;final 修饰类：该类不可被继承，而且该类的所有成员都会被隐式地指定为 final 方法。   
&nbsp;&nbsp;final 修饰方法：可锁定方法，防止任何继承类修改它的含义；类的 private 方法会隐式地被指定为 final 方法。   
&nbsp;&nbsp;final 修饰变量：这是 final 用的最多的地方。对于一个 final 变量，如果是基本数据类型的变量，则其数值在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
&nbsp;&nbsp;final 修饰的引用对象不能只指向另一个对象，但是该对象内部的内容是可变的。
#### 2.5 implements 实现
&nbsp;&nbsp;implements 声明的类必须提供在接口中所声明的所有方法的实现，一个类可以实现多个接口。
#### 2.6 static 静态
&nbsp;&nbsp;静态方法一般用于工具类中。
&nbsp;&nbsp;static 的主要目的就是创建独立于具体对象的域变量与方法，也就是没有创建对象也能调用方法。   
&nbsp;&nbsp;在加载类的同时加载 static 修饰的部分,静态变量不能引用非静态方法，因为在加载静态的时候，非静态的方法，变量还不存在。   
static 用法：
1. 静态导入：静态导入，就是把一个静态变量或者静态方法一次性导入，导入后可以直接使用该方法或者变量，而不再需要写对象名。如：import static java.lang.Math.*;一般不建议这样使用，使用哪个方法就导入哪个方法 import static java.lang.Math.PI;不要滥用静态导入。
2. 静态变量：静态变量属于类，在内存中只有一个实例。所在类被加载时，就会为该静态变量分配空间，实例变量属于对象，只有对象创建后才会分配空间，所以，当某一个变量会经常被外部代码访问时，可以考虑设计为静态。
3. 静态方法：静态方法的理解与静态变量相似。静态方法中不能使用 this 和 super 关键字。用途：单例模式 

   ```java
    public class Singleton {
        private static Singleton singleton;
    
        public static Singleton getInstance() {
            if (singleton == null) {
                singleton = new Singleton();
            }
            return singleton;
        }
    
        private Singleton() {
    
        }
    }
   ```
   单例模式的特点是一个类只能有一个实例，为了实现这一功能，必须隐藏该类的构造函数，即把构造函数声明为private，并提供一个创建对象的方法。
4. 静态代码段：静态代码块就是用static修饰的用{}括起来的代码段，它的主要目的就是对静态属性进行初始化。JVM加载类时会执行这些静态的代码块，如果static代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。
   ```java
    public class Person{
        private Date birthDate;
        private static Date startDate,endDate;
        static{
            startDate = Date.valueOf("1990");
            endDate = Date.valueOf("1999");
        }
    
        public Person(Date birthDate) {
            this.birthDate = birthDate;
        }
    
        boolean isBornBoomer() {
            return birthDate.compareTo(startDate)>=0 && birthDate.compareTo(endDate) < 0;
        }
    }
   ```
   所以，我们将一些只需要进行一次的初始化操作都放在static代码块中进行。
5. 静态内部类：只有内部类才能被static修饰，普通的类不可以。被static修饰的内部类，它可以不依赖于外部类实例对象而被实例化，而通常的内部类需要在外部类实例化后才能实例化。静态内部类不能与外部类有相同的名字，不能访问外部类的普通成员变量，只能访问内部类中的静态成员和静态方法（包括私有类型）。

#### 2.7 strictfp 严格、精准
&nbsp;&nbsp;strictfp也就是精确浮点的意思。如果没有指定strictfp关键字时，Java的编译器以及运行环境在对浮点运算的表达式是采取一种近似于我行我素的行为来完成这些操作，以致于得到的结果往往无法令人满意。而一旦使用了strictfp来声明一个类、接口或者方法时，那么所声明的范围内Java的编译器以及运行环境会完全依照浮点规范IEEE-754来执行。因此如果想让浮点运算更加精确，而且不会因为不同的硬件平台所执行的结果不一致的话，那就请用关键字strictfp。
#### 2.8 synchronized 线程、同步
synchronized 关键字可以应用于方法或语句块，并为一次只应由一个线程执行的关键代码段提供保护。    
synchronized 关键字可防止代码的关键代码段一次被多个线程执行。    
如果应用于静态方法，那么，当该方法一次由一个线程执行时，整个类将被锁定。    
如果应用于实例方法，那么，当该方法一次由一个线程访问时，该实例将被锁定。    
如果应用于对象或数组，当关联的代码块一次由一个线程执行时，对象或数组将被锁定。  
#### 2.9 transient 短暂
transient 关键字可以应用于类的成员变量，以便指出该成员变量不应在包含它的类实例已序列化时被序列化。当一个对象被串行化的时候，transient型变量的值不包括在串行化的表示中，Java的serialization提供了一种持久化对象实例的机制。当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。
#### 2.a volatile 易失
volatile 关键字用于表示可以被多个线程异步修改的成员变量。Java 语言中的 volatile 变量可以被看作是一种 “程度较轻的 synchronized”；与 synchronized 块相比，volatile 变量所需的编码较少，并且运行时开销也较少，但是它所能实现的功能也仅是 synchronized 的一部分。

### 3.变量引用
 
#### 3.1 super 父类、超类
super 关键字用于引用使用该关键字的类的超类。作为独立语句出现的 super 表示调用超类的构造方法。   
super.<methodName>()表示调用超类的方法。只有在如下情况中才需要采用这种用法：要调用在该类中被重写的方法，以便指定应当调用在超类中的该方法。   
super 用在子类中，目的是访问直接父类(类之上最接近的超类)中被屏蔽的成员。若调用父类的构造方法，super(参数列表)方式调用时，只能用在子类构造方法体中的第一行。
#### 3.2 this 本类
this 关键字用于引用当前实例。当引用可能不明确时，可以使用 this 关键字来引用当前的实例。   
this只能用于方法方法体内。当一个对象创建后，Java虚拟机（JVM）就会给这个对象分配一个引用自身的指针，这个指针的名字就是this。因此，this只能在类中的非静态方法中使用，静态方法和静态的代码块中绝对不能出现this。   
this 使用场景:   
1. 通过this调用另一个构造方法，用法是this(参数列表)，这个仅仅用在类的构造方法中。   
2. 函数参数或者函数中的局部变量和成员变量同名时，成员变量屏蔽，此时要访问成员变量则需要用“this.成员变量名”的方式来引用成员变量。
3. 在函数中，需要引用该函数所属类的当前对象的时候直接用 this。   
总的来说，this 是指向对象本身的一个指针。