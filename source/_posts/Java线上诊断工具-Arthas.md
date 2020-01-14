-----------------------------
title: Java 线上诊断工具 Arthas
date: 2019-05-17 14:56:40
tags: Arthas
categories: Expand
-----------------------------

### Description
对于很多线上发生的问题，我们无法在线上debug，线下也无法重现，这个时候我们便可以使用这个工具来进行诊断，它可以监控jvm实时运行状态，
可以观察指定类中的某个方法的内部调用路径，也可以对监控后的请求进行重现

### 相关命令和使用场景

更多命令查阅[官方文档](https://alibaba.github.io/arthas/install-detail.html)

#### 1.启动

   ```
    java -jar arthas-boot.jar
    
    #打印更多详细信息
    java -jar arthas-boot.jar -h   
   ```

如果有多个应用，输入相应数字选择

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/aras启动.jpg" alt="aras启动" title="aras启动">
</div>

#### 2.查看当前进程信息

**[dashboard命令](https://alibaba.github.io/arthas/dashboard.html#dashboard)**

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/dashboard.jpg" alt="dashboard" title="dashboard">
</div>
   
信息会自动刷新，cpu使用比例，堆内存使用情况，应用GC次数，应用GC耗时都很清楚。   
`ctrl+c` 可中断执行，可查看`main`方法的线程id，通过`thread id`的命令来获取`Main Class`

#### 3.查看进程信息

**[thread命令](https://alibaba.github.io/arthas/thread.html#thread)**: 查看当前线程信息，查看线程的堆栈

   ```
    #显示所有线程信息
    thread
    
    #打印线程ID 1的栈
    thread 1
    
    #显示最忙的前n个线程，打印堆栈
    thread -n 3
    
    #找出阻塞其他线程的线程
    thread -b
   ```

#### 4.方法执行数据观测

**[watch命令](https://alibaba.github.io/arthas/watch.html#watch)**: 观察指定方法的调用情况(返回值、抛出异常、入参)

请求观察的方法后即可观察到控制台输出

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/watch演示.jpg" alt="watch演示" title="watch演示">
</div>


参数名称         |  参数说明
---             |---
class-pattern	|类名表达式匹配
method-pattern	|方法名表达式匹配
express	        |观察表达式
condition-express	|条件表达式
[b]	            |在方法调用之前观察
[e]	            |在方法异常之后观察
[s]	            |在方法返回之后观察
[f]	            |在方法结束之后(正常返回和异常返回)观察
[E]	            |开启正则表达式匹配，默认为通配符匹配
[x:]	        |指定输出结果的属性遍历深度，默认为 1

条件表达式

   ```
    $ watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0"
    Press Ctrl+C to abort.
    Affect(class-cnt:1 , method-cnt:1) cost in 68 ms.
    ts=2018-12-03 19:36:04; [cost=0.530255ms] result=@ArrayList[
        @Integer[-18178089],
        @MathGame[demo.MathGame@41cf53f9],
    ]
   ```
   
数据过滤

   ```
    $ watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2
    Press Ctrl+C to abort.
    Affect(class-cnt:1 , method-cnt:1) cost in 66 ms.
    ts=2018-12-03 19:40:28; [cost=2112.168897ms] result=@ArrayList[
        @Object[][
            @Integer[2141897465],
        ],
        @ArrayList[
            @Integer[5],
            @Integer[428379493],
        ],
    ]
   ```
   
**观察某个类以及类的属性，可实现线上断点的效果，非常方便**

1.查看方法运行前后，当前对象中的属性，可以使用`target`关键字，代表当前对象

   ```
    $ watch demo.MathGame primeFactors 'target'
    Press Ctrl+C to abort.
    Affect(class-cnt:1 , method-cnt:1) cost in 52 ms.
    ts=2018-12-03 19:41:52; [cost=0.477882ms] result=@MathGame[
        random=@Random[java.util.Random@522b408a],
        illegalArgumentCount=@Integer[13355],
    ]
   ```

2.使用`target.field_name`访问当前对象的某个属性

   ```
    $ watch demo.MathGame primeFactors 'target.illegalArgumentCount'
    Press Ctrl+C to abort.
    Affect(class-cnt:1 , method-cnt:1) cost in 67 ms.
    ts=2018-12-03 20:04:34; [cost=131.303498ms] result=@Integer[8]
    ts=2018-12-03 20:04:35; [cost=0.961441ms] result=@Integer[8]
   ```

测试请求时，查看某个`service`中的属性和相应方法中的对象的属性

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/watch断点.jpg" alt="watch断点" title="watch断点">
</div>

#### 5.查看方法内部调用路径及耗时

**[trace命令](https://alibaba.github.io/arthas/trace.html#trace)**: 方法内部调用路径，并输出方法路径上的每个节点上耗时

可以找出某个请求速度慢的原因所在

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/trace耗时.jpg" alt="trace耗时" title="trace耗时">
</div>

查看多个类多个函数

   ```
    trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
   ```
   
#### 6.查看方法完整调用路径

**[stack命令](https://alibaba.github.io/arthas/stack.html#stack)**: 输出当前方法被调用的调用路径

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/stack.jpg" alt="stack" title="stack">
</div>

#### 7.记录指定方法的调用

**[tt命令](https://alibaba.github.io/arthas/tt.html#tt)**: 记录下指定方法的调用，后续对其进行筛选，查看，重现等

`tt`和`watch`命令我们通常都会在后台执行，并输出到相应日志文件保存，以便后续操作

在线上请求量过多的时候一定要设置`-n`的值，请求数量达到这个值后便会停止记录，将请求记录到相应日志文件中，`watch`通常也这样使用

   ```
    tt -t -n 3 com.test.testM.service.TestService test >> /data1/test.log &
   ```
   
通过`jobs`查看后台执行的任务，已经删除任务

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/jobs.jpg" alt="查看后台执行的任务" title="查看后台执行的任务">
</div>

查看所有记录

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/tt-l.jpg" alt="查看所有记录" title="查看所有记录">
</div>

对参数进行筛选

   ```
    tt -s "params[0].testId=='xfan_test_id'"
   ```
查看具体的记录信息

   ```jshelllanguage
    tt -i 1006
   ```

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/tt查看具体信息.jpg" alt="查看具体信息" title="查看具体信息">
</div>
   
重新模拟一次调用

   ```jshelllanguage
    tt -i 1006 -p
   ```

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/arthas/模拟调用.jpg" alt="模拟调用" title="模拟调用">
</div>

好了，最常用的这些命令先整理到这里，更多命令及详细使用方法查阅[官方文档](https://alibaba.github.io/arthas/install-detail.html)