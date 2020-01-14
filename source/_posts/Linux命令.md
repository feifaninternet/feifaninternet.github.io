------------------
title: Linux命令
tags: Linux
categories: Expand
date: 2018-03-29 18:20:31
------------------

### top 命令
监控系统的运行状态，并且可以按照cpu，内存，执行时间进行排序。

### pgrep/pkill 命令
根据名称或者其它属性查询（发送信号）进程信息。

1.pgrep命令根据提供的条件查询进程的pid，查询条件是and方式的，对于同一个选项，使用","分隔可以按照or方式查询。

查询进程名为sshd，并且属主是root的进程 

   ```
    pgrep -u root sshd
   ```

查询属主是root或者daemon的进程

   ```
    pgrep -u root,daemon
   ```
   
2.pkill 使用与pgrep类似，不过它不是用来查询进程pid，而是给进程发送信号，默认会发送 SIGTERM信号。

   ```
    pgrep -u root named # 查找named进程的pid
    pkill -HUP syslogd  # 告诉syslogd重新读取配置文件
   ```
   
查看哪些信息可用可以使用<span> kill -l </span>列出所有的信号及数值

### expect 命令
- send 发送一个字符串给进程
- expect 等待来自进程返回的字符串
- spawn 开始一个命令

实现控制台 ssh 直接登录Linux服务器

   ```
    !/usr/bin/expect
    
    set timeout 20
    
    set ip "IP地址"
    set user "用户名"
    set password "密码"
    
    spawn ssh "$user\@$ip"
    
    expect "$user@$ip's password:"
    send "$password\r"
    
    interact
   ```

### pstack 命令
pstack是一个shell脚本，用于打印正在运行的进程的栈跟踪信息，它实际上是gstack的一个链接。   
该命令只需要提供一个参数，进程的pid即可。

   ```
    $ sudo pstack $(pgrep -uroot php-fpm)
   ```
   
pstack是gdb的一部分，如果系统没有pstack命令，使用yum搜索安装gdb即可。

### strace 命令
strace命令用于跟踪系统调用和信息。主要用于诊断，调试程序，使用该命令能够打印出进程执行的系统调用信息。   

#### 找出应用程序启动时读取的配置文件

   ```
    $ strace php 2>&1 | grep php.ini
   ```
   
这里的2>&1是将标准错误输出重定向到标准输出

#### 查找为什么程序没有打开指定文件

   ```
    $ strace -e open,access 2>&1 |grep your-filename
   ```

-e参数指定了一个限定表达式用于指定要跟踪的时间和如何跟踪他们。

查看进程正在执行什么操作

   ```
    root@dev:~# strace -c -p 11084
   ```

#### 查看为什么xxx无法连接到服务器

   ```
    $ strace -e poll,select,connect,recvfrom,sendto nc www.news.com 80
   ```
   
### nc 命令
该命令用于创建任意的tcp/udp连接或者是监听连接

#### 建立一个基本的C/S模型(文件远程复制)
在server1上使用nc命令创建一个服务端 :

   ```
    server1 $ nc -l 1234
   ```
   
在server2上，使用nc作为客户端连接到server1
 
   ```
    server2 $ nc server1的IP地址 1234
   ```
   
上面的例子可以改造实现文件远程发送

   ```
    server1 $ nc -l 1234 > filename.out
   ```
   
   ```
    server2 $ nc server1的IP地址 1234 < filename.in
   ```

-l指定了nc应该作为server端监听指定的端口

#### 模拟HTTP请求

   ```
    # echo -n "GET / HTTP/1.0\r\n\r\n" | nc php.net 80
    HTTP/1.1 400 Bad Request
    Server: nginx/1.6.2
    Date: Tue, 16 Dec 2014 08:09:35 GMT
    Content-Type: text/html
    Content-Length: 172
    Connection: close
    
    <html>
    <head><title>400 Bad Request</title></head>
    <body bgcolor="white">
    <center><h1>400 Bad Request</h1></center>
    <hr><center>nginx/1.6.2</center>
    </body>
    </html>
   ```

### 端口扫描
端口扫描的作用还是比较大的，使用nc可以方便的进行端口扫描

   ```
    # nc -z letv.com 1-100
   ```

这里的1-100指定了扫描的端口范围，-z参数告诉nc命令只报告开放的端口。   
默认nc命令发送的是tcp请求，通过指定参数-u可以发送udp请求。

#### 目录传输
将server2的phpredis-master目录拷贝到server1。

server1 :

   ```
    # nc -l 1234|tar zxvf -
   ```

server2 :

   ```
    # tar zcvf - phpredis-master|nc server1的IP地址 1234
   ```

### pstree命令
该命令用于显示进程树，以树的形式显示在运行的进程，树的根节点是指定的pid

   ```
    [root@cdn ~]# pstree -p $(pgrep -uroot php-fpm)
   ```

### ss 命令
ss命令用于显示socket的统计信息。

#### 显示socket的汇总信息
-s选项用于显示汇总信息

   ```
    # ss -s
   ```
   
