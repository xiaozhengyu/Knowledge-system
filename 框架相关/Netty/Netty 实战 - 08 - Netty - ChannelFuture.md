# Netty 对 Future-Listener 机制的应用之 ChannelFuture

[TOC]

## 代码



### 服务端

```java
package com.xzy;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 服务端
 *
 * @author xzy.xiao
 * @date 2022/8/22  15:39
 */
public class NettyServer {
    public static final Logger LOGGER = LoggerFactory.getLogger(NettyServer.class);
    public static final String ADDRESS = "127.0.0.1";
    public static final Integer PORT = 6668;

    public static void main(String[] args) throws InterruptedException {
        // 1.事件循环组
        EventLoopGroup boosGroup = new NioEventLoopGroup();     // 负责处理连接请求 —— 酒店前台（多个）
        EventLoopGroup workerGroup = new NioEventLoopGroup();   // 负责处理业务逻辑 —— 酒店接待员（多个）

        try {

            // 2.服务端启动器
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap
                    .group(boosGroup, workerGroup)
                    .handler(new LoggingHandler(LogLevel.INFO)) // boosGroup的channel处理器
                    .channel(NioServerSocketChannel.class) // 通道实现类
                    .option(ChannelOption.SO_BACKLOG, 128) // 阻塞数量
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() { // workerGroup的channel处理器
                        // This method will be called once the {@link Channel} was registered.
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new NettyServerHandler()); // 自定义的处理器
                        }
                    });

            // 3.启动服务端，开始监听
            LOGGER.info("Server is ok...");
            ChannelFuture bindFuture = serverBootstrap.bind(PORT).sync();
            bindFuture.addListener(future -> {
                if (future.isSuccess()) {
                    LOGGER.info("Bind port succeed");
                } else {
                    LOGGER.warn("Bind port failed");
                }
            });


            // 4.关闭
            ChannelFuture closeFuture = bindFuture.channel().closeFuture().sync();
            closeFuture.addListener(future -> {
                if (future.isSuccess()) {
                    LOGGER.info("Close server succeed");
                } else {
                    LOGGER.warn("Close server failed");
                }
            });

        } finally {

            boosGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();

        }
    }
}
```



### 客户端

```java
package com.xzy;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import static com.xzy.NettyServer.ADDRESS;
import static com.xzy.NettyServer.PORT;

/**
 * 客户端
 *
 * @author xzy.xiao
 * @date 2022/8/22  15:40
 */
public class NettyClient {

    public static final Logger LOGGER = LoggerFactory.getLogger(NettyClient.class);

    public static void main(String[] args) throws InterruptedException {
        // 1.事件循环组
        EventLoopGroup eventLoopGroup = new NioEventLoopGroup();

        try {

            // 2.客户端启动器
            Bootstrap clientBootStrap = new Bootstrap();
            clientBootStrap
                    .group(eventLoopGroup)
                    .channel(NioSocketChannel.class) // 通道实现类
                    .handler(new ChannelInitializer<SocketChannel>() {
                        // This method will be called once the {@link Channel} was registered.
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new NettyClientHandler());// 处理器
                        }
                    });

            // 3.启动客户端，开始连接
            LOGGER.info("Client is ok...");
            ChannelFuture connectFuture = clientBootStrap.connect(ADDRESS, PORT).sync();
            connectFuture.addListener(future -> {
                if (future.isSuccess()) {
                    LOGGER.info("Connect server succeed");
                } else {
                    LOGGER.warn("Connect server failed");
                }
            });

            // 4.关闭
            ChannelFuture closeFuture = connectFuture.channel().closeFuture().sync();
            closeFuture.addListener(future -> {
                if (future.isSuccess()) {
                    LOGGER.info("Close client succeed");
                } else {
                    LOGGER.warn("Close client failed");
                }
            });
        } finally {

            eventLoopGroup.shutdownGracefully();

        }
    }
}
```

## 演示