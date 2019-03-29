-----------------------
title: 深入理解JAVA反射
tags: REFLECT
categories: JAVA
-----------------------
date: 2018-08-07 12:21:51

### Description
对于反射的定义，可以引用oracle官方的说明: 通过反射，java代码可以发现有关已加载类的字段，方法和构造函数的信息，并可以在安全限制内对这些字段，方法和构造函数进行操作，这种能力就是反射。   
通俗的说就是   
- 对于任意一个类，都能够知道这个类的所有属性和方法，即使为私有
- 对于任意一个对象，都能够调用它的任意一个方法和属性

### Class 类
java.lang.Class是反射能够实现的基础，是JDK提供的一个类

   ```java
    public final class Class<T> implements java.io.Serializable, GenericDeclaration, Type, AnnotatedElement {
       
    }
   ```
对于每一种类，java虚拟机都会初始化出一个Class类型的实例，每当我们编写并且编译一个新创建的类就会产生一个对应Class对象，并且这个Class对象会被保存在同名.class文件里。当我们new一个新对象或者引用静态成员变量时，Java虚拟机(JVM)中的类加载器系统会将对应Class对象加载到JVM中，然后JVM再根据这个类型信息相关的Class对象创建我们需要实例对象或者提供静态变量的引用值。

反射实例化一个对象

   ```java
   @Getter @setter
    public class Dog{
        private String name;
        private int age;
        
        public Dog(String name, int age){
            this.name = name;
            this.age = age;
        }
    }
    
    public class TestReflection{
        public void testReflection(){ 
            Class c = Dog.class;
            try {
                 //通过Class对象反射获取Animal类的构造方法
                 Constructor constructor = c.getConstructor(String.class, int.class);
                 //调用构造方法获取Animal实例
                 Dog dog = (Dog) constructor.newInstance( "Titi", 3);
            } catch (NoSuchMethodException e) {
                 e.printStackTrace();
            } catch (IllegalAccessException e) {
                 e.printStackTrace();
            } catch (InstantiationException e) {
                 e.printStackTrace();
            } catch (InvocationTargetException e) {
                 e.printStackTrace();
            }
        }
    }
   ```
这样的操作比new对象复杂多了，但是如果对象中的属性和方法为私有的，那么这个时候Class的优势就展现出来了

#### 获取Class对象的方法
1. Object.getClass(),基本类型无法使用此方法
2. .class,类的类型获取，基本类型也可以使用
3. Class.forName()，通过类的全限定名获取，基本类型无法使用
4. .TYPE,包装类使用，如Double.TYPE
5. .getSuperclass()，已经获取了一个类的Class对象，就可以通过反射获取这个类的父类的Class对象

### 通过Class获取类修饰符和类型
以HashMap为例

   ```java
    public class TestReflection {
        private static final String TAG = "Reflection";
        public void testReflection() {
            Class<?> c = HashMap.class;
            //获取类名
            System.out.println("Class : " + c.getCanonicalName());
            //获取类限定符
            System.out.println("Modifiers : " + Modifier.toString(c.getModifiers()));
            //获取类泛型信息
            TypeVariable[] tv = c.getTypeParameters();
            if (tv.length != 0) {
                StringBuilder parameter = new StringBuilder("Parameters : ");
                for (TypeVariable t : tv) {
                    parameter.append(t.getName());
                    parameter.append(" ");
                }
                System.out.println(parameter.toString());
            } else {
                System.out.println("-- No Type Parameters --");
            }
            //获取类实现的所有接口
            Type[] intfs = c.getGenericInterfaces();
            if (intfs.length != 0) {
                StringBuilder interfaces = new StringBuilder("Implemented Interfaces : ");
                for (Type intf : intfs){
                    interfaces.append(intf.toString());
                    interfaces.append(" ");
                }
                System.out.println(interfaces.toString());
            } else {
                System.out.println("-- No Implemented Interfaces --");
            }
            //获取类继承数上的所有父类
            List<Class> l = new ArrayList<>();
            printAncestor(c, l);
            if (l.size() != 0) {
                StringBuilder inheritance = new StringBuilder("Inheritance Path : ");
                for (Class<?> cl : l){
                    inheritance.append(cl.getCanonicalName());
                    inheritance.append(" ");
                }
                System.out.println(inheritance.toString());
            } else {
                System.out.println("-- No Super Classes --%n%n");
            }
            //获取类的注解(只能获取到 RUNTIME 类型的注解)
            Annotation[] ann = c.getAnnotations();
            if (ann.length != 0) {
                StringBuilder annotation = new StringBuilder("Annotations : ");
                for (Annotation a : ann){
                    annotation.append(a.toString());
                    annotation.append(" ");
                }
                System.out.println(annotation.toString());
            } else {
                System.out.println("-- No Annotations --%n%n");
            }
        }
        private static void printAncestor(Class<?> c, List<Class> l) {
            Class<?> ancestor = c.getSuperclass();
            if (ancestor != null) {
                l.add(ancestor);
                printAncestor(ancestor, l);
            }
        }
    }
   ```
   
