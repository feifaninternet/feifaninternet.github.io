---------------------------
title: NOSQL数据库--SSDB
date: 2019-04-15 14:24:55
tags: NSQ
categories: Expand
---------------------------

### Description
SSDB 是一个 C/C++ 语言开发的高性能 NoSQL 数据库, 支持 KV, list, map(hash), zset(sorted set) 等数据结构， 用来替代或者与 Redis 配合存储十亿级别列表的数据

#### 为什么要学习SSDB？
SSDB是Redis的替代和增强方案，SSDB的QPS在某些时候略高于Redis，读写效率也很高，有必要了解学习一下

### SSDB的基本组件和工作原理


### 搭建SSDB高可用集群
我们先来了解一下SSDB的配置文件

    # ssdb-server config
     
    work_dir = ./var
    pidfile = ./var/ssdb.pid
     
    server:
            ip: 127.0.0.1
            port: 8888
            # bind to public ip
            #ip: 0.0.0.0
            # format: allow|deny: all|ip_prefix
            # multiple allows or denys is supported
            #deny: all
            #allow: 127.0.0.1
            #allow: 192.168
            # auth password must be at least 32 characters
            #auth: very-strong-password
            #readonly: yes
            # in ms, to log slowlog with WARN level
            #slowlog_timeout: 5
     
    replication:
            binlog: yes
            # Limit sync speed to *MB/s, -1: no limit
            sync_speed: -1
            slaveof:
                    # to identify a master even if it moved(ip, port changed)
                    # if set to empty or not defined, ip:port will be used.
                    id: node-1
                    # sync|mirror, default is sync
                    type: sync
                    host: 120.76.78.6
                    port: 8888
     
    logger:
    	level: info
    	output: log.txt
    	rotate:
    		size: 1000000000
     
    leveldb:
    	# in MB
    	cache_size: 500
    	# in KB
    	block_size: 32
    	# in MB
    	write_buffer_size: 64
    	# in MB
    	compaction_speed: 100
    	# yes|no
    	compression: yes


- `work_dir`: ssdb-server 的工作目录, 启动后, 会在这个目录下生成 data 和 meta 两个目录, 用来保存 LevelDB 的数据库文件. 这个目录是相对于 ssdb.conf 的相对路径, 也可以指定绝对路径.

- `server`: 

配置项       |说明    	|备注
---         |---    	|---
ip          |默认的配置文件监听 127.0.0.1 本地回路网络, 所以无法从其它机器上连接此 SSDB 服务器. 如果你希望从其它机器上连接 SSDB 服务器, 必须把 127.0.0.1 改为 0.0.0.0，但是出于安全考虑我们一般不会这样配置  
port        |端口    	|
format      |设置拦截和放行规则    	|
deny        |拦截访问    	|
allow       |允许访问    	|
auth        |访问密钥    	|
readonly    |只读，只读设置下，所有的写操作都会被拒绝    	|
slowlog_timeout    |脚本执行超过多长时间，就可以记录到日志文件    	|

- `replication`: 

配置项       |说明    	|备注
---         |---    	|---
binlog          |记录数据库更新的语句，类似mysql 
sync_speed        |同步速率 	|
slaveof      |指向的节点信息    	|可有多个
id        |节点标识id    	|
type       |类型，sync主从，mirror双主    	|主从：所有子节点指向master节点，双主：各自指向对方节点，多主：每个节点要指向其他n-1个节点
host        |指向的节点的域名    	|
port        |指向的节点的端口    	|

- `logger`: 

配置项       |说明    	|备注
---         |---    	|---
level       |日志等级     |debug, info, warn, error, fatal，一般我们设置为debug
output      |日志输出 	|输出到指定文件，如果要改为输出到控制台，则配置为 stdout
rotate      |指向的节点信息    	|可有多个
size        |日志拆分时的大小    	|日志会按 1000MB 大小进行切分, 切分后的文件名格式如: log.txt.20150723-230422.切分后的日志文件不会自动被清理, 你需要自己写 crontab 脚本来清理.

- `leveldb`: 

配置项       |说明    	|备注
---         |---    	|---
cache_size       |内存缓存大小, 单位 MB     |一般地, 这个数字越大, 性能越好, 你可设置为物理内存的一半. 如果你的机器内存较小, 那就把它改小, 最小值是 16.
block_size      |不用关心 	|官方说不用关心 = =
write_buffer_size      |写缓冲区大小, 单位 MB    	|如果你的机器内存小, 那就把它改小, 否则改大. 它应该在这个范围内: [4, 128];
compaction_speed   |压缩速率    	|一般情况下, 不用关心. 如果你的硬盘性能非常差, 同时, 你的数据几乎不变动, 也没有什么新数据写入, 可以把它改小(最好大于 50).                        
compression        |压缩硬盘上的数据    	|最好设置为 yes! 如果是 yes, 一般你能存储 10 倍硬盘空间的数据, 而且性能会更好.
                                     
未完待续                         
                             




