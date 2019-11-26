---
title: 02_WebSocket_API介绍
date: 2019-11-25 22:34:36
tags: 
 - WebSocket
categories:
 - WebSocket
---

# 02_WebSocket_API介绍

实现一个WebSocket应用程序需要如下步骤：

1. 开启连接
2. 客户端给服务器端发送数据
3. 服务器端接收数据
4. 服务器端给客户端发送数据
5. 客户端接收数据
6. 监听三类基本事件：
   - onopen：打开连接时的响应事件
   - onmessage：发送数据时的响应事件
   - onclose：关闭连接时的响应事件



前端页面：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>$Title$</title>
  </head>
  <body>
    <button onclick="connect();">open</button>
    <br>
    <input id="msg">
    <br>
    <button onclick="sendMsg()">send</button>
    <div id="textDiv"></div>
  </body>

  <script>
    var ws = null;
    function connect() {
      var target = "ws://localhost:8080/echo";
      if(target == ''){
        return;
      }
      if("WebSocket" in window){
        ws = new WebSocket(target);
      } else if ("MozWebSocket" in window){
        ws = new MozWebSocket(target);
      } else {
        return;
      }
      ws.onopen = function () {
        console.log("已开启")
      };
      ws.onmessage = function (ev) {
        document.getElementById("textDiv").innerText += ev.data;
      };
      ws.onclose = function () {
        console.log("已关闭")
      }
    }

    function sendMsg() {
      var msg = document.getElementById("msg").value;
      ws.send(msg);
      document.getElementById("msg").value="";
    }
  </script>
</html>
```



后台代码：

```java
@ServerEndpoint("/echo")
public class EchoSocket {

    public EchoSocket() {
        System.out.println("EchoSocket构造方法");
    }

    @OnOpen
    public void open(Session session) {
        // 一个session 代表 一个通信会话
        System.out.println("打开" + session.getId());
    }

    @OnClose
    public void close(Session session) {
        System.out.println("关闭" + session.getId());
    }

    @OnMessage
    public void message(Session session, String msg) throws IOException {
        System.out.println("接收客户端消息：" + msg);
        session.getBasicRemote().sendText("收到了信息");
    }

}
```



