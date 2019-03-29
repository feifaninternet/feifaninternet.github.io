-----------------------------
title: 探究synchronized和Lock
tags: Thread Pool
categories: JAVA
-----------------------------
date: 2018-06-28 16:28:01

### Description
我们在JAVA中常常是使用synchronized 和 Lock 来实现线程同步，为何有两种方式，肯定是他们之间各自存在优劣，这两者之间有什么区别，在不同的场景下我们又该如何取舍呢，带着这样的问题，我开始了研究

### 区别
我用一张表来大概展示下两者之间的区别

类别                          |synchronized                           |Lock
---                           |---                                    |---
存在层次                      |Java关键字，在JVM层面上                |是一个类
锁的释放                      |1.获取锁的线程执行完同步代码释放锁；2.线程执行发生异常JVM会让线程释放锁                                         |在finally中必须释放锁，不然容易造成线程死锁
锁的获取                      |假设A线程获得锁，B线程等待，如果A线程阻塞，B线程会一直等待                                          |分情况而定，Lock有多个锁的获取方式，可以尝试获得锁，线程不用一直等待
锁的状态                      |无法判断                                          |可以判断
锁的类型                      |可重入，不可中断，非公平                                          |可重入，可判断，可公平
性能比较                      |少量同步                                         |大量同步

### synchronized 原理
对于 synchronized 关键字我们重点了解一下它的原理和可重入性

#### 原理
我们利用反编译来看下 synchronized 是如何来进行同步的

   ```java
    public class SynchronizedDemo {
         public void method() {
             synchronized (this) {
                System.out.println("Method 1 start");
             }
         }
     }
   ```
反编译结果
![反编译结果](/picture/javaSynchronized.png)

这里我们引入Java对象头概念   
每个对象有一个监视器锁(monitor),当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权
1. 如果monitor的进入数为0，则该线程进入monitor，然后进入数设置为1，该线程即为monitor所有者
2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1
3. 如果其他线程已经占有率monitor，则该线程进入的等待(阻塞)状态，直到monitor的进入数为0，再尝试重新获取monitor的所有权

线程执行monitorexit
1. 执行该指令的线程必须是monitor的所有者
2. 指令执行时monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不在是所有者，其他被这个monitor阻塞的线程可以尝试获取所有权

synchronized的底层是通过monitor对象来完成的，wait/notify方法也依赖于monitor，这就是为什么只有在同步的块或者方法中才能调用wait/notify   
我们来看下同步方法反编译

   ```java
    public class SynchronizedMethod {
         public synchronized void method() {
            System.out.println("Hello World!");
        }
     }
   ```
反编译结果
![方法反编译](/picture/synchronizedMethod.png)

同步方法的synchronized实现并没有通过指令monitorenter和monitorexit来完成，对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符，JVM根据该标识符来实现同步:当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。

#### 可重入性
前面已经提到了synchronized的锁的获取，当一个线程拥有锁之后，其他的线程会阻塞，但是当拥有锁的这个线程再次请求自己持有的对象锁的临界资源时，这种情况就属于重入锁，synchronized是具有可重入性的，允许一个线程得到一个对象锁后再次请求该对象锁

   ```java
    public class AccountingSync implements Runnable{
        static AccountingSync instance=new AccountingSync();
        static int i=0;
        static int j=0;
        @Override
        public void run() {
            for(int j=0;j<1000000;j++){
    
                //this,当前实例对象锁
                synchronized(this){
                    i++;
                    increase();//synchronized的可重入性
                }
            }
        }
    
        public synchronized void increase(){
            j++;
        }
    
    
        public static void main(String[] args) throws InterruptedException {
            Thread t1=new Thread(instance);
            Thread t2=new Thread(instance);
            t1.start();t2.start();
            t1.join();t2.join();
            System.out.println(i);
        }
    }
   ```
当子类继承父类时，子类也可以通过重入锁调用父类的同步方法，每次重入monitor计数器都会加1。

### Lock
Lock是一个接口

   ```java
    public interface Lock {
        void lock();
        void lockInterruptibly() throws InterruptedException;
        boolean tryLock();
        boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
        //释放锁
        void unlock();
        Condition newCondition();
    }
   ```

1.lock(): 这是平时使用最多的一个方法，用来获取锁，如果锁已经被其他线程获取，则进行等待。
使用Lock必须要主动释放锁，即使发生异常锁也不会自动释放，因此，使用Lock必须在try,catch块中进行，并且释放锁的操作放在finally块进行，以保证所一定被释放，防止死锁的发生

   ```java
    Lock lock = ...;
    lock.lock();
    try{
        //处理任务
    }catch(Exception ex){
         
    }finally{
        lock.unlock();   //释放锁
    }
   ```

2.tryLock(): 这个方式是有返回值的它表示用来尝试获取锁，如果获取成功则返回true，如果获取失败则返回false，即使获取不到锁也会立即返回，不会一直等待

   ```java
    Lock lock = ...;
    if(lock.tryLock()) {
         try{
             //处理任务
         }catch(Exception ex){
             
         }finally{
             lock.unlock();   //释放锁
         } 
    }else {
        //如果不能获取锁，则直接做其他事情
    }
   ```
