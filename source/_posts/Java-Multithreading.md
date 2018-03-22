--------------------------
title: Java Multithreading
--------------------------
date: 2018-02-08 14:20:32
tags: multithreading


### Description

1. Thread: Each part of the program is called one Threads, and each thread defines an independent execution path.Multithreading is a special form of multitasking. Multithreading requires less overhead than multitasking.   
2. Process: A process includes the memory space allocated by the operating system, including one or more threads. Threads can not exist independently, it must be part of the process. A process will run until all non-waiting threads end.

#### Thread Lift Cycle
![Picture](/picture/ThreadLife.jpg)   

#### Key Words
&nbsp;&nbsp;New state: A newly generated thread begins its life cycle from the new state. It stays in this state until the program starts this thread.

&nbsp;&nbsp;Running state: When a new state thread is started, the thread becomes runnable and the thread begins executing its task in this state.

&nbsp;&nbsp;Ready state: When a thread waiting for another thread to perform a task, the thread into the ready state. When another thread to the ready state,When the thread sends a signal, the thread switches back to running.

&nbsp;&nbsp;Sleep state: This thread goes to sleep from running due to the exhaustion of one thread's time slice. When the time interval or waiting time expires.And, the state of the thread to switch to running.

&nbsp;&nbsp;Termination: A running state of the thread to complete the task or other termination conditions, the thread will switch to the termination of the state.

### Thread's priority

&nbsp;&nbsp;Every Java thread has a priority, which helps the operating system to determine the thread's scheduling order. Java priority is in MIN_PRIORITY (1) and MAX_PRIORITY (10) in the range between. By default, every thread is assigned a priority of NORM_PRIORITY (5).Threads with higher priority are more important to the program and should allocate processor time before the lower priority thread, but can not guarantee thread execution order and depend on the platform.

### Two ways to create a thread
1. By implementing the Runnable interface.
2. By inheriting the Thread class itself.

#### 1.Implements Runnable
   ```java
    class NewThread implements Runnable {  
       Thread t;  
       NewThread() {  
          // Create a second new thread  
          t = new Thread(this, "Demo Thread");  
          System.out.println("Child thread: " + t);  
          t.start();   
       }  
      
       // The second thread entry 
       public void run() {  
          try {  
             for(int i = 5; i > 0; i--) {  
                System.out.println("Child Thread: " + i);  
                Thread.sleep(50);  
             }  
         } catch (InterruptedException e) {  
             System.out.println("Child interrupted");  
         }  
         System.out.println("Exiting child thread");  
       }  
    }  
      
    public class ThreadDemo {  
       public static void main(String args[]) {  
          // create a new thread 
          new NewThread();   
          try {  
             for(int i = 5; i > 0; i--) {  
               System.out.println("Main Thread: " + i);  
               Thread.sleep(100);  
             }  
          } catch (InterruptedException e) {  
             System.out.println("Main thread interrupted");  
          }  
          System.out.println("Main thread exiting");  
       }
    }  
   ```
#### 2.Extends thread class
   ```java
    class NewThread extends Thread {  
       NewThread() {  
          // Create a second new thread   
          super("Demo Thread");  
          System.out.println("Child thread: " + this);  
          start();   
       }  
        
       // The second thread entry 
       public void run() {  
          try {  
             for(int i = 5; i > 0; i--) {  
                System.out.println("Child Thread: " + i);  
                Thread.sleep(50);  
             }  
          } catch (InterruptedException e) {  
             System.out.println("Child interrupted.");  
          }  
          System.out.println("Exiting child thread.");  
       }  
    }  
        
    public class ExtendsThread {  
       public static void main(String args[]) {  
          // create a new thread 
          new NewThread();   
          try {  
             for(int i = 5; i > 0; i--) {  
                System.out.println("Main Thread: " + i);  
                Thread.sleep(100);  
             }  
          } catch (InterruptedException e) {  
             System.out.println("Main thread interrupted.");  
          }  
          System.out.println("Main thread exiting.");  
       }
    }
   ```
   
   ```
    output:
        Child thread: Thread[Demo Thread, 5, main]
        Main Thread: 5
        Child Thread: 5
        Child Thread: 4
        Main Thread: 4
        Child Thread: 3
        Child Thread: 2
        Main Thread: 3
        Child Thread: 1
        Exiting child thread
        Main Thread: 2
        Main Thread: 1
        Main thread exiting
   ```
### Thread class method
1. Executed using this thread

   ```
    public void start()
   ```
2. Runnable object's run method

   ```
    public void run()
   ```
3. Change the name of the thread to the same name as the parameter

   ```
    public final void setName(String name)
   ```
4. Change the priority of the thread

   ```
    public final void setPriority(int priority)
   ```
