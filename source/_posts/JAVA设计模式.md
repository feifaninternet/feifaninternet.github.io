------------------
title: JAVA设计模式
tags: JAVA
categories: Expand
------------------
date: 2018-04-03 17:32:15

### 分类
- 创建型模式(5种)：工厂方法模式，抽象工厂模式，单例模式，建造者模式，原型模式。
- 结构型模式(7种)：适配器模式，装饰器模式，代理模式，外观模式，桥接模式，组合模式，享元模式。
- 行为性模式(11种)：策略模式，模板方法模式，观察者模式，迭代子模式，责任链模式，命令模式，备忘录模式，状态模式，访问者模式，中介者模式，解释器模式。

### 设计模式遵循的原则
1. 开闭原则：对扩展开放，对修改关闭
2. 里氏代换原则：只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。
3. 依赖倒转原则：这个是开闭原则的基础，对接口编程，依赖于抽象而不依赖于具体。
4. 接口隔离原则：使用多个隔离的接口来降低耦合度。
5. 迪米特法则(最少知道原则)：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立
6. 合成复用原则：原则是尽量使用合成/聚合的方式，而不是使用继承。继承实际上破坏了类的封装性，超类的方法可能会被子类修改。

### 1.工厂模式
常用的工厂模式是静态工厂，利用static方法，作为一种类似于常见的工具类Utils等辅助效果，一般情况下工厂类不需要实例化。

   ```java
    interface food{}
    
    class A implements food{}
    class B implements food{}
    class C implements food{}
    
    public class StaticFactory {
    
        private StaticFactory(){}
        
        public static food getA(){  return new A(); }
        public static food getB(){  return new B(); }
        public static food getC(){  return new C(); }
    }
    
    class Client{
        //客户端代码只需要将相应的参数传入即可得到对象
        //用户不需要了解工厂类内部的逻辑。
        public void get(String name){
            food x = null ;
            if ( name.equals("A")) {
                x = StaticFactory.getA();
            }else if ( name.equals("B")){
                x = StaticFactory.getB();
            }else {
                x = StaticFactory.getC();
            }
        }
    }
   ```

### 2.抽象工厂模式
一个基础接口定义了功能，每个实现接口的子类就是产品，然后定义一个工厂接口，实现了工厂接口的就是工厂，这时候，接口编程的优点就出现了，我们可以新增产品类，只需要同时新增一个工厂类，客户端就可以轻松调用新产品的代码。   
抽象工厂的灵活性就体现在，无需改动原有的代码就可以轻松的新增扩展类。

   ```java
    interface food{}
    
    class A implements food{}
    class B implements food{}
    
    interface produce{ food get();}
    
    class FactoryForA implements produce{
        @Override
        public food get() {
            return new A();
        }
    }
    class FactoryForB implements produce{
        @Override
        public food get() {
            return new B();
        }
    }
    public class AbstractFactory {
        public void ClientCode(String name){
            food x= new FactoryForA().get();
            x = new FactoryForB().get();
        }
    }
   ```

### 3.单例模式
在内部创建一个实例，构造器设置为private，所有方法均在该实例上改动，在创建上要注意类的实例化只能执行一次，可以使用Synchronized关键字或者内部类的机制来实现。

#### 懒汉单例模式
在类记载时不初始化，类加载快，获取对象速度慢。

   ```java
    public class SingletonDemo1 {
        private static SingletonDemo1 instance;
        private SingletonDemo1(){}
        /**
        * 去掉 synchronized 后线程不安全 
        */
        public static synchronized SingletonDemo1 getInstance(){
            if (instance == null) {
                instance = new SingletonDemo1();
            }
            return instance;
        }
    }
   ```
   
#### 饿汉单例模式
在类加载时就完成了初始化，类加载慢，获取对象速度快，基于classLoader机制避免了多线程的同步问题。

   ```java
    public class SingletonDemo2 {
        private static SingletonDemo2 instance = null;
        static{
            instance = new SingletonDemo4();
        }
        private SingletonDemo2(){}
        public static SingletonDemo2 getInstance(){
            return instance;
        }
    }
    //或者
    public class SingletonDemo3 {
        private static SingletonDemo3 instance = new SingletonDemo3();
        private SingletonDemo3(){}
        public static SingletonDemo3 getInstance(){
            return instance;
        }
    }
   ```
 
