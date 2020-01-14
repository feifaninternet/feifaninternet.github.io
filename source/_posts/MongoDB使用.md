-------------------------
title: MongoDB使用
tags: MongoDB
categories: database
date: 2018-03-28 12:02:46
-------------------------

### MongoDB 命令

#### 数据库相关
- 创建数据库

   ```
    use database_name
   ```
   
如果数据库不存在则创建数据库，否则切换到指定数据库。

- 切换到指定数据库

   ```
    switched to db xfan
   ```
   
- 查看所有数据库

   ```
    show dbs
   ```
   
刚创建的数据库并不会在数据库列表中，我们需要向其中插入一些数据后它才会显示。

- 删除数据库

   ```
   db.dropDatabase()
   ```
   
先切换到需要删除的数据库，在执行这个操作。该命令为删除当前数据库，默认为test

- 删除集合

   ```
   db.collection.drop()
   ```
   
#### 文档操作
- 插入文档

   ```
   db.collection_name.insert(document)
   ```
   
示例：   
将文档存储在xfan集合中

   ```
	db.xfan.insert({
		    title: 'MongoDB命令',
		    by: 'xfan',
		    url: 'http://www.mongodb.org.cn',
		    tags: ['mongodb','database','Nosql'],
		    likes: 100	
	})
   ```
   
如果数据库中不存在集合xfan，mongodb会自动创建集合并插入文档。

我们也可以将数据定义为一个变量，然后再进行插入：

   ```
	document = ({
	    title: 'MongoDB命令',
	    by: 'xfan',
	    url: 'http://www.mongodb.org.cn',
	    tags: ['mongodb','database','Nosql'],
	    likes: 100	
	})
   ```

   ```
	db.col.insert(document)
   ```
   
- 查看已插入的文档

   ```
   db.col.find()
   ```
   
- 更新文档

   ```
	   db.collection.update(    
            <query>, 
            <update>, 
            {       
                upsert: <boolean>,   
                multi: <boolean>,  
                writeConcern: <document>
            }
	    )
   ```
   
参数说明：   

- query: update的查询条件，类似 sql update查询内where后面的。
- update： updated的对象和一些更新的操作符等，也可以理解为sql update查询内set后面的。
- upsert： 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew
- multi： 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- writeConcern： 可选，抛出异常的级别。

如：我们在集合col中插入如下数据：   

   ```
	db.col.insert({  
            title: 'xfan blog',  
            description: 'xfan blog have a nice party',   
            by: 'xfan',   
            url: 'http://www.mongodb.org.cn',   
            tags: ['mongodb', 'database', 'NoSQL'],  
            likes: 100  
	})
   ```

   ```
	db.col.update(
            {'title':'xfan blog'},
            {$set:{'title':'xfan'}}
	)
   ```

   ```
	db.col.find().pretty()
	结果：{

            "_id" : ObjectId("56064f89ade2f21f36b03136"),  
            "title" : "xfan",   
            "description" : "xfan blog have a nice party",  
            "by" : "xfan",      
            "url" : "http://www.mongodb.org.cn", 
            "tags" : [   
                "mongodb",   
                "database",   
                "NoSQL"      
            ],      
            "likes" : 100  
	}
   ```
发现该文档已经更新，但是以上语句只会修改第一条发现的文档，如果需要修改多条相同的文档，则需要设置multi参数为true。   
如：   

   ```
	db.col.update({
	    'title':'xfan blog'},
	    {$set:{'title':'xfan'}}，
	    {multi:true}
	)
   ```
   
- save()更新文档

我们也可以使用save()方法替换已有文档，来达到更新文档的目的。

   ```
	db.collection.save(    
	    <document>,     
	    {      
		      writeConcern: <document> 
	    }  
	)  
   ```
   
参数说明：

- document: 文档数据。
- writeConcern: 可选，抛出异常的级别。

执行此命令，保存一个_id与需要更新的文档相同的文档，即可替换需要更新的文档达到目的。

- 删除文档

   ```
	db.collection.remove(     
            <query>,     
            {       
                justOne: <boolean>,
                writeConcern: <document> 
            } 
	    )
   ```
   
参数说明：

- query: 可选，删除文档的条件。
- justOne: 可选，如果设置为true或者1，则只删除一个文档。
- writeConcern: 可选，抛出异常的级别。

移除 title 为 'xfan blog' 的文档:

   ```
	db.col.remove({'title':'MongoDB 教程'})
   ```
   
只移除第一条找到的记录

   ```
	db.COLLECTION_NAME.remove(DELETION_CRITERIA,1)
   ```
   
删除集合中的所有数据

   ```
	db.col.remove({})  
   ```
      
#### MongoDB 操作符
- 大于(>) ---- $gt  
- 小于(<) ---- $lt
- 大于等于(>=) ---- $gte
- 小于等于(<=) ---- $lte

如：db.xfan.find({"likes":{$gt:100}})

#### MongoDB $type 操作符
每种类型对应的数字如下：  
 
| 类型            | 数字               | 备注  |
| --------------- |:------------------:| -----:|
| Double          | 1                  |  |
| String          | 2                  |  |
| Object          | 3                  |  |
| Array           | 4                  |  |
| Binary data     | 5                  |  |
| Undefined       | 6                  |已废弃  |
| Object id       | 7                  |  |
| Boolean         | 8                  |  |
| Date            | 9                  |  |
| Null            | 10                 |  |
| Regular Expression | 11              |  |
| JavaScript      | 13                 |  |
| Symbol          | 14                 |  |
| JavaScript (with scope) | 15         |  |
| 32-bit integer  | 16                 |  |
| Timestamp       | 17                 |  |
| 64-bit integer  | 18                 |  |
| Min key         | 255                | Query with -1|
| Max key         | 127                |              |

如果想获取'xfan'集合中title为String的数据，可以使用
   ```
    db.xfan.find({"title" : {$type : 2}}) 
   ```

#### MongoDB 的 Limit 与 Skip 方法
查询集合中的两条记录: 

   ```
    db.col.find({},{"title":1,_id:0}).limit(2)
   ```
   
如果没有指定limit()方法中的参数则显示集合中的所有数据。

使用 skip() 方法只显示第二条文档数据: 

   ```
    db.col.find({},{"title":1,_id:0}).limit(1).skip(1)
   ```
   
skip() 方法默认参数为0。

#### MongoDB 的排序
在MongoDB中使用使用sort()方法对数据进行排序，sort()方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。   
语法如下:

- 升序

   ```
    db.COLLECTION_NAME.find().sort({KEY:1})
   ```
   
- 降序

   ```
    db.COLLECTION_NAME.find().sort({KEY:-1})
   ```