#### 查看所有打开的网络端口
-l选项用于列出当前正在监听的socket

   ```
    # ss -l
   ```
   
使用ss -pl可以查看使用网络端口的进程名称，这里的-p选项用于显示进程信息

   ```
    # ss -pl
   ```

#### 显示所有的TCP/UDP Socket
参数-a用于显示所有的socket，-t指定是TCP，-u是UDP，-w是RAW，-x是UNIX。

   ```
    # ss -t -a
    # ss -u -a
    # ss -w -a
    # ss -x -a
   ```
   
### w/who 命令
w命令用于查看当前哪些用户登录到系统和他们正在做什么，who命令仅用于查看哪些用户登录系统

   ```
    # w
   ```
 
### iostat 命令 

   ```
    # iostat
   ```
 
通过指定 -d 参数可以设定自动按照指定时间间隔显示统计信息。   
例如，下列命令每隔2s显示一次。

   ```
    # iostat -d 2
   ```
 
### iptraf 命令
实时网络统计，交互式的IP网络实时监控工具，图形化界面，比较方便

   ```
    # iptraf
   ```
   
#### 查看 Linux 的版本
在RedHat和Cent OS下，查看当前系统的版本

   ```
    $ cat /etc/centos-release
   ```
   
### time 命令
统计程序执行时间，这些时间包括程序从被调用到终止的时间，用户CPU时间，系统CPU时间。

   ```
    $ time ls
   ```

### tee 命令
用于将标准输入拷贝到标准输出

   ```
    $ echo "hello,world"|tee -a test.txt
   ```
   
上述命令将hello，world字符串输出到test.txt文件中，"-a"默认情况下，tee命令会使用">"覆盖输出到文件，使用"-a"属性，会使用">>"追加方式

### netstat 命令
查看端口占用情况

   ```
    # netstat -apn
   ```
   
- -a：显示所有的socket信息
- -p：显示每个socket所属于的进程名称和pid
- -n：显示数字形式的地址而不是符号化的主机名/端口或者用户名

### perf 命令
perf命令是随Linux内核代码一同发布和维护的性能诊断工具，它还可以用于内核代码的性能统计和分析   
没有该命令的话，可以使用yum进行安装

   ```
    # yum install perf
   ```
   
#### perf stat
perf stat通过概括精简的方式提供被调试程序运行的整体情况和汇总数据。

   ```
    $ perf stat ./test
   ```

#### perf top
用于实时显示当前系统的性能统计信息。该命令主要用来观察整个系统当前的状态，比如可以通过查看该命令的输出来查看当前系统最耗时的内核函数或某个用户进程。   
执行该命令需要root权限。   

   ```
    $ sudo perf top
   ```
   
### lsof 命令
工具lsof是一个可以列出操作系统打开的文件的工具，常用参数：

- lsof filename 显示打开指定文件的所有进程
- lsof -a 表示两个参数都必须满足时才显示结果
- lsof -c string 显示COMMAND列中包含指定字符的进程所有打开的文件
- lsof -u username 显示所属user进程打开的文件
- lsof -g gid 显示归属gid的进程情况
- lsof +d /DIR/ 显示目录下被进程打开的文件
- lsof +D /DIR/ 同上，但是会搜索目录下的所有目录，时间相对较长
- lsof -d FD 显示指定文件描述符的进程
- lsof -n 不将IP转换为hostname，缺省是不加上-n参数
- lsof -i 用以显示符合条件的进程情况
- lsof -p PID 选择指定PID
- lsof -i[46] protocol[:service|port]

   ```
    46: IPv4 or IPv6
    protocol: TCP or UDP
    hostname: Internet host name
    hostaddr: IPv4地址
    service: /etc/service中的 service name (可以不只一个)
    port: 端口号 (可以不只一个)
   ```
   
### unzip 命令
unzip命令用于解压.zip文件，常用参数：

- -f：只更新磁盘上已经存在的文件
- -u：更新磁盘上存在的文件，文件不存在则创建
- -o：如果文件已经存在则直接覆盖，不提示
- d：指定解压到的目录

解压test.zip到/var/www目录，部署web站点

   ```
   # unzip -u -o -d /var/www test.zip
   # chown -R www:www /var/www
   ```
   
### 使用pushd和popd命令快速切换目录

   ```
   $ pwd
   /Users/mylxsw/codes/php/lecloud/api
   $ pushd .
   ~/codes/php/lecloud/api ~/codes/php/lecloud/api
   $ cd ../album/
   $ pwd
   /Users/mylxsw/codes/php/lecloud/album
   $ popd
   ~/codes/php/lecloud/api
   $ pwd
   /Users/mylxsw/codes/php/lecloud/api
   ```
   
### SCP 命令
服务器和本地计算机之间传递文件。   
从服务器下载文件

   ```
   scp username@服务器地址:/path/文件名 本地保存路径
   ```
   
上传文件到服务器

   ```
   scp 本地文件路径 username@服务器地址:/保存到服务器的路径
   ```
   





   

   









     

   
   

