-------------------------
title: JQuery zTree树插件
tags: zTree
categories: Expand
date: 2018-05-24 14:28:52
-------------------------

### Description
zTree是一个依靠JQuery实现的多功能树插件。优异的性能，灵活的配置，多功能的组合是zTree的优点。

- 特点：

1. zTree 3.x 将核心代码按照功能进行分割，不需要的代码可以不用加载
2. 采用延迟加载技术，上万节点轻松加载
3. 兼容IE,Chrome,Safari等浏览器
4. 支持静态和Ajax异步加载节点数据
5. 支持任意更换皮肤，自定义图标
6. 支持及其灵活的checkbox或radio选择功能
7. 提供多种事件响应回调
8. 灵活的编辑功能，可随意拖拽节点，还可以多节点拖拽
9. 在一个页面内同时生成多个Tree实例   
10. 简单的参数配置实现灵活多变的功能

[zTree官方文档](http://www.treejs.cn/v3/api.php)

### zTree 的使用
- 引入资源：向页面中引入JQuery的js文件，zTree的js文件和CSS文件
- 页面添加zTree的容器，class属性为ztree

   ```html
    <ul id="treeDemo" class="ztree"></ul>
   ```
- 引入数据：zTree可以解析json格式的数据，有两种方式可以把数据传给zTree组件来生成树状结构。

1.直接把json数据传给组件

   ```javascript
    $(document).ready(function(){
        $.fn.zTree.init($("#treeDemo"), setting, zNodes);
    });
   ```
   
2.异步获取json格式数据，第三个参数传null或者空着

   ```javascript
    $(document).ready(function(){
        $.fn.zTree.init($("#treeDemo"), setting, null);
    });
   ```
   
- 使用 Demo

   ```javascript
    var setting = {
        data: {
            key : {
                title : "c01name", //鼠标悬停显示的信息
                name : "c01name" //网页上显示出节点的名称
            },
            simpleData: {
                enable: true,
                idKey: "c01id", //修改默认的ID为自己的ID
                pIdKey: "c01parentid",//修改默认父级ID为自己数据的父级ID
                rootPId: 000     //根节点的ID
            }
        }
    };
   ```
   
有时候异步处理得到的数据并不是一个单纯的jsonArray数据，我们需要对它进行一个简单的提取操作：

   ```javascript
    var setting = {
        async: {
            enable: true,//采用异步加载
            dataFilter: ajaxDataFilter,    //预处理数据
            url : "http://127.0.0.1/WeChat/admin/C01Action_listC01.action",
            dataType : "json"
        },
        data : {
            key : {
                title : "c01name",    
                name : "c01name"
            },
            simpleData : {
                enable : true,
                idKey : "c01id",
                pIdKey : "c01parentid",
                rootPid : 000 
            }
        },
        callback : {
            beforeClick: zTreeBeforeClick,
            onClick : zTreeOnClick,
            onAsyncSuccess: zTreeOnAsyncSuccess //异步加载完成调用
        }
    };
    /* 获取返回的数据，进行预操作 */
    function ajaxDataFilter(treeId, parentNode, responseData) {
        responseData = responseData.jsonArray;
        return responseData;
    };
    //异步加载完成时运行，此方法将所有的节点打开
    function zTreeOnAsyncSuccess(event, treeId, msg) {
        var treeObj = $.fn.zTree.getZTreeObj("treeDemo");
        treeObj.expandAll(true);
    }
   ```
   
- 使用案例：

   ```html
    <!DOCTYPE html>
    <HTML>
        <HEAD>
        <TITLE> zTree Demo </TITLE>
        <meta http-equiv="content-type" content="text/html; charset=UTF-8">
        <link rel="stylesheet" href="zTreeStyle/zTreeStyle.css" type="text/css">
        <script type="text/javascript" src="jquery-1.4.2.js"></script>
        <script type="text/javascript" src="jquery.ztree.core-3.x.js"></script>
        <SCRIPT LANGUAGE="JavaScript">
        var zTreeObj;
        var setting = {}; // zTree 的参数配置，后面详解
        var zNodes = [ // zTree 的数据属性，此处使用标准json格式
        {
        name: "test1", open: true, children: [
        { name: "test1_1" }, { name: "test1_2" }]
        },
        {
        name: "test2", open: true, children: [
        { name: "test2_1" }, { name: "test2_2" }]
        } ];
        $(document).ready(function () {
        zTreeObj = $.fn.zTree.init($("#treeDemo"), setting, zNodes); //初始化zTree，三个参数一次分别是容器(zTree 的容器 className 别忘了设置为 "ztree")、参数配置、数据源
        });
        </SCRIPT>
        </HEAD>
        <BODY>
        <div>
        <ul id="treeDemo" class="ztree"></ul>
        </div>
        </BODY>
    </HTML>
   ```
   
效果图如下：

![菜单示例图](/picture/zTree.png)

详细应用请参考[zTree官方文档](http://www.treejs.cn/v3/api.php)

