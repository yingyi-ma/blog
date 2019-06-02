---
title: Spring实现WebSocket功能案例
date: 2019-04-16 16:25:46
tags: Java
---



#### 前言
>&emsp;&emsp;它有很多名字WebSocket，WebSocket协议和WebSocket API。从首选的消息传递应用程序到流行的在线多人游戏，WebSocket在当今最常用的Web应用程序中是至关重要的。

#### 概述

&emsp;&emsp;websocket是Html5新增加特性之一，目的是浏览器与服务端建立全双工的通信方式，解决http请求-响应带来过多的资源消耗，同时对特殊场景应用提供了全新的实现方式，比如聊天、股票交易、游戏等对对实时性要求较高的行业领域。

![](/images/01/1878_1.gif)


#### HTTP 与 WebSocket 的区别
WebSocket 与 HTTPWebSocket 协议在2008年诞生，2011年成为国际标准。现在所有浏览器都已经支持了。WebSocket 的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话。HTTP 有 1.1 和 1.0 之说，也就是所谓的 keep-alive ，把多个 HTTP 请求合并为一个，但是 Websocket 其实是一个新协议，跟 HTTP 协议基本没有关系，只是为了兼容现有浏览器，所以在握手阶段使用了 HTTP 。    

下面一张图说明了 HTTP 与 WebSocket 的主要区别：


![](/images/01/1880_1.png)



WebSocket 的其他特点：建立在 TCP 协议之上，服务器端的实现比较容易。与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。数据格式比较轻量，性能开销小，通信高效。可以发送文本，也可以发送二进制数据。没有同源限制，客户端可以与任意服务器通信。协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。

首先，WebSocket 是一个持久化的协议，相对于 HTTP 这种非持久的协议来说。  
简单的举个例子吧，用目前应用比较广泛的 PHP 生命周期来解释。HTTP 的生命周期通过 Request 来界定，也就是一个 Request 一个 Response ，那么在 HTTP1.0 中，这次 HTTP 请求就结束了。在 HTTP1.1 中进行了改进，使得有一个 keep-alive，也就是说，在一个 HTTP 连接中，可以发送多个 Request，接收多个 Response。但是请记住 Request = Response， 在 HTTP 中永远是这样，也就是说一个 Request 只能有一个 Response。而且这个 Response 也是被动的，不能主动发起。你 BB 了这么多，跟 WebSocket 有什么关系呢？ 好吧，我正准备说 WebSocket 呢。首先 WebSocket 是基于 HTTP 协议的，或者说借用了 HTTP 协议来完成一部分握手。

#### Spring-websocket  实现代码
Servlet +Spring-websocket 实现简单《聊天室》功能

##### 一、POM文件

```
 <properties>
        <spring.version>4.3.10.RELEASE</spring.version>
    </properties>
    <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-websocket</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>${spring.version}</version>
            </dependency>
        <!-- 工具类-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.5</version>
        </dependency>
        <!-- servlet相关接口-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
    </dependencies>
```
##### 二、Spring配置
```
    <context:component-scan base-package="com.test.webservice" annotation-config="true"/>
    <!-- webSocket 配置-->
    <!-- webSocket 类名称-->
    <bean id="websocket" class="com.test.webservice.SpringWebSocketHandler"/>

    <!-- 业务处理类路径-->
    <websocket:handlers allowed-origins="*">
        <websocket:mapping path="/websocket" handler="websocket"/>
        <!-- 拦截器类名称配置-->
        <websocket:handshake-interceptors>
            <bean class="com.test.webservice.HandlerInterceptor"/>
        </websocket:handshake-interceptors>
    </websocket:handlers>
```
##### 三、web.xml文件配置
```
  <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <async-supported>true</async-supported>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <!--配置spring-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--servlet-->
    <servlet>
        <servlet-name>LoginServlet</servlet-name>
        <servlet-class>com.test.servlet.LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LoginServlet</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.html</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>login.html</welcome-file>
    </welcome-file-list>

```
##### 四、登录类

```
package com.test.servlet;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * 登录
 */
public class LoginServlet extends HttpServlet {


    private static Map<String, String> userInfoMap = new HashMap<>();

    static {
        userInfoMap.put("admin", "123456");
        userInfoMap.put("user1", "123456");
        userInfoMap.put("user2", "123456");

    }


    public void doGet(HttpServletRequest request,
                      HttpServletResponse response)
            throws ServletException, IOException {
        //设置编码表
        response.setContentType("text/html;charset=utf-8");
        //获取表单数据
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        if (login(username,password)) {
            response.getWriter().write("登录失败!");
            //结束方法后面不再执行
            return;
        }

        //设置Session 用户名称
        HttpSession httpSession = request.getSession();
        httpSession.setAttribute("SESSION_USERNAME",username);

        //登录成功时
        response.getWriter().write("登录成功,3秒后跳转");
        response.sendRedirect(request.getContextPath() + "/test.html");
    }

    /**
     * 登录
     */
    private boolean login(String username, String password) {
        if (userInfoMap.get(username) == null) {
            return true;
        }
        if (!password.equals(userInfoMap.get(username))) {
            return true;
        }
        return false;
    }


    public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}

```
##### 五、websocket
拦截器