### Member 接口
Member的三个实现类
1. [java.lang.reflect.Field()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Field.html) ：对应类变量
2. [java.lang.reflect.Method()](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fapi%2Fjava%2Flang%2Freflect%2FMethod.html) ：对应类方法
3. [java.lang.reflect.Constructor](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html) ：对应类构造函数

反射就是通过这三个类才能在运行时改变对象状类，下面我们分别来了解一下

#### Field
通过Field你可以访问给定对象的类变量，包括获取变量的类型，修饰符，注解，变量名，变量的值或者重新设置变量值，即使变量是private的。

获取Field
1. [getDeclaredField(String name)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredField-java.lang.String-)：获取指定的变量，只要是声明的变量都能获得，包括private
2. [getField(String name)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getField-java.lang.String-)：获取指定的变量（只能获得public的）
3. [getDeclaredFields()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredFields--)：获取所有声明的变量（包括private）
4. [getFields()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getFields--)：获取所有的public变量

获取变量，修饰符，注解

   ```java
    public void testField(){
            Class c = Cat.class;
            Field[] fields = c.getDeclaredFields();
            for(Field f : fields){
                StringBuilder builder = new StringBuilder();
                //获取名称
                builder.append("filed name = ");
                builder.append(f.getName());
                //获取类型
                builder.append(" type = ");
                builder.append(f.getType());
                //获取修饰符
                builder.append(" modifiers = ");
                builder.append(Modifier.toString(f.getModifiers()));
                //获取注解
                Annotation[] ann = f.getAnnotations();
                if (ann.length != 0) {
                    builder.append(" annotations = ");
                    for (Annotation a : ann){
                        builder.append(a.toString());
                        builder.append(" ");
                    }
                } else {
                    builder.append("  -- No Annotations --");
                }
               System.out.println(builder.toString());
            }
        }
   ```
获取设置变量值

   ```java
    public void testField(){
            Cat cat = new Cat("Tom", 2);
            Class c = cat.getClass();
            try {
                //注意获取private变量时，需要用getDeclaredField
                Field fieldName = c.getDeclaredField("name");
                Field fieldAge = c.getField("age");
                //反射获取名字, 年龄
                String name = (String) fieldName.get(cat);
                int age = fieldAge.getInt(cat);
                System.out.println("before set, Cat name = " + name + " age = " + age);
                //反射重新set名字和年龄
                fieldName.set(cat, "Timmy");
                fieldAge.setInt(cat, 3);
                System.out.println("after set, Cat " + cat.toString());
            } catch (NoSuchFieldException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
   ```
