---
title: SpringBoot整合Websocket
category: springboot
order: 19
---



### 后端

#### pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```



#### WebSocketConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class WebSocketConfig{

    /**
     * ServerEndpointExporter 作用
     *
     * 这个Bean会自动注册使用@ServerEndpoint注解声明的websocket endpoint
     *
     * @return
     */
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}

```



#### WebSocketServer.java

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

@ServerEndpoint("/webSocket/{sid}")
@Component
public class WebSocketServer {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static AtomicInteger onlineNum = new AtomicInteger();

    //concurrent包的线程安全Set，用来存放每个客户端对应的WebSocketServer对象。
    private static ConcurrentHashMap<String, Session> sessionPools = new ConcurrentHashMap<>();

    //发送消息
    public void sendMessage(Session session, String message) throws IOException {
        if(session != null){
            synchronized (session) {
                logger.info("发送数据：" + message);
                session.getBasicRemote().sendText(message);
            }
        }
    }

    //给指定用户发送信息
    public void sendInfo(String userName, String message){
        Session session = sessionPools.get(userName);
        try {
            sendMessage(session, message);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    //建立连接成功调用
    @OnOpen
    public void onOpen(Session session, @PathParam(value = "sid") String userName){
        sessionPools.put(userName, session);
        addOnlineCount();
        logger.info(userName + "加入webSocket！当前人数为" + onlineNum);
    }

    //关闭连接时调用
    @OnClose
    public void onClose(@PathParam(value = "sid") String userName){
        sessionPools.remove(userName);
        subOnlineCount();
        logger.info(userName + "断开webSocket连接！当前人数为" + onlineNum);
    }

    //收到客户端信息
    @OnMessage
    public void onMessage(String message) throws IOException{
        message = "客户端：" + message + ",已收到";
        logger.info(message);
        for (Session session: sessionPools.values()) {
            try {
                sendMessage(session, message);
            } catch(Exception e){
                e.printStackTrace();
                continue;
            }
        }
    }

    //错误时调用
    @OnError
    public void onError(Session session, Throwable throwable){
        logger.info("发生错误");
        throwable.printStackTrace();
    }

    public static void addOnlineCount(){
        onlineNum.incrementAndGet();
    }

    public static void subOnlineCount() {
        onlineNum.decrementAndGet();
    }
}

```



### 前端vue

```shell
npm install vue-native-websocket --save
```



#### Websocket.vue

```vue
<template>
  <div>
      {{ msg }}
  </div>
</template>
<script>
import { USER_NAME } from '@/store/mutation-types'
import Vue from 'vue'
import websocket from 'vue-native-websocket'
Vue.prototype.$websocket = websocket
export default {
  name: 'Websocket',
  data() {
    return {
      websock: null,
      msg: ''
    }
  },
  created() {
    this.initWebSocket()
  },
  destroyed() {
    this.websocketclose()
  },
  components: { DayReportForm },
  methods: {
    initWebSocket() {
      const websocketUrl = process.env.VUE_APP_WEBSOCKET_URL + `/webSocket/${Vue.ls.get(USER_NAME)}`
      this.$websocket = new WebSocket(websocketUrl)
      this.$websocket.onopen = this.websocketonopen
      this.$websocket.onerror = this.websocketonerror
      this.$websocket.onmessage = this.websocketonmessage
      this.$websocket.onclose = this.websocketclose
    },

    websocketonopen() {
      console.log('WebSocket连接成功')
    },
    websocketonerror() {
      console.log('WebSocket连接发生错误')
    },
    websocketonmessage(e) {
      this.msg = e.data
      const msgJson = JSON.parse(e.data)
      const key = `open${Date.now()}`
      this.$notification.open({
          message: msgJson.title,
          key,
          btn: h => {
              return h(
                  'a-button',
                  {
                      props: {
                          type: 'primary',
                          size: 'small'
                      },
                      on: {
                          click: () => {
                              // TODO 业务逻辑
                              
                              // 关闭弹窗
                              this.$notification.close(key)
                          }
                      }
                  },
                  '查看详情'
              )
          },
          duration: 5,
          icon: <a-icon type="smile" style="color: #108ee9" />
      })
    },
    websocketclose(e) {
      console.log('connection closed (' + e.code + ')')
    }
  }
}

 </script>

```