```
package com.test.webservice;

import org.springframework.http.server.ServerHttpRequest;
import org.springframework.http.server.ServerHttpResponse;
import org.springframework.http.server.ServletServerHttpRequest;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor;

import javax.servlet.http.HttpSession;
import java.util.*;

/**
 * WebSocket拦截器
 */
public class HandlerInterceptor extends HttpSessionHandshakeInterceptor {

    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler,
                                   Map<String, Object> attributes) throws Exception {
        System.out.println("Before Handshake");
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            HttpSession session = servletRequest.getServletRequest().getSession(false);
            if (session != null) {
                //使用userName区分WebSocketHandler，以便定向发送消息
                String userName = (String) session.getAttribute("SESSION_USERNAME");
                if (userName==null) {
                    System.out.println("用户身份验证失败");
                    return false;
                }
                attributes.put("WEBSOCKET_USERNAME",userName);
            }
        }
        return super.beforeHandshake(request, response, wsHandler, attributes);

    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler,
                               Exception ex) {
        super.afterHandshake(request, response, wsHandler, ex);
    }
}

```
Handler处理

```
package com.test.webservice;

import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.io.IOException;
import java.util.*;

/**
 * 继承WebSocketHandler对象。
 */
public class SpringWebSocketHandler extends TextWebSocketHandler {

    private final static Map<String, WebSocketSession> users = new HashMap<>();

    /**
     * 连接成功时候，会触发页面上onopen方法
     */
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        System.out.println("connect to the websocket success......当前数量:" + users.size());
    }

    /**
     * js调用websocket.send时候，会调用该方法
     */
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        Map<String, Object> attrMap = session.getAttributes();
        String username = attrMap.get("WEBSOCKET_USERNAME").toString();
        users.put(username, session);
        System.out.println("user:-" + username + ":" + message.getPayload());
        TextMessage dataPushMsg = new TextMessage(username + ":" + message.getPayload());
        sendMessageToUsers(dataPushMsg);
    }

    /**
     * 关闭连接时触发
     */
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
        System.out.println("websocket connection closed......");
        String username = (String) session.getAttributes().get("WEBSOCKET_USERNAME");
        System.out.println("用户" + username + "已退出！");
        users.remove(session);
        System.out.println("剩余在线用户" + users.size());
    }

    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        if (session.isOpen()) {
            session.close();
        }
        System.out.println("websocket connection closed......");
        users.remove(session);
    }

    public boolean supportsPartialMessages() {
        return false;
    }


    /**
     * 给所有在线用户发送消息
     *
     * @param message
     */
    private void sendMessageToUsers(TextMessage message) {
        for (Map.Entry entry : users.entrySet()) {
            {
                WebSocketSession user = (WebSocketSession) entry.getValue();
                try {
                    if (user.isOpen()) {
                        user.sendMessage(message);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}

```

##### 六、登录、聊天室页面
登录

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
</head>
<body>
<fieldset>
    <legend>用户登录</legend>
    <br/>
    <!-- form 表单的action 属性值要和配置在web.xml文件中的servlet的url-pattern相同 -->
    <form action="login" method="post" name="login">
        用户名：<input type="text" name="username"/> <br/> <br/>
        密&nbsp;码：<input type="password" name="password"/> <br/> <br/>
        <input type="submit" value="登录"/>
    </form>
</fieldset>
</body>
</html>
```

聊天室

```
<!DOCTYPE html>  
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>聊天室</title>
</head>
<body>
<script type="text/javascript" src="http://cdn.bootcss.com/jquery/3.1.0/jquery.min.js"></script>
<script type="text/javascript" src="http://cdn.bootcss.com/sockjs-client/1.1.1/sockjs.js"></script>
<script type="text/javascript">
    var websocket = null;
    if ('WebSocket' in window) {
        websocket = new WebSocket("ws://localhost:8080/websocket");
    } 
    else if ('MozWebSocket' in window) {
        websocket = new MozWebSocket("ws://localhost:8080/websocket");
    } 
    else {
        websocket = new SockJS("http://localhost:8080/websocket");
    }
    websocket.onopen = onOpen;
    websocket.onmessage = onMessage;
    websocket.onerror = onError;
    websocket.onclose = onClose;
              
    function onOpen(openEvt) {
        //alert(openEvt.Data);
    }
    
    function onMessage(evt) {
        //document.getElementById('publicMsg').innerHTML=evt.data.innerHTML=evt.data;
        var evet = document.getElementById('publicMsg');
        var para=document.createElement("p");
        var node=document.createTextNode(evt.data);
        para.appendChild(node);
        evet.appendChild(para);
    }

        websocket.onmessage = function(event) {
　　　　　　　　//接收来自服务器的数据，这里客户端没有发送任何请求，任何时间接收到数据都可以异步调用
              onMessage(event);
        };

    function onError() {}
    function onClose() {}
    
    function doSend() {
        if (websocket.readyState == websocket.OPEN) {          
            var msg = document.getElementById("inputMsg").value;  
            websocket.send(msg);//调用后台handleTextMessage方法
        } else {  
            alert("连接失败!");  
        }  
    }
　　　window.close=function()
　　　{
　　　　　websocket.onclose();
　　　}



</script>
请输入：<textarea rows="5" cols="30" id="inputMsg" name="inputMsg"></textarea>
<button onclick="doSend();">发送</button>


<div id="publicMsg" style=" width: 500px; height: 240px; border:1px solid #000;overflow-y:auto">
    
</div>
</body>
</html>
```
#### 实际效果
![](/images/01/1892_1.png)