如果有属性为私有，上面会报错，我们可以使用[setAccessible(boolean flag)](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AccessibleObject.html#setAccessible-boolean-)取消java语言的访问权限检查，任何继承[AccessibleObject](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AccessibleObject.html)的类都可以使用此方法来去除检查，去除检查之后便可操作私有属性

### Method
通过反射获取对象的方法
1. [getDeclaredMethod(String name, Class<?>... parameterTypes)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethod-java.lang.String-java.lang.Class...-)：根据方法名获得指定的方法，参数name为方法名，parameterTypes为参数类型
2. [getMethod(String name, Class<?>... parameterTypes)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethod-java.lang.String-java.lang.Class...-)：根据方法名获取指定的public方法
3. [getDeclaredMethods()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredMethods--)：获取所有声明的方法
4. [getMethods()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getMethods--)：获取所有的public方法

其他方法
1. [getReturnType()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#getReturnType--)：获取方法返回类型对应的Class对象
2. [getParameterTypes()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#getParameterTypes--)：获取方法参数类型
3. [getExceptionTypes()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#getExceptionTypes--)：获取目标方法抛出的异常类型对应的Class对象

获取方法参数名称   
.class文件中默认不存储方法参数名称，如果想要获取方法参数名称，需要在编译的时候加上-parameters参数

   ```java
    //这里的m可以是普通方法Method，也可以是构造方法Constructor
    //获取方法所有参数
    Parameter[] params = m.getParameters();
    for (int i = 0; i < params.length; i++) {
        Parameter p = params[i];
        p.getType();   //获取参数类型
        p.getName();  //获取参数名称，如果编译时未加上`-parameters`，返回的名称形如`argX`, X为参数在方法声明中的位置，从0开始
        p.getModifiers(); //获取参数修饰符
        p.isNamePresent();  //.class文件中是否保存参数名称, 编译时加上`-parameters`返回true,反之flase
    }
   ```
   
通过反射调用方法   
反射通过Method的invoke()方法来调用目标方法，第一个参数为需要调用的目标类对象，如果方法为static，则该参数为null，后面的参数为目标方法的参数值，顺序与目标方法声明一样

   ```java
    public native Object invoke(Object obj, Object... args)
                throws IllegalAccessException, IllegalArgumentException, InvocationTargetException
   ```
*注意：如果方法是private的，可以使用method.setAccessible(true)方法绕过权限检查*

示例

   ```java
    Class<?> c = Cat.class;
     try {
         //构造Cat实例
         Constructor constructor = c.getConstructor(String.class, int.class);
         Object cat = constructor.newInstance( "Jack", 3);
         //调用无参方法
         Method sleep = c.getDeclaredMethod("sleep");
         sleep.invoke(cat);
         //调用定项参数方法
         Method eat = c.getDeclaredMethod("eat", String.class);
         eat.invoke(cat, "grass");
         //调用不定项参数方法
         //不定项参数可以当成数组来处理
         Class[] argTypes = new Class[] { String[].class };
         Method varargsEat = c.getDeclaredMethod("eat", argTypes);
         String[] foods = new String[]{
              "grass", "meat"
         };
         varargsEat.invoke(cat, (Object)foods);
      } catch (InstantiationException | IllegalAccessException | NoSuchMethodException | InvocationTargetException e) {
         e.printStackTrace();
     }
   ```
被调用方法所抛出的异常在反射中都会以InvocationTargetException抛出

### Constructor
获取构造方法
1. [getDeclaredConstructor(Class<?>... parameterTypes)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructor-java.lang.Class...-)：获取指定构造函数，参数parameterTypes为构造方法的参数类型
2. [getConstructor(Class<?>... parameterTypes)](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructor-java.lang.Class...-)：获取指定public构造函数，参数parameterTypes为构造方法的参数类型
3. [getDeclaredConstructors()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getDeclaredConstructors--)：获取所有声明的构造方法
4. [getConstructors()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#getConstructors--)：获取所有的public构造方法

我们可以使用反射来创建对象

1. [java.lang.reflect.Constructor.newInstance()](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Constructor.html#newInstance-java.lang.Object...-)
2. [Class.newInstance()](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html#newInstance--)

这两中方法的异同之处

1. Class.newInstance()仅可用来调用无参的构造方法；Constructor.newInstance()可以调用任意参数的构造方法。
2. Class.newInstance()会将构造方法中抛出的异常不作处理原样抛出;Constructor.newInstance()会将构造方法中抛出的异常都包装成InvocationTargetException抛出。
3. Class.newInstance()需要拥有构造方法的访问权限;Constructor.newInstance()可以通过setAccessible(true)方法绕过访问权限访问private构造方法。

利用反射初始化数组
[java.lang.reflect.Array](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Array.html)

   ```java
    //创建数组， 参数componentType为数组元素的类型，后面不定项参数的个数代表数组的维度，参数值为数组长度
    Array.newInstance(Class<?> componentType, int... dimensions)
    
    //设置数组值，array为数组对象，index为数组的下标，value为需要设置的值
    Array.set(Object array, int index, int value)
    
    //获取数组的值，array为数组对象，index为数组的下标
    Array.get(Object array, int index)
   ```
   
   ```java
    Object array = Array.newInstance(int.class, 2);
    Array.setInt(array , 0, 1);
    Array.setInt(array , 1, 2);
   ```
反射支持数组自动变宽，但不能自动变窄,你可以理解为set方法在int类型数组中可以set short类型的数据，但是不能set long类型的数据，否则会报IllegalArgumentException

### 反射的缺点
- 内部曝光：反射允许代码执行一些在正常情况下不被允许的操作，比如私有属性的访问，破坏抽象性
- 安全限制：使用反射技术要求程序必须在一个没有安全限制的环境中运行，
- 性能开销：反射涉及类型动态解析，所以其效率比较低

原则：如果使用常规方法能够实现，那么就尽量不要使用反射