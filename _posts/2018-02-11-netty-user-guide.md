---
layout: post
title: Netty 4.x 用户指南翻译
date:   2018-02-11
categories: "netty"
tags: [netty, java]
author: Leason
comment: false
---

## 前言

### 问题在哪

如今我们都会使用通用应用程序和库来相互通信。
举个例子，我们通常使用一个HTTP通信库来从一个web服务器获取信息并且通过web services调用RPC。
然而，有时候传统的协议及其实现不能很好的扩展。
就像我们不能使用HTTP协议来交换大容量文件、电子邮件信息和一些类似于财务信息和多人游戏数据的高实时性消息。
这些所需要的是一个专用于特殊用途的高度优化的协议。例如，你可能想实现一个专门对ajax聊天程序、媒体流或者大文件传输进行优化的HTTP服务器。
你甚至可以设计实现一个完全根据你的需要量身定制的新协议。不可避免的是你必须处理传统的专有协议来确保新旧系统的兼容性。
在这种情况下，重要的是我们既能更快的实现这个协议也不会牺牲最终应用程序的性能和稳定性。

## 解决方案

[The Netty project](http://netty.io)旨在为快速开发可维护高性能和高扩展性协议的服务器和客户端提供一个异步的基于事件驱动的网络应用框架和工具。

换句话说，Netty是一个NIO的客户端服务器框架，可以快速和简单的开发如协议服务器和客户端的网络应用程序。
它很大程度的简化了TCP和UDP套接字等网络编程的开发。

‘又快又简单’并不意味着最终的应用程序会在稳定性和性能上遭受损失。
Netty从大量的像FTP、SMTP、HTTP和各种基于二进制和文本的传统协议中汲取经验，精心设计。
正因如此，Netty具有开发简单、高性能、高稳定性和灵活性的特点。

某些用户可能已经发现一些其他的网络框架宣称有相同的优点，你可能会问Netty和它们有什么不同。
答案就是它们出发点不同。Netty设计的原则是为了从一开始为你提供在API和实现方面最舒适的体验。
这是看不见的，但你会意识到，当你阅读本指南并使用Netty时这些都会使你工作起来更轻松。

## 入门

本章围绕Netty的核心架构，用简单的例子让你快速入门。在本章最后你将能够写一个基于Netty的客户端服务器程序。

如果你更喜欢自上而下的学习方法，你可能需要从第二张‘体系结构概述’开始，然后返回到这里。

### 入门之前

运行本章示例程序的最低要求:最新的Netty、JDK不低于1.6。
最新的Netty可以在[这里](http://netty.io/downloads.html)获取到。
要下载正确版本的JDK，请参阅你的首选JDK供应商网站。

当你阅读时，你也许会对本章介绍的类有疑问，你可以参考API手册。为了你的方便，本文档的所有类名都链接到在线API参考。
另外，你可以在[这里](http://netty.io/community.html)进行社区交流，让我们知道是否存在语法错误和错别字，或者你有更好的建议来帮助我们改进本文档。

### 写一个Discard Server

最简单的协议不是‘Hello World’，而是[DISCARD](https://tools.ietf.org/html/rfc863)。
是一种丢弃任何接收到的数据并且不进行任何响应的协议。

实现DISCARD协议，你仅仅需要做的是忽略任何接受到的数据。
让我们直接从handler实现开始，它处理由Netty产生的I/O事件。

{% highlight java %}
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
{% endhighlight %}

1. DiscardServerHandler继承自[ChannelInboundHandlerAdapter](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html)，它是[ChannelInboundHandler](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html)的一种实现。
[ChannelInboundHandler](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html)提供多种你可以用来重写的event handler方法。现在只用扩展[ChannelInboundHandlerAdapter](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html)就够了，而不是自己实现这个handler接口。

2. 在这，我们重写了channelRead() 事件处理方法。这个方法将在收到消息时被调用，而不管是不是从客户端发来的新消息。本例中，接收到的消息类型是[ByteBuf](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)。

3. 为了实现DISCARD协议，handler必须忽略所收到的消息。[ByteBuf](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)是一个reference-counted对象，必须通过release()方法释放。请注意，通过handler的任意reference-counted对象必须由其自身释放。通常，channelRead()方法的实现如下:

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```

4. 当Netty发生I/O错误或者handler实现中事件操作抛出异常，exceptionCaught()方法将被调用。大多数情况下，尽管这个方法的实现会根据你具体如何处理异常情况而不同，但是捕获的异常在这应该都被记录并且关闭其相关联的channel。

目前为止，我们已经实现了DISCARD服务器的前一半。我们剩下来的工作就是写一个main()方法来启动这个DiscardServerHandler服务器。

```java
package io.netty.example.discard;
    
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        } else {
            port = 8080;
        }
        new DiscardServer(port).run();
    }
}
```

1. [NioEventLoopGroup](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html)是一个多线程事件轮询器来处理I/O操作。在这个例子中，我们实现一个服务端的应用程序，因此将使用两个[NioEventLoopGroup](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html)。第一个通常被称为‘boss’，用来接收传过来的连接。第二个通常被称为‘worker’，一旦‘boos’接收到连接并将其注册给‘worker’，‘worker’将会处理连接中的流量。使用多少个线程以及如何映射到已创建的(Channel)[http://netty.io/4.0/api/io/netty/channel/Channel.html]s取决于[EventLoopGroup](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html)的实现，并且甚至可以通过构造方法来进行配置。

2. [ServerBootstrap](http://netty.io/4.0/api/io/netty/bootstrap/ServerBootstrap.html)是一个可以启动服务器的帮助类。你也可以直接通过(Channel)[http://netty.io/4.0/api/io/netty/channel/Channel.html]启动服务器。然而这是一个乏味的过程，大多数情况下并不需要那么做。

3. 在这里我们明确使用[NioServerSocketChannel](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)类用来实例化一个新的[Channel](http://netty.io/4.0/api/io/netty/channel/Channel.html)，作用是接收传过来的连接。

4. 这个handler通常处理新接收的Channel。[ChannelInitializer](http://netty.io/4.0/api/io/netty/channel/ChannelInitializer.html)是一个特殊的handler，用来帮助用户配置新的[Channel](http://netty.io/4.0/api/io/netty/channel/Channel.html)。你很可能希望通过添加一些handler(如DiscardServerHandler)来配置新[Channel](http://netty.io/4.0/api/io/netty/channel/Channel.html)的[ChannelPipeline](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html),以实现你的网络应用程序。当应用变得复杂时，你可能会在pipeline中添加更多的channel，并最终提取到 新的/顶级 类中。

5. 你也可以设置参数用来指定Channel的实现。我们正在实现一个TCP/IP服务器，所以我们可以设置scoket选项，类似于tcpNoDelay和keepAlive之类的。请参考关于[ChannelOption](http://netty.io/4.0/api/io/netty/channel/ChannelOption.html)的api文档和[ChannelConfig](http://netty.io/4.0/api/io/netty/channel/ChannelConfig.html)的实现，以获得所支持ChannelOptions的大致信息。

6. 你注意到option()和childOption()方法了吗？option()用于接收传过来的[NioServerSocketChannel](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)。childOption()适用于父[ServerChannel](http://netty.io/4.0/api/io/netty/channel/ServerChannel.html)接收的channel，在这个例子中也就是[NioServerSocketChannel](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)。

7. 现在一切准备就绪，剩下来的就是绑定端口启动服务。在这里，我们绑定主机的所有NIC(network interface cards)的8080端口。你可以调用bind()方法多次(用来绑定多个地址)。

恭喜，你已经完成基于Netty的第一个服务器。