5. Mark this thread as a daemon or user thread

   ```
    public final void setDaemon(boolean on)
   ```
6. The maximum time to wait for this thread to expire is sec milliseconds

   ```
    public final void join(long sec)
   ```
7. Interrupt thread

   ```
    public void interrupt()
   ```
8. Test whether the thread is active

   ```
    public final boolean isAlive()
   ```
#### Static method
1. Suspend the current thread and execute other threads

   ```
    public static void yield()
   ```
2. Sleep sec milliseconds

   ```
    public static void sleep(long sec)
   ```
3. True if and only if the current thread is holding the monitor lock on the specified object

   ```
    public static boolean holdsLock(Object obj)
   ```
4. Returns a reference to the currently executing thread object

   ```
    public static Thread currentThread()
   ```
5. Print the current thread's stack trace to the standard error stream

   ```
    public static void dumpStack()
   ```
   
### Some of the main concepts in threads
### 1.Thread synchronization
&nbsp;&nbsp;Having multiple threads accessing a variable or object at the same time, having both read and write operations in those threads can cause confusion in the value of the variable or the state of the object, causing the program to become abnormal. So add a synchronization lock to avoid being called by other threads before the thread has not completed the operation, thus ensuring the uniqueness and accuracy of the variable or object.
#### Synchronization method
1. Use the synchronized keyword modification method

   ```java
    public class Bank {  
        
        // balance
        private int count = 0;  
    
        // save money  
        public  synchronized void addMoney(int money){  
            count += money;  
            System.out.println(System.currentTimeMillis() + "Save：" + money);  
        }  
    
        // take out money  
        public  synchronized void subMoney(int money){  
            if(count - money < 0){  
                System.out.println("Insufficient balance");  
                return;  
            }  
            count -= money;  
            System.out.println(+System.currentTimeMillis() + "Take："+money);  
        }  
    
        // select  
        public void lookMoney(){  
            System.out.println("Balance：" + count);  
        }  
    }  
   ```
   ```java
    public class SyncThreadTest {  
    
        public static void main(String args[]){  
            final Bank bank = new Bank();  
    
            Thread threadAdd = new Thread(new Runnable() {  
    
                @Override  
                public void run() {  
                    // TODO Auto-generated method stub  
                    while(true){  
                        try {  
                            Thread.sleep(1000);  
                        } catch (InterruptedException e) {  
                            // TODO Auto-generated catch block  
                            e.printStackTrace();  
                        }  
                        bank.addMoney(100);  
                        bank.lookMoney();  
                        System.out.println("\n");  
    
                    }  
                }  
            });  
    
            Thread threadSub = new Thread(new Runnable() {  
    
                @Override  
                public void run() {  
                    // TODO Auto-generated method stub  
                    while(true){  
                        bank.subMoney(100);  
                        bank.lookMoney();  
                        System.out.println("\n");  
                        try {  
                            Thread.sleep(1000);  
                        } catch (InterruptedException e) {  
                            // TODO Auto-generated catch block  
                            e.printStackTrace();  
                        }     
                    }  
                }  
            });  
            threadSub.start();   
            threadAdd.start();  
        }  
    }  
   ```
2. Use special domain variables (volatile) to achieve thread synchronization
   ```
    private volatile int count = 0;
   ```

&nbsp;&nbsp;a. The volatile keyword provides a lock-free mechanism for accessing domain variables   

&nbsp;&nbsp;b. The use of volatile modified domain is equivalent to tell the virtual machine that the domain may be updated by other threads   
   
&nbsp;&nbsp;c. Instead of using the value in the register each time you use this field, recalculate   

&nbsp;&nbsp;d. Volatile does not provide any atomic operations, it can not be used to modify final type variables   

&nbsp;&nbsp;Its principle is to read the volatile modified variables every time the thread is read from memory, rather than read the cache, so each thread to access the variable values ​​are the same. This ensures synchronization.Not recommended.

3.Use re-lock to achieve thread synchronization
&nbsp;&nbsp;The ReentrantLock class is a reentrant, mutually exclusive, lock that implements the Lock interface, which has the same basic behavior and semantics as the synchronized method, and extends its capabilities.   
&nbsp;&nbsp;ReentrantLock commonly used methods:   
- Create an ReentrantLock instance
   
   ```
    ReentrantLock()
   ```
- Get lock
   
   ```
    lock()
   ```
- Release lock
   
   ```
    unlock()
   ```

