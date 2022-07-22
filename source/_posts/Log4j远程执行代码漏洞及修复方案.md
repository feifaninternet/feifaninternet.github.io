-------------------------
title: Log4j远程执行代码漏洞及修复方案
tags: JAVA
categories: Expand
date: 2021-12-20 19:42:41
-------------------------


Apache Log4j 2 是一款开源的日志记录工具，被广泛应用于各类框架中。近期，Apache Log4j 2 被爆出存在漏洞，
漏洞现已公开，本文为 KubeSphere 用户提供建议的修复方案。

此次漏洞是由于 Log4j 2 提供的 lookup 功能造成的，该功能允许开发者通过一些协议去读取相应环境中的配置。
但在实现的过程中，并未对输入进行严格的判断，从而造成漏洞的发生。由于大量的软件都使用了 Log4j 2 插件，所以大量的 
Java 类产品均被波及。

受影响的 Log4j 版本为 Apache Log4j 2.x < 2.15.0-rc2。目前官方发布了 Apache 2.15.0-rc2 
版本对该漏洞进行了修复，但是该版本并非正式发行版，故存在不稳定的因素，如要升级建议对相关数据进行备份。

同时，也提供了三种方法对漏洞进行补救，为

- 将系统环境变量 FORMAT_MESSAGES_PATTERN_DISABLE_LOOKUPS 设置为 true
- 修改配置 log4j2.formatMsgNoLookups=True
- 修改 JVM 参数 -Dlog4j2.formatMsgNoLookups=true

以下三种解决方法，可以任选其中一种进行参考。

方法一：修改系统环境变量
由于 KubeSphere 默认使用了 ElasticSearch 收集日志，所以也应该在 KubeSphere 修改相应的配置来对漏洞进行修复。以下说明如何在 KubeSphere 中进行相应的操作对 ElasticSearch 进行修复。

将系统环境变量 FORMAT_MESSAGES_PATTERN_DISABLE_LOOKUPS 设置为 True，为此，我们需要修改 ElasticSearch 的 Yaml 文件，因为它是一个 StatefulSet 文件，所以需要进行如下修改：

    kubectl edit  statefulset  elasticsearch-logging-data -n kubesphere-logging-system
    kubectl edit  statefulset  elasticsearch-logging-discovery  -n kubesphere-logging-system
    
在这两个 Yaml 文件中插入环境变量设置：

    env:
    - name: FORMAT_MESSAGES_PATTERN_DISABLE_LOOKUPS
      value: "true"
      
方法二：修改 Log4j 2 配置
另外，您也可以修改配置 log4j2.formatMsgNoLookups=True，您可以执行如下命令：

    kubectl edit configmaps elasticsearch-logging  -n kubesphere-logging-system
    
然后插入上面所提到的配置：

    log4j2.properties: |-
        status=error
        appender.console.type=Console
        appender.console.name=console
        appender.console.layout.type=PatternLayout
        appender.console.layout.pattern=[%d{ISO8601}][%-5p][%-25c{1.}] %marker%m%n
        rootLogger.level=info
        rootLogger.appenderRef.console.ref=console
        logger.searchguard.name=com.floragunn
        logger.searchguard.level=info
        # 插入此行
        log4j2.formatMsgNoLookups=true
        
注意：

修改后请注意相关配置是否挂载进去，如果没有挂载进去，请重启 Pod。
如果您将 KubeSphere Logging 组件重新安装，ks-installer 可能会导致该 ConfigMap 的配置被重置，需要再参考方法二手动配置一遍，或者采取方法一，设置系统环境变量 FORMAT_MESSAGES_PATTERN_DISABLE_LOOKUPS 为 true。

方法三：修改 ElasticSearch 的 JVM 参数
除了上述两种方法，您还可以选择在 KubeSphere 集群中的 ElasticSearch 添加配置文件，单独配置 JVM 参数，详见 [ElasticSearch 公告声明](https://discuss.elastic.co/t/apache-log4j2-remote-code-execution-rce-vulnerability-cve-2021-44228-esa-2021-31/291476).