#### 静态内部类

   ```java
    public class SingletonDemo5 {
        private static class SingletonHolder{
            private static final SingletonDemo5 instance = new SingletonDemo5();
        }
        private SingletonDemo5(){}
        public static final SingletonDemo5 getInsatance(){
            return SingletonHolder.instance;
        }
    }
   ```
   
### 4.建造者模式

   ```java
    public class Builder {
    
        static class Student{
            String name = null ;
            int number = -1 ;
            String sex = null ;
            int age = -1 ;
            String school = null ;
    
            //构建器，利用构建器作为参数来构建Student对象
            static class StudentBuilder{
                String name = null ;
                int number = -1 ;
                String sex = null ;
                int age = -1 ;
                String school = null ;
                public StudentBuilder setName(String name) {
                    this.name = name;
                    return  this ;
                }
    
                public StudentBuilder setNumber(int number) {
                    this.number = number;
                    return  this ;
                }
    
                public StudentBuilder setSex(String sex) {
                    this.sex = sex;
                    return  this ;
                }
    
                public StudentBuilder setAge(int age) {
                    this.age = age;
                    return  this ;
                }
    
                public StudentBuilder setSchool(String school) {
                    this.school = school;
                    return  this ;
                }
                public Student build() {
                    return new Student(this);
                }
            }
    
            public Student(StudentBuilder builder){
                this.age = builder.age;
                this.name = builder.name;
                this.number = builder.number;
                this.school = builder.school ;
                this.sex = builder.sex ;
            }
        }
    
        public static void main( String[] args ){
            Student a = new Student.StudentBuilder().setAge(13).setName("LiHua").build();
            Student b = new Student.StudentBuilder().setSchool("sc").setSex("Male").setName("ZhangSan").build();
        }
    }
   ```
   
### 5. 原型模式
原型模式就是将一个对象作为原型，使用clone()方法来创建新的实例

#### 浅克隆/浅拷贝

   ```java
    public class Prototype implements Cloneable{
    
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        @Override
        protected Object clone()   {
            try {
                return super.clone();
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }finally {
                return null;
            }
        }
    
        public static void main ( String[] args){
            Prototype pro = new Prototype();
            Prototype pro1 = (Prototype)pro.clone();
        }
    }
   ```

只克隆了对象，克隆的对象仍然保留了原有对象的引用，值随着改变而改变

#### 深克隆/深拷贝

   ```java
     public class Prototype implements Cloneable{
        
            private String name;
        
            public String getName() {
                return name;
            }
        
            public void setName(String name) {
                this.name = name;
            }
        
            @Override
            protected Object clone()   {
                try {
                    Object obj = super.clone();
                    Prototype p = (Prototype) obj;
                    p.name = this.name.clone();
                    return p;
                } catch (CloneNotSupportedException e) {
                    e.printStackTrace();
                }finally {
                    return null;
                }
            }
        
            public static void main ( String[] args){
                Prototype pro = new Prototype();         
                Prototype pro1 = (Prototype)pro.clone();
            }
        }
   ```
克隆对象，并且把该对象的所有属性也克隆出一份新的，克隆的对象的属性不会随着改变而改变。

### 6.适配器模式
适配器模式的作用就是在原来的类上提供新功能,主要可分为3种：
- 类适配：创建新类，继承源类，并实现新接口

   ```java
    class  adapter extends oldClass  implements newFunc{}
   ```
   
- 对象适配：创建新类持源类的实例，并实现新接口

   ```java
    class adapter implements newFunc { private oldClass oldInstance ;}
   ```
   
- 接口适配：创建新的抽象类实现旧接口方法

   ```java
    abstract class adapter implements oldClassFunc { void newFunc();}
   ```
   
### 7.装饰模式
给一类对象增加新的功能，装饰方法与具体的内部逻辑无关

   ```java
      interface Source{ void method();}
      public class Decorator implements Source{
      
          private Source source ;
          public void decotate1(){
              System.out.println("decorate");
          }
          @Override
          public void method() {
              decotate1();
              source.method();
          }
      }
   ```
   
