# springboot-整合netty-socketio

### pom文件

```xml
			 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <!-- netty-socketio： 仿`node.js`实现的socket.io服务端 -->
        <dependency>
            <groupId>com.corundumstudio.socketio</groupId>
            <artifactId>netty-socketio</artifactId>
            <version>2.0.3</version>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.18</version>
        </dependency>
```

### socket服务配置

```yml
#自定义socketio配置,你可以直接硬编码，看个人喜好
socketio:
  # socketio请求地址
  host: 0.0.0.0
  # socketio端口
  port: 9998
  # 设置最大每帧处理数据的长度，防止他人利用大数据来攻击服务器
  maxFramePayloadLength: 1048576
  # 设置http交互最大内容长度
  maxHttpContentLength: 1048576
  # socket连接数大小（如只监听一个端口boss线程组为1即可）
  bossCount: 1
  # 连接数大小
  workCount: 100
  # 允许客户请求
  allowCustomRequests: true
  # 协议升级超时时间（毫秒），默认10秒。HTTP握手升级为ws协议超时时间
  upgradeTimeout: 1000000
  # Ping消息超时时间（毫秒），默认60秒，这个时间间隔内没有接收到心跳消息就会发送超时事件
  pingTimeout: 6000000
  # Ping消息间隔（毫秒），默认25秒。客户端向服务器发送一条心跳消息间隔
  pingInterval: 25000
  # 命名空间，多个以逗号分隔，每个空间需要对应一个Bean的名字
  namespaces: /chat
```

### socketIO的config代码

```java
package com.yolo.demo.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@Data
@ConfigurationProperties(prefix = "socketio")
public class SocketIoProperties {


    private String host;

    private Integer port;

    private int bossCount;

    private int workCount;

    private boolean allowCustomRequests;

    private int upgradeTimeout;

    private int pingTimeout;

    private int pingInterval;

    private String[] namespaces;

}

```

```java
package com.yolo.demo.config;

import com.corundumstudio.socketio.SocketConfig;
import com.corundumstudio.socketio.SocketIOServer;
import com.corundumstudio.socketio.annotation.SpringAnnotationScanner;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;
import java.util.Optional;

//@Configuration
@RequiredArgsConstructor
public class SocketIOConfig {


    private final SocketIoProperties socketIoProperties;

    @Bean(name = "SocketIOServer")
    public SocketIOServer socketIoServer() {
        SocketConfig socketConfig = new SocketConfig();
        socketConfig.setTcpNoDelay(true);
        socketConfig.setSoLinger(0);
        com.corundumstudio.socketio.Configuration config = new com.corundumstudio.socketio.Configuration();
        config.setSocketConfig(socketConfig);
        config.setHostname(socketIoProperties.getHost());
        config.setPort(socketIoProperties.getPort());
        config.setBossThreads(socketIoProperties.getBossCount());

        config.setWorkerThreads(socketIoProperties.getWorkCount());
        config.setAllowCustomRequests(socketIoProperties.isAllowCustomRequests());
        config.setUpgradeTimeout(socketIoProperties.getUpgradeTimeout());
        config.setPingTimeout(socketIoProperties.getPingTimeout());
        config.setPingInterval(socketIoProperties.getPingInterval());

        //服务端
        final SocketIOServer server = new SocketIOServer(config);

        //添加命名空间（如果你不需要命名空间，下面的代码可以去掉）
        Optional.ofNullable(socketIoProperties.getNamespaces()).ifPresent(nss ->
                Arrays.stream(nss).forEach(server::addNamespace));


        return server;
    }

    //这个对象是用来扫描socketio的注解，比如 @OnConnect、@OnEvent
    @Bean
    public SpringAnnotationScanner springAnnotationScanner(@Qualifier("SocketIOServer") SocketIOServer socketIoServer) {

        return new SpringAnnotationScanner(socketIoServer);
    }
}

```

### 这是存储用户的缓存信息

