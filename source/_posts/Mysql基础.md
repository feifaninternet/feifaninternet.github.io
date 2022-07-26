title: Mysql基础
tags: mysql
categories: Database
date: 2021-05-15 16:28:01
-----------------------------

MySQL是在开发过程中使用的最多的一个关系型数据库。所以了解和掌握对它的调优是很有必要的。

MySQL数据类型
MySQL的支持哪些数据类型，对应的大小及用途等信息，附上几个表格。

- 数字

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/database/mysql01.png" alt="数据类型" title="数据类型">
</div>

- 字符

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/database/mysql02.png" alt="数据类型" title="数据类型">
</div>

- 日期

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/database/mysql03.png" alt="数据类型" title="数据类型">
</div>

引擎
MySQL的官方引擎主要有两个：MyISAM和InnoDB。

MyISAM最大的特点是支持全文索引、查询效率较高。缺点是不支持事物、表级锁。

InnoDB是MySQL5.5之后MySQL默认引擎。特点是支持ACID事务、支持外键、支持行级锁提高了并发效率。

有关事务、外键、锁下面会一一介绍。

数据库事务
数据库中事务的特性：ACID，这个也是上面说的InnoDB支持ACID事务。

ACID分别为：原子性、一致性、隔离性、持久性。

原子性：事务的操作要么全部成功要么失败回滚。

一致性：事务不能破坏数据库的一致性和完整性。例如事务操作了多张表，要么多张表全是操作前的数据要么多张表全是操作后的数据。

隔离性：隔离性是针对并发问题，多用户的事务相互隔离。

持久性：事务提交成功后对数据库的修改是永久的。

数据库并发
刚才提到事务的隔离性是针对并发问题的。如果对事务没有进行隔离会出现什么问题呢？

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/database/mysql03.png" alt="示意图" title="示意图">
</div>


如果没有进行事务隔离，则将会出现脏读、不可重复度、幻读等情况，分别说明一下。

脏读：事物A读取到了事物B未提交的数据。

不可重复读：事物A多次读区数据库中同一条数据时返回不同的数据。

幻读：事物A执行完全相同的语句时返回结果集不同。与不可重复的区别是不可重复是读区一条数据。幻读是由于增加和删除记录导致的。

不同的事务隔离级别能解决不同的问题，如上图所示。MySQL默认是可重复读级别。

事务分类
事务分为：扁平化事务、带保存点扁平化事务、链事务、嵌套事务、分布式事务。

扁平化事务：要么全部成功，要么全部回滚。这个也是使用最多的。

带保存点扁平化事务：是在事务执行过程中插入保存点，事务失败时回滚到保存点。

链事务：可以回滚到最近的保存点。和带保存点扁平化事务的区别在于带保存点扁平化事务可以回滚到任意保存点。

嵌套事务：事务的嵌套，如果上层事务回滚所有事务回滚。

分布式事务：分布式系统的扁平化事务，这个块内容很多后面可以单独拿出来写一篇。

MySQL锁
上面说过InnoDB是行级锁，MyISAM是表级锁。下面介绍一下几个常见的锁。

行级锁：锁的颗粒度小，并发效率高，可能出现锁死。

表级锁：锁的颗粒度大，并发效率低，无锁死。

共享锁：也叫读锁，就是其他事务可以读就是不能写。上锁语句：lock in share mode。

排他锁：排他锁就是写锁，其他事务不能读取，也不能写。UPDATE、DELETE 和 INSERT 语句，InnoDB 会自动给涉及的数据集加排他锁。上锁语句：select for update

需要说明的是：行级锁和表级锁是锁的颗粒度，而共享锁和排他锁是锁的策略。

外键
InnoDB支持外键，那外键是什么呢？简单介绍一下。

外键：A表保存的B表的主键id。

可能会想，这个不是一直都这样吗？其实不是的一般情况下，外键开发人员知道，但是数据库不知道，现在数据库知道了以后就可以帮我们确保数据的完整性和一致性。比如如果删除主表的那从表自动删除等等。

索引
索引是我们最常用的优化方式，可以大幅提升数据库的查询性能。当然它也有代价，代价就是牺牲磁盘空间以及新增、修改、删除等操作增加额外开销。

索引分为：唯一索引、主键索引、普通索引、联合索引、全文索引

唯一索引：全表唯一，允许未空。

主键索引：全表唯一，不允许未空。

普通索引：允许索引列值相同。

联合索引：是由多个列组成的索引，并不是多个索引组成。联合索引遵循《最左原则》所以选择列时顺序很重要。

全文索引：只能在 CHAR、VARCHAR、TEXT 类型字段上使用，底层使用倒排索引实现。这个慎用，会非常消耗磁盘资源。

MySQL调优
MySQL的调优是经常用到的所有很有必要掌握，大概调优思路如下：

表结构+索引  -> SQL优化  ->MySQL参数优化  ->  硬件及系统配置

当然最有效果的是最右边的方案，但也是成本最高的。

下面罗列一下优化方案：

1. 要在设计表结构时，考虑数据库的水平与垂直扩展能力。

2. 要选择合适的字段大小及类型。

3. 避免一个表子段过多。

4. 做适当的反范式，注意适当的。

5. 对经常查询的维度要做好索引。

6. 列尽量设置为not null，因为MySQL对可以为null的列无法做查询优化，允许未null使得索引统计更加复杂，以及占用更多的空间。

7. 通过慢查询日志来针对性的优化。

8. 学会使用Explain来分析语句。

9. 尽量使用索引排序。