Example
   ```java
    public class Bank {  
    
        private  int count = 0;
    
        // declare ReentrantLock
        private Lock lock = new ReentrantLock();  
      
        public void addMoney(int money) {  
            // lock
            lock.lock(); 
            
            try{                
                count += money;  
                System.out.println(System.currentTimeMillis() + "Save：" + money);  
            }finally{  
                // release
                lock.unlock(); 
            }  
        }  
    
        public void subMoney(int money) {  
            lock.lock();  
            try{  
    
            if (count - money < 0) {  
                System.out.println("Insufficient balance");  
                return;  
            }  
            count -= money;  
            System.out.println(+System.currentTimeMillis() + "Take：" + money);  
            }finally{  
                lock.unlock();  
            }  
        }  
    
        public void lookMoney() {  
            System.out.println("Balance：" + count);  
        }  
    }
   ```
4.Use local variables for thread synchronization
   ```java
    public class Bank {  
    
        private static ThreadLocal<Integer> count = new ThreadLocal<Integer>(){  
    
            @Override  
            protected Integer initialValue() {  
                // TODO Auto-generated method stub  
                return 0;  
            }  
    
        };  
        
    
        public void addMoney(int money) {  
            count.set(count.get()+money);  
            System.out.println(System.currentTimeMillis() + "Save：" + money);  
    
        }  
    
        public void subMoney(int money) {  
            if (count.get() - money < 0) {  
                System.out.println("Insufficient balance");  
                return;  
            }  
            count.set(count.get()- money);  
            System.out.println(+System.currentTimeMillis() + "Take：" + money);  
        }  
        
        public void lookMoney() {  
            System.out.println("Balance：" + count.get());  
        }  
    }  
   ```
&nbsp;&nbsp;When using ThreadLocal to manage variables, each thread that uses the variable gets a copy of the variable, independent of each other, so that each thread is free to modify its own copy of the variable without affecting other threads.

### Thread Communication
#### &nbsp;&nbsp;Demand: Write two threads, one thread printing 1 ~ 52, another thread printing letters A ~ Z, print order 12A34B56C ...... 5152Z, requires the use of communication between threads.
```java
   /**
   * Auxiliary tools
   */
    public enum HelpUtil {
        instance;
        private static final ExecutorService tPool = Executors.newFixedThreadPool(2);
    
        public static String[] buildNumArr(int max){
            String[] numArr = new String[max];
            for (int i = 0; i < max; i++){
                numArr[i] = Integer.toString(i + 1);
            }
            return numArr;
        }
    
        public static String[] buildCharArr(int max){
            String[] charArr = new String[max];
            int temp = 65;
            for (int i = 0; i < max; i++){
                charArr[i] = String.valueOf((char)(temp + i));
            }
            return charArr;
        }
    
        public static void print(String... input){
            if (input == null){
                return;
            }
            for (String each : input){
                System.out.print(each + " ");
            }
        }
    
        public void run(Runnable r){
            tPool.submit(r);
        }
    
        public void shutdown(){
            tPool.shutdown();
        }
    }
   ```
### 1.By sharing variables to achieve
### 1.1 Use the most basic synchronized,notify,wait
   ```java
    public class MethodOne {
    
        /*
        * 3.Use volatile
        * 4.Use AtomicInteger
        */
    
        class ThreadToGo{
            int value = 1;
        }
    
        private final ThreadToGo threadToGo = new ThreadToGo();
    
        private Runnable newThreadOne(){
            final String[] numArrInit = HelpUtil.buildNumArr(52);
            return new Runnable() {
                private String[] numArr = numArrInit;
                @Override
                public void run() {
                    try {
                        for (int i = 0;i < numArr.length; i = i+2){
                            synchronized (threadToGo){
                                while (threadToGo.value == 2){
                                    threadToGo.wait();
                                }
                                HelpUtil.print(numArr[i],numArr[i + 1]);
                                threadToGo.value = 2;
                                threadToGo.notify();
                            }
                        }
                    }catch (InterruptedException e){
                        System.out.println("Oops....");
                    }
                }
            };
        }
    
        private Runnable newThreadTwo(){
            final String[] charArrInit = HelpUtil.buildCharArr(26);
            return new Runnable() {
                private String[] charArr = charArrInit;
                @Override
                public void run() {
                    try {
                        for (int i = 0; i < charArr.length; i++){
                            synchronized (threadToGo){
                                while (threadToGo.value == 1){
                                    threadToGo.wait();
                                }
                                HelpUtil.print(charArr[i]);
                                threadToGo.value = 1;
                                threadToGo.notify();
                            }
                        }
                    }catch (InterruptedException e){
                        System.out.println("Oops....");
                    }
                }
            };
        }
    
        public static void main(String[] args) {
            MethodOne one = new MethodOne();
            HelpUtil.instance.run(one.newThreadOne());
            HelpUtil.instance.run(one.newThreadTwo());
            HelpUtil.instance.shutdown();
        }
    }
   ```
### 1.2 Use Lock and Condition 
   