```java
package org.mye.framework.engine.socketio;

import cn.hutool.json.JSONUtil;
import com.corundumstudio.socketio.SocketIOClient;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 这是存储用户的缓存信息
 */
@Component
@Slf4j
public class ClientCache {


    /**
     * 用于存储用户的socket缓存信息
     * 用户id   sessionId  SocketIOClient
     */
    public static ConcurrentHashMap<String, HashMap<UUID, SocketIOClient>> CONCURRENT_HASH_MAP = new ConcurrentHashMap<>();

    /**
     * 保存用户信息
     *
     * @param userId         用户id
     * @param sessionId      会话id
     * @param socketIoClient socketIoClient
     */
    public void saveClient(String userId,UUID sessionId,SocketIOClient socketIoClient){
        HashMap<UUID, SocketIOClient> sessionIdClientCache = CONCURRENT_HASH_MAP.get(userId);
        if(sessionIdClientCache == null){
            sessionIdClientCache = new HashMap<>();
        }
        sessionIdClientCache.put(sessionId,socketIoClient);
        CONCURRENT_HASH_MAP.put(userId,sessionIdClientCache);
    }


    /**
     * 获取用户信息
     *
     * @param userId 用户id
     * @return {@link HashMap}<{@link UUID},{@link SocketIOClient}>
     */
    public HashMap<UUID,SocketIOClient> getUserClient(String userId){
        return CONCURRENT_HASH_MAP.get(userId);
    }


    /**
     * 根据用户id和session删除用户某个session信息
     *
     * @param userId    用户id
     * @param sessionId 会话id
     */
    public void deleteSessionClientByUserId(String userId,UUID sessionId){
        CONCURRENT_HASH_MAP.get(userId).remove(sessionId);
    }


    /**
     * 删除用户缓存信息
     *
     * @param userId 用户id
     */
    public void deleteUserCacheByUserId(String userId){
        CONCURRENT_HASH_MAP.remove(userId);
    }
}
```

### 创建namespace所属的Handler

```java
package org.mye.framework.engine.socketio;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.json.JSONUtil;
import com.corundumstudio.socketio.AckRequest;
import com.corundumstudio.socketio.SocketIOClient;
import com.corundumstudio.socketio.SocketIOServer;
import com.corundumstudio.socketio.annotation.OnEvent;
import com.fasterxml.jackson.core.JsonProcessingException;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.mye.framework.engine.websocket.WsInfo;
import org.springframework.stereotype.Component;

import java.util.Calendar;
import java.util.HashMap;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

@Component(value= "fireFightingMessageEventHandler")
@Slf4j
@RequiredArgsConstructor
public class FireFightingMessageEventHandler {

    private final SocketIOServer socketIOServer;

    private final ClientCache clientCache;
    private static final String NAMESPACE = "/fireFighting";

    //测试使用
    @OnEvent("socketIOHandler")
    public void testHandler(SocketIOClient client, String data, AckRequest ackRequest){
        log.info("SocketIOHandler:{}",data);
        if(ackRequest.isAckRequested()){
            //返回给客户端，说我接收到了
            ackRequest.sendAckData("SocketIOHandler",data);
        }
    }


    //加入房间
    @OnEvent("joinRoom")
    public void joinRooms(SocketIOClient client, String data, AckRequest ackRequest){
        client.joinRoom(data);
        if(ackRequest.isAckRequested()){
            //返回给客户端，说我接收到了
            ackRequest.sendAckData("加入房间","成功");
        }
    }


    //离开房间
    @OnEvent("leaveRoom")
    public void leaveRoom(SocketIOClient client, String data, AckRequest ackRequest){
        client.leaveRoom(data);
        if(ackRequest.isAckRequested()){
            //返回给客户端，说我接收到了
            ackRequest.sendAckData("离开房间","成功");
        }
    }

    //获取该用户所有房间
    @OnEvent("getUserRooms")
    public void getUserRooms(SocketIOClient client, String data, AckRequest ackRequest){
        String userId = client.getHandshakeData().getSingleUrlParam("userId");
        Set<String> allRooms = client.getAllRooms();
        for (String room:allRooms){
            System.out.println("房间名称："+room);
        }
        log.info("服务器收到消息,客户端用户id：{} | 客户发送的消息：{} | 是否需要返回给客户端内容:{} ",userId,data,ackRequest.isAckRequested());

        if(ackRequest.isAckRequested()){
            //返回给客户端，说我接收到了
            ackRequest.sendAckData("你好","哈哈哈");
        }
    }


    @OnEvent("sendRoomMessage")
    public void sendRoomMessage(SocketIOClient client, String data, AckRequest ackRequest){
        String userId = client.getHandshakeData().getSingleUrlParam("userId");
        Set<String> allRooms = client.getAllRooms();
        for (String room:allRooms){
            log.info("房间：{}",room);
            //发送给指定空间名称以及房间的人，并且排除不发给自己
            socketIOServer.getNamespace(NAMESPACE).getRoomOperations(room).sendEvent("message",client, data);
            //发送给指定空间名称以及房间的人，包括自己
            //socketIoServer.getNamespace("/socketIO").getRoomOperations(room).sendEvent("message", data);;
        }
        if(ackRequest.isAckRequested()){
            //返回给客户端，说我接收到了
            ackRequest.sendAckData("发送消息到指定的房间","成功");
        }

    }

    //广播消息给指定的Namespace下所有客户端
    @OnEvent("sendNamespaceMessage")
    public void sendNamespaceMessage(SocketIOClient client, String data, AckRequest ackRequest){
        socketIOServer.getNamespace(NAMESPACE).getBroadcastOperations().sendEvent("message",client, data);;
        if(ackRequest.isAckRequested()){
            //返回给客户端，说我接收到了
            ackRequest.sendAckData("发送消息到指定的房间","成功");
        }
    }


    /**
     * 发送消息 全部客户端
     */
    public void pushMessage(WsInfo wsInfo){
        ConcurrentHashMap<String, HashMap<UUID, SocketIOClient>> concurrentHashMap = ClientCache.CONCURRENT_HASH_MAP;

        if (CollUtil.isNotEmpty(concurrentHashMap)){
            for (String userId : concurrentHashMap.keySet()) {
                HashMap<UUID, SocketIOClient> sessionIdClientCacheMap = concurrentHashMap.get(userId);
                for (UUID sessionId : sessionIdClientCacheMap.keySet()) {
                    socketIOServer.getNamespace(NAMESPACE).getClient(sessionId).sendEvent("fireFightingMessage", wsInfo);
                }
            }
        }
    }
}
```

