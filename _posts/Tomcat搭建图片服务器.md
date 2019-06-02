---
title: Tomcat搭建图片服务器
date: 2019-04-24 17:20:16
tags: Java
keywords:
description:
---



配置tomcat的虚拟映射路径   1、修改Tomcat的conf/server.xml文件

```

<Host name="localhost"  appBase="webapps"  unpackWARs="true" autoDeploy="true">

        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"  prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />          
               <!-- 设置图片虚拟路径[访问时路径为/photo] -->  
         <Context path="/photo" docBase="D:\upFiles" reloadable="true" />  

<!-- 也可以这样设置图片虚拟路径 -->  
<Host name="10.0.0.123" appBase="webapps"   unpackWARs="true" autoDeploy="true"  xmlValidation="false" xmlNamespaceAware="false"> 
    <Context path="" docBase="F:\temp" reloadable="false" ></Context> 
 </Host>  
```
其中path是映射的虚拟路径（可视具体情况配置），docBase是静态资源存放的真实物理路径，reloadable指有文件更新时，是否重新加载，一般设置为true后，tomcat不需要重启启动，自动热加载！

[参考资源](https://www.cnblogs.com/magic101/p/7756402.html)

