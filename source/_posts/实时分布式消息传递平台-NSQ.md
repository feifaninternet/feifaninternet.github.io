---------------------------------
title: 实时分布式消息传递平台 -- NSQ
date: 2019-04-15 14:24:55
tags: NSQ
categories: Expand
---------------------------------

### Description
网上关于 `NSQ` 在 `java` 平台使用的资料非常少，我们来看下 `NSQ` 到底与其他消息中间件有何不同，到底应该如何使用？

### NSQ简介
`NSQ` 是一个基于 `go` 语言开发的分布式实时消息架构。`NSQ` 主要用于处理大规模消息任务，每天可以处理的任务可达十亿级别。

#### 特点
 - 分布式，去中心化拓扑
 - 可伸缩，横向扩展
 - 操作友好，简单的配置&部署


#### 主要模块
1. **[nsqlookupd](https://nsq.io/components/nsqlookupd.html)** : 管理拓扑信息的守护程序。客户端查询`nsqlookupd`以发现`nsqd`特定主题的生产者，并且`nsqd`节点广播主题和信道信息。
2. **[nsqd](https://nsq.io/components/nsqd.html)** : 守护进程，它接收，排队并向客户端发送消息，可独立运行。
3. **[nsqadmin](https://nsq.io/components/nsqadmin.html)** : `Web UI`，可实时查看聚合的群集统计信息并执行各种管理任务。
4. **[utilities](https://nsq.io/components/utilities.html)** : 常见基础功能，数据流处理 `nsq_stat` `nsq_to_file`等

<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="https://upload-images.jianshu.io/upload_images/14001673-adcf8b01a5004396.gif" alt="发布与消费" title="发布与消费">
</div>

#### 主要角色

##### 生产者 Producer
连接`nsqd`发布消息到`topic`

`Producer`断线后需要手动重连，`Consumer`断线后会自动重连，`Consumer`的重连时间默认为60s，可设置短一点
`Producer`不能发布空`message`否则会导致`panic`
如果`Producer`连接的`nsqd`中断，那么`message`就会发布失败，所以我们也要考虑这种情况，做好备用方案。

##### 消费者 Consumer
消费者有两种方式与`nsqd`建立连接。
1. 消费者直连`nsqd`，这是最简单的方式，缺点是`nsqd`服务无法实现动态伸缩了
2. 消费者通过`http` 查询`nsqlookupd`获取该`nsqlookupd`上所有`nsqd`的连接地址，然后再分别和这些`nsqd`建立连接 (官方推荐的做法)，但是客户端会不停的向`nsqlookupd`查询最新的`nsqd`地址目录


### NSQ与Kafka比较
类别          |NSQ                                                                                   |Kafka
---          |---                                                                                   |---
存储          |默认是把消息放到内存中，只有当队列里消息的数量超过–mem-queue-size配置的限制时，才会对消息进行持久化 |写到磁盘中进行持久化，并通过顺序读写磁盘来保障性能。持久化能够让Kafka做更多的事情：消息的重新消费（重置offset）；让数据更加安全，不那么容易丢失。同时Kafka还通过partition的机制，对消息做了备份，进一步增强了消息的安全性
推拉模型       |使用推模型，推模型能够使得时延非常小，消息到了马上就能够推送给下游消费，但是下游消费能够无法控制，推送过快可能导致下游过载  |使用拉模型，拉模型能够让消费者自己掌握节奏，但是这样轮询会让整个消费的时延增加，不过消息队列本身对时延的要求不是很大，这一点影响不是很大
消息的顺序性   |不能把特性消息和消费者对应起来，所以无法实现消息的有序性 |因为消息在Partition中写入是有序的，同时一个Partition只能够被一个Consumer消费，这样就可能实现消息在Partition中的有序。自定义写入哪个Partition的规则能够让需要有序消费的相关消息都进入同一个Partition中被消费，这样达到”全局有序“
数据备份      |只把消息存储到一台机器中，不做任何备份，一旦机器奔溃，磁盘损坏，消息就永久丢失了                    |通过partition的机制，对消息做了备份，增强了消息的安全性
消息投递      |支持至少一次，也就是说，消息有可能被多次投递，消费者必须自己保证消息处理的幂等性                    |支持准确一次
吞吐          |一般                                                                                   |高
是否可回溯      |否                                                                                    |是
requeue和defer |消费失败requeue，延时消费defer                                                          |无
生态          |无                                                                                     |Hadoop


### NSQ的安装和环境变量配置
安装配置[Go](https://golang.google.cn/dl/)开发环境

1.下载[nsq](https://nsq.io/deployment/installing.html)   
2.解压

   ```
    tar -xvf /downloads/NSQ-1.1.0.darwin-amd64.go1.10.3.tar.gz
   ```
3.删除压缩包，解压后到文件移动到`/usr/local`

   ```
    rm -rf /downloads/NSQ-1.1.0.darwin-amd64.go1.10.3.tar.gz
    mv NSQ-1.1.0.darwin-amd64.go1.10.3 /usr/local/nsq
   ```
4.配置环境变量

   ```
    #打开配置文件
    vim /etc/profile
    
    #在最后面加上下面的配置
    export NSQ_HOME=/usr/local/nsq
    export PATH=$PATH:$NSQ_HOME/bin
    
    #使环境变量生效
    source /etc/profile
   ```
5.测试使用
1.启动`nsqlookupd`

   ```
    nsqlookupd
   ```
   
2.启动`nsqd`

   ```
    nsqd --lookupd-tcp-address=127.0.0.1:4160 -broadcast-address=127.0.0.1
   ```

3.启动`nsqadmin`

   ```
    nsqadmin --lookupd-http-address=127.0.0.1:4161
   ```
   
4.创建`topic`和`channel`，发布消息

   ```
    #创建topic
    curl -X POST http://127.0.0.1:4151/topic/create?topic=name
    #创建channel
        curl -X POST http://127.0.0.1:4151/channel/create?topic=name&channel=name
    #发布消息
    curl -d "<message>" http://127.0.0.1:4151/pub?topic=name
   ```

5.在浏览器输入`127.0.0.1:4171`查看消息与连接状态
<div class="wrap effect" style="box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	webkit-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	moz-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;
	o-box-shadow:0px 1px 4px rgba(0,0,0,0.3),0 0 40px rgba(0,0,0,0.1) inset;">
	<img src="/picture/nsq/nsqadmin.jpg" alt="nsq的WebUI" title="nsq的WebUI">
</div>

### JAVA + NSQ
关于`java`使用NSQ，我是采取了类似`kafka`在`springboot`中集成的方法来实现，你可以在[wbean的项目](https://github.com/wbean/spring-boot-starter-nsq)中找到它，但是他好像并没有成功将项目上传至`maven`库，并不能直接导入，你可以下载项目后打包成`jar`，然后导入项目使用，
我没有使用最常用的[JavaNSQClient](https://github.com/brainlag/JavaNSQClient)，因为我觉得[wbean的项目](https://github.com/wbean/spring-boot-starter-nsq)集成显然是很优秀的

#### 1.配置 application.properties

   ```
    #nsq
    nsq.host=127.0.0.1
    nsq.port=4161
    
    #nsqd
    nsqd.host=127.0.0.1
    nsqd.port=4151
   ```
   
#### 2.创建 nsqd 配置类
关于`nsq`的配置，集成的`jar`会自动加载

   ```java
    package com.teapot.nsq;
    
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.stereotype.Component;
    
    import javax.annotation.PostConstruct;
    
    /**
     * @author xfan
     * @date 2019-04-19
     * @desc Nsqd配置属性
     */
    @Component
    @ConfigurationProperties(prefix = "nsqd")
    public class NsqdProperties {
    
        private String host;
    
        private String port;
    
        private static NsqdProperties nsqdProperties;
    
        @PostConstruct
        public void init(){
            nsqdProperties = this;
        }
    
        public static String nsqdHost(){
            return nsqdProperties.host;
        }
    
        public static String nsqdPort(){
            return nsqdProperties.port;
        }
    
        public String getHost() {
            return host;
        }
    
        public void setHost(String host) {
            this.host = host;
        }
    
        public String getPort() {
            return port;
        }
    
        public void setPort(String port) {
            this.port = port;
        }
    }

   ```
   
#### 3.发布消息方法
这里我们使用官方推荐的`http`方式

   ```java
    package com.teapot.nsq;
    
    import com.alibaba.fastjson.JSON;
    import com.google.gson.Gson;
    import com.teapot.exception.BusinessException;
    import com.teapot.support.ApiPostUtil;
    import net.sf.json.JSONObject;
    import org.apache.http.client.methods.HttpPost;
    import org.springframework.stereotype.Component;
    
    /**
     * @author xfan
     * @date 2019-04-19
     * @desc NSQ发布消息到topic
     */
    @Component
    public class NsqPutSupport {
    
        private static Gson gson = new Gson();
        private final static String SUCCESS_STR = "OK\n";
    
        public static void publishMessage(String topicName, Object object){
            String url = "http://" + NsqdProperties.nsqdHost() + ":" + NsqdProperties.nsqdPort() + "/pub?topic=" + topicName;
            JSONObject msgObj = new JSONObject();
            msgObj.put("message", JSON.toJSONString(object));
            HttpPost post = new HttpPost(url);
            String res = ApiPostUtil.post(gson.toJson(msgObj), post);
            System.out.println(res);
            if (!res.equals(SUCCESS_STR)){
                throw new BusinessException(2999, "消息发送失败");
            }
        }
    }

   ```
   
#### 4.测试

   ```java
    public class Test{
        private final static String DEVICE_TOPIC = "xfan_test";
        public static void main(String[] args) {
            String message = "test";
            NsqPutSupport.publishMessage(DEVICE_TOPIC, message);
        }
    }
   ```
   
在另一个项目中配置`nsq`和`nsqd`,使用如下代码消费消息

   ```java
    @RestController
    public class TestController{
    
        /**
        * 指定topic和channel来消费消息
        */
        @NsqListener(topic = "xfan_test", channel = "xfan_local")
        public void test(NSQMessage nsqMessage){
            String message = new String(nsqMessage.getMessage());
            System.out.println("接收到消息：" + message);
        }
    }
   ```


### 总结
`nsq`在java的应用还是比较少，资料也比较少，性能的话其实也不比`kafka`强，我还是建议使用`kafka`，但是喜欢`go`的同学可以研究使用`nsq`