### 建立关闭连接

```java
package org.mye.framework.engine.socketio;

import com.corundumstudio.socketio.SocketIOClient;
import com.corundumstudio.socketio.annotation.OnConnect;
import com.corundumstudio.socketio.annotation.OnDisconnect;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.UUID;
import java.util.concurrent.atomic.AtomicInteger;


@Slf4j
@Component
@RequiredArgsConstructor
public class SocketIOConnectHandler {

    private final ClientCache clientCache;

    /**
     * 连接数
     */
    public static AtomicInteger onlineCount = new AtomicInteger(0);


    /**
     * 建立连接
     *
     * @param client 客户端的SocketIO
     */
    @OnConnect
    public void onConnect(SocketIOClient client) {

        //因为我定义用户的参数为userId，你也可以定义其他名称 客户端请求 http://localhost:9999?userId=12345
        String userId = client.getHandshakeData().getSingleUrlParam("userId");
        if (userId == null) {
            System.err.println("客户端" + client.getSessionId() + "建立websocket连接失败，userId不能为null");
            client.disconnect();
            return;
        }

        //同一个页面sessionid一样的
        UUID sessionId = client.getSessionId();

        //保存用户的信息在缓存里面
        clientCache.saveClient(userId, sessionId, client);
        onlineCount.addAndGet(1);
        log.info("SocketIOServerHandler-用户id:{},sessionId:{},建立连接成功,在线人数:{}", userId, sessionId,onlineCount.get());


    }

    /**
     * 关闭连接
     *
     * @param client 客户端的SocketIO
     */
    @OnDisconnect
    public void onDisconnect(SocketIOClient client) {

        //因为我定义用户的参数为userId，你也可以定义其他名称
        String userId = client.getHandshakeData().getSingleUrlParam("userId");

        //sessionId,页面唯一标识
        UUID sessionId = client.getSessionId();

        //clientCache.deleteUserCacheByUserId(userId);
        //只会删除用户某个页面会话的缓存，不会删除该用户不同会话的缓存，比如：用户同时打开了谷歌和QQ浏览器，当你关闭谷歌时候，只会删除该用户谷歌的缓存会话
        clientCache.deleteSessionClientByUserId(userId, sessionId);
        onlineCount.addAndGet(-1);
        log.info("SocketIOServerHandler-用户id:{},sessionId:{},关闭连接成功,在线人数:{}", userId, sessionId,onlineCount.get());
    }


}


```

### SocketIOServer启动或关闭

```java
package org.mye.framework.engine.socketio;

import com.corundumstudio.socketio.SocketIOServer;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
@Slf4j
@RequiredArgsConstructor
class SocketIOServerRunner implements CommandLineRunner, DisposableBean {


    private final SocketIOServer socketIoServer;

    private final FireFightingMessageEventHandler fireFightingMessageEventHandler;



    @Override
    public void run(String... args) {
        //namespace分别交给各自的Handler监听,这样就可以隔离，只有客户端指定namespace，才能访问对应Handler。
        //比如：http://localhost:9999/fireFighting?userId=12345
        socketIoServer.getNamespace("/fireFighting").addListeners(fireFightingMessageEventHandler);

        socketIoServer.start();
        log.info("SocketIOServer==============================启动成功");
    }


    @Override
    public void destroy() {
        //如果用kill -9  这个监听是没用的，有可能会导致你服务kill掉了，但是socket服务没有kill掉
        socketIoServer.stop();
        log.info("SocketIOServer==============================关闭成功");
    }
}
```

> 参考地址：
>
> [日常记录-SpringBoot整合netty-socketio](https://blog.csdn.net/qq407995680/article/details/131956249)
>
> [springboot学习(四十三) springboot使用netty-socketio实现消息推送](https://blog.csdn.net/u011943534/article/details/115232123)
>
> [Spring-Boot快速集成netty-socketio(socket服务实现，支持认证)](https://blog.csdn.net/qq_42271561/article/details/107892447)

