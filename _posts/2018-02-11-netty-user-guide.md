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