3.lockInterruptibly(): 通过这个方法获取锁时，线程可以中断等待状态，由于lockInterruptibly()的声明中抛出了异常，所以lock.lockInterruptibly()必须放在try块中或者在调用lockInterruptibly()的方法外声明抛出InterruptedException。

   ```java
    public void method() throws InterruptedException {
        lock.lockInterruptibly();
        try {  
         //.....
        }
        finally {
            lock.unlock();
        }  
    }
   ```
interrupt()方法只能中断阻塞中的线程，不能中断正在运行过程中的线程。

### ReentrantLock
可重入锁。ReentrantLock是唯一实现了Lock接口的类，下面我们来看看运用   
我们需要将lock声明为类的属性，这样才是正确的使用方法

   ```java
    public class Test {
        private ArrayList<Integer> arrayList = new ArrayList<Integer>();
        private Lock lock = new ReentrantLock();    //注意这个地方
        public static void main(String[] args)  {
            final Test test = new Test();
             
            new Thread(){
                public void run() {
                    test.insert(Thread.currentThread());
                };
            }.start();
             
            new Thread(){
                public void run() {
                    test.insert(Thread.currentThread());
                };
            }.start();
        }  
         
        public void insert(Thread thread) {
            lock.lock();
            try {
                System.out.println(thread.getName()+"得到了锁");
                for(int i=0;i<5;i++) {
                    arrayList.add(i);
                }
            } catch (Exception e) {
                // TODO: handle exception
            }finally {
                System.out.println(thread.getName()+"释放了锁");
                lock.unlock();
            }
        }
    }
   ```
下面我们来看一下tryLock()的使用方法

   ```java
    public class Test {
        private ArrayList<Integer> arrayList = new ArrayList<Integer>();
        private Lock lock = new ReentrantLock();  
        public static void main(String[] args)  {
            final Test test = new Test();
             
            new Thread(){
                public void run() {
                    test.insert(Thread.currentThread());
                };
            }.start();
             
            new Thread(){
                public void run() {
                    test.insert(Thread.currentThread());
                };
            }.start();
        }  
         
        public void insert(Thread thread) {
            if(lock.tryLock()) {
                try {
                    System.out.println(thread.getName()+"得到了锁");
                    for(int i=0;i<5;i++) {
                        arrayList.add(i);
                    }
                } catch (Exception e) {
                    // TODO: handle exception
                }finally {
                    System.out.println(thread.getName()+"释放了锁");
                    lock.unlock();
                }
            } else {
                System.out.println(thread.getName()+"获取锁失败");
            }
        }
    }
   ```
lockInterruptibly()中断等待状态

   ```java
    public class Test {
        private Lock lock = new ReentrantLock();   
        public static void main(String[] args)  {
            Test test = new Test();
            MyThread thread1 = new MyThread(test);
            MyThread thread2 = new MyThread(test);
            thread1.start();
            thread2.start();
             
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            thread2.interrupt();
        }  
         
        public void insert(Thread thread) throws InterruptedException{
            lock.lockInterruptibly();   //注意，如果需要正确中断等待锁的线程，必须将获取锁放在外面，然后将InterruptedException抛出
            try {  
                System.out.println(thread.getName()+"得到了锁");
                long startTime = System.currentTimeMillis();
                for(    ;     ;) {
                    if(System.currentTimeMillis() - startTime >= Integer.MAX_VALUE)
                        break;
                    //插入数据
                }
            }
            finally {
                System.out.println(Thread.currentThread().getName()+"执行finally");
                lock.unlock();
                System.out.println(thread.getName()+"释放了锁");
            }  
        }
    }
     
    class MyThread extends Thread {
        private Test test = null;
        public MyThread(Test test) {
            this.test = test;
        }
        @Override
        public void run() {
             
            try {
                test.insert(Thread.currentThread());
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName()+"被中断");
            }
        }
    }
   ```
### ReentrantReadWriteLock
ReentrantReadWriteLock实现了ReadWriteLock接口，可以通过readLock()获取读锁和writeLock()获取写锁   
相对于synchronized，它可以使2个线程同时进行读操作，大大提高了效率，而synchronized只能等待一个线程的读操作结束另一个线程的读操作才能开始

   ```java
    public class Test {
        private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
         
        public static void main(String[] args)  {
            final Test test = new Test();
             
            new Thread(){
                public void run() {
                    test.get(Thread.currentThread());
                };
            }.start();
             
            new Thread(){
                public void run() {
                    test.get(Thread.currentThread());
                };
            }.start();
             
        }  
         
        public void get(Thread thread) {
            rwl.readLock().lock();
            try {
                long start = System.currentTimeMillis();
                 
                while(System.currentTimeMillis() - start <= 1) {
                    System.out.println(thread.getName()+"正在进行读操作");
                }
                System.out.println(thread.getName()+"读操作完毕");
            } finally {
                rwl.readLock().unlock();
            }
        }
    }
   ```
### synchronized 和 lock 的选择
最后我们来总结一下如何取舍 :   
在性能上来说，如果资源竞争不激烈，两者的性能是差不多的，而当线程之间的竞争非常激烈时，此时Lock的性能优势就会展现出来，所以结果显而易见。如果需要仔细了解线程相关知识，我推荐大家去看一下人手一本的《JAVA并发编程实战》