8. 代理模式
客户端通过代理类访问，代理类实现具体的实现细节，客户只需要使用代理类即可实现操作，这种模式可以对旧功能进行代理，用一个代理类调用原有的方法，且对产生的结果进行控制。

   ```java
    interface Source{void method();}
    class OldClass implements Source{
       @Override
       public void method(){
        
       }
    }
    
    class Proxy implements Source{
       private Source source = new OldClass();
       
       void doSomething(){}
       @Override
       public void method(){
           new Class1().Func1();
           source.method();
           new Class2().Func2();
           doSomething();
       }
    }
   ```
   
9. 外观模式
为子系统中的一组接口提供一个一致的界面，定义一个高层接口，这个接口时的这一子系统更加容易使用。

   ```java
    public class Facade {
        private subSystem1 subSystem1 = new subSystem1();
        private subSystem2 subSystem2 = new subSystem2();
        private subSystem3 subSystem3 = new subSystem3();
        
        public void startSystem(){
            subSystem1.start();
            subSystem2.start();
            subSystem3.start();
        }
        
        public void stopSystem(){
            subSystem1.stop();
            subSystem2.stop();
            subSystem3.stop();
        }
    }
   ```
   
10.桥接模式
桥接模式是将抽象部分与它的实现部分分离，使它们都可以独立地变化。

   ```java
    interface DrawAPI {
        void drawCircle(int radius, int x, int y);
    }
    class RedCircle implements DrawAPI {
        @Override
        public void drawCircle(int radius, int x, int y) {
            System.out.println("Drawing Circle[ color: red, radius: "
                    + radius +", x: " +x+", "+ y +"]");
        }
    }
    class GreenCircle implements DrawAPI {
        @Override
        public void drawCircle(int radius, int x, int y) {
            System.out.println("Drawing Circle[ color: green, radius: "
                    + radius +", x: " +x+", "+ y +"]");
        }
    }
    
    abstract class Shape {
        protected DrawAPI drawAPI;
        protected Shape(DrawAPI drawAPI){
            this.drawAPI = drawAPI;
        }
        public abstract void draw();
    }
    
    class Circle extends Shape {
        private int x, y, radius;
    
        public Circle(int x, int y, int radius, DrawAPI drawAPI) {
            super(drawAPI);
            this.x = x;
            this.y = y;
            this.radius = radius;
        }
    
        public void draw() {
            drawAPI.drawCircle(radius,x,y);
        }
    }
    
    //客户端使用代码
    class Client{
        public void drawCircle(){
             Shape redCircle = new Circle(100,100, 10, new RedCircle());
             Shape greenCircle = new Circle(100,100, 10, new GreenCircle());
             redCircle.draw();
             greenCircle.draw();
        }
       
    }
    
   ```
   
11. 组合模式
组合模式是为了表示那些层次结构，同时部分和整体也可能是一样的结构，常见的如文件或者树。

   ```java
    abstract class component{}
    
    class File extends component{String filename;}
    
    class Folder extends  component{
        component[] files ;  //既可以放文件File类，也可以放文件夹Folder类。Folder类下又有子文件或子文件夹。
        String foldername ;
        public Folder(component[] source){ files = source ;}
        
        public void scan(){
            for ( component f:files){
                if ( f instanceof File){
                    System.out.println("File "+((File) f).filename);
                }else if(f instanceof Folder){
                    Folder e = (Folder)f ;
                    System.out.println("Folder "+e.foldername);
                    e.scan();
                }
            }
        }
        
    }
   ```
 
12. 享元模式
使用共享对象的方法，用来尽可能减少内存使用量以及分享资讯。通常使用工厂类辅助，例子中使用一个HashMap类进行辅助判断，数据池中是否已经有了目标实例，如果有，直接返回，不需要多次创建重复实例。

   ```java
    abstract class flywei{ }
    
    public class Flyweight extends flywei{
        Object obj ;
        public Flyweight(Object obj){
            this.obj = obj;
        }
    }
    
    class  FlyweightFactory{
        private HashMap<Object,Flyweight> data;
    
        public FlyweightFactory(){ data = new HashMap<>();}
    
        public Flyweight getFlyweight(Object object){
            if ( data.containsKey(object)){
                return data.get(object);
            }else {
                Flyweight flyweight = new Flyweight(object);
                data.put(object,flyweight);
                return flyweight;
            }
        }
    }
   ```