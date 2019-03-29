-------------------------
title: 如何应对恶意IP访问
tags: http
categories: Expand
-------------------------
date: 2019-03-13 16:20:25

### Description
最近上线的一个项目，上线几天突然崩溃了，客户反馈的时候我也是纳闷，啥也没有改动呀，上服务器一看访问日志，很明显是有人在恶意攻击，项目运行内存不足挂掉了。

![请求日志](/picture/reqLog.jpg)

遇到这种情况，我们应该如何应对呢？

### 限制单个IP访问接口的频率
在拦截器中使用 `Guava Cache` 实现

   ```java
   //初始化 cache ，定义过期时间
    public final static Cache<Object, Object> IP_CONTROL = CacheBuilder.newBuilder()
                .expireAfterAccess(1, TimeUnit.SECONDS).build();
    
    //拦截在限制时间内的单ip访问相同接口
    if (IP_CONTROL.getIfPresent(ip + url) != null){
        logger.warn("拦截恶意请求, ip:{}, url:{}", ip, url);
        return false;
    }
    //通过的ip放入缓存
    IP_CONTROL.put(ip + url, "pass");
   ```

这样在同一秒内同一个IP对相同的接口发起的请求将被拦截
缺陷: 
1. 限制了所有IP访问接口，会误伤非恶意IP
2. 请求还是会进入应用程序，不够完美


### 使用RateLimiter限流
可在被攻击的控制层进行处理，[令牌桶算法和RateLimiter原理](https://www.jianshu.com/p/5d4fe4b2a726)

   ```java
    @RestController  
    public class IndexController {
        //设置放行速率
        RateLimiter rateLimiter = RateLimiter.create(1);  
      
        @RequestMapping("/test")  
        public Object test(int count, String code) {  
            System.out.println("等待时间" + rateLimiter.acquire());
            return "pass";  
        }  
    }
   ```
   
可宏观控制系统的稳定性，控制请求放行的速率达到稳定系统的效果
缺陷: 
1. 尽管控制了放行速率，但是非法IP还是会有源源不断的进入，而且还会占用正常用户的排队时间
2. 请求还是会进入应用程序，不够完美

### Nginx 限制单IP并发访问数量和访问速度

1.在 `nginx` 相应的配置文件中加入下面的配置，然后重启 `nginx`

   ```
    http {
        #定义一个名为allips的limit_req_zone用来存储session，大小是10M内存，
        #以$binary_remote_addr 为key,限制平均每秒的请求为20个，
        #1M能存储16000个状态，rete的值必须为整数，
        #如果限制两秒钟一个请求，可以设置成30r/m
        limit_req_zone $binary_remote_addr zone=allips:10m rate=20r/s;
        #连接数限制
        limit_conn_zone   one  $binary_remote_addr  10m;
        ...
        server {
            ...
            location /***/ {
                #限制每ip每秒不超过20个请求，漏桶数burst为5
                #brust的意思就是，如果第1-4秒请求为19个，第5秒的请求为25个是被允许的，可以不设置
                #但是如果你第1秒就25个请求，第2秒超过20的请求返回503错误。
                #nodelay，如果不设置该选项，严格使用平均速率限制请求数
                #如果不设置第1秒25个请求时，5个请求放到第2秒执行，
                #设置nodelay，25个请求将在第1秒执行。
                #我处理时只设置了 limit_req zone=allips
                limit_req zone=allips burst=5 nodelay;
                
                #连接数限制，我处理时未设置此参数
                limit_conn one 20;          
                #带宽限制,对单个连接限数，如果一个ip两个连接，就是500x2k，我处理时未设置此参数
                limit_rate 500k; 
            }
    }
   ```
   
控制之后查看 `nginx` 的请求日志，发现许多请求已被拦截，返回 `503`

![nginx日志](/picture/attackLog.png)

请求未到应用层，系统压力大大降低，恶意请求绝大部分都被拦截
缺陷: 
1. 恶意IP还是会有少量的请求到达应用层，致使恶意请求的次数被控制
2. 未禁止恶意IP访问，不够完美

### 使用Shell脚本限制恶意IP
我们可以使用 `Shell` 脚本，分隔日志，进行分析，找出其中的恶意IP，再将恶意IP加入 `nginx` 的配置中进行拦截

1.编写 `Shell` 脚本 `vim /shell/nginx_cutaccesslog.sh`

   ```
    #!/bin/bash
    #日志目录
    log_path=/var/log/nginx
    date=`date -d "10 min ago" +%Y%m%d-%H:%M:%S`
    cd ${log_path}
    #过滤appvalley-access_log中正常访问API接口并在10分钟
    #内访问量最高的100个IP，取值如果此IP访问量大于100次，则把此IP放入黑名单
    #awk '{ print $1}'：取数据的低1域（第1列），第一列为访问次数，第二列为ip
    #awk '{if($1>100) print "deny "$2";"}' 第一列访问次数大于100时，将第二列的ip记录
    #sort：对IP部分进行排序。
    #uniq -c：打印每一重复行出现的次数。（并去掉重复行）
    #sort -nr -k1：按照重复行出现的次序倒序排列,-k1以第一列为标准排序。
    #head -n 100：取排在前100位的IP 。
    cat appvalley-access_log | grep -v 403 | awk '{print $1}'|sort|uniq -c | sort -nr -k1 
    | head -n 100 | awk '{if($1>100) print "deny "$2";"}' > /etc/nginx/website/denyip.conf
    #将appvalley-access_log中的日志移动到accesslog.bak/中，accesslog.bak/是切割的日志
    mv ${log_path}/appvalley-access_log ${log_path}/accesslog.bak/access_${date}.log
    # 重载nginx
    cd /sbin/
    nginx -s reload
    
   ```

2.修改 `nginx.conf` 使其包含 `denyip.conf`，重载 `nginx`，因为之前配置项目路径时已经加入了 `include /etc/nginx/website/*.conf;`，所以就不用再设置了

3.运行 `Shell` 脚本进行验证，进入 `accesslog.bak/` 目录中，查看日志分隔是否成功

![分隔日志](/picture/bakLog.jpg)

4.查看 `denyip.conf` 文件，发现已经写入了拦截的恶意IP

![拦截IP](/picture/interceptor.jpg)

5.再查看 `nginx` 访问日志是否成功拦截请求

![nginx请求日志](/picture/403.jpg)

6.脚本验证成功后我们可以添加定时任务，每10分钟执行一次脚本，去禁用恶意IP

   ```
    */10 * * * * /bin/bash /shell/nginx_cutaccesslog.sh > /dev/null 2>&1
   ```

*剩余疑问：如何获取真实IP进行拦截？*