---
author: wanls4583
comments: true
date: 2018-05-27
layout: post
title: webkit学习笔记-(3)Webkit构架和模块
categories:
- webkit

tags:
- webkit
---

## Webkit构架和模块

### Webkit构架

因为不同浏览器的需求，在Webkit中，一些代码是可以共享的，但是另一部分是不同的，这些不同的部分称为Webkit的移植(Ports)。

Webkit构架：

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-1.png)

WebCore部分包含了被各个浏览器所使用的 **Webkit 共享部分**，这些都是加载和渲染网页的基础部分，他们必不可少，具体包括HTML解释器、CSS解释器、SVG、DOM、渲染树(renderObject,renderLayer)，以及Inspector(调试器)。

JavaScriptCore 引擎是Webkit中默认的 Javascript 引擎。在Webkit中，对 Javascript 的调用是独立于引擎的。在 Google 的 Chrominum 中，它被替换成 V8 引擎。

**Webkit Ports** 指的是 Webkit 中非共享部分，对于不同浏览器使用的 Webkit 来说，移植中的这些模块由于平台差异、依赖的第三方库和需求不同等方面原因，往往按照自己的方式来设计和实现，这就产生了移植部分，这是导致众多 Webkit 版本的行为并非一致的重要原因。这其中包括硬件加速构架、网络栈、视频解码、图片解码等。

嵌入式编程接口是提供给浏览器调用的。因为接口与具体的移植有关，所以有一个与浏览器相关的绑定层（可能不同的平台调用的同一个功能的 webkit 的接口不一样）。绑定层上面就是 Webkit 项目对外暴露的接口层。

### Webkit源码目录

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-2.png)<br>

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-3.png)

重要的目录包括 JavaScriptCore、Platform、WebCore、Webkit、Webkit2。JavaScriptCore 是 Webkit 渲染引擎的默认 Javascript 引擎。Platform 本来是 Chrominum 接口代码目录之一，现已被移除。WebCore 就是前面图中 WebCore 对应的相关代码。Webkit 和 Webkit2 是绑定和嵌入式接口层。

## 基于 Blink 的 Chrominum 浏览器结构

### Chrominum 浏览器的架构及模块

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-4.png)

“Content模块”和“Content Shell”将下面的渲染机制、安全机制和插件机制等隐藏起来，提供一个接口层。该接口层被 Chrominum 浏览器、Content Shell等调用。Chrominum 浏览器和 Content Shell 是构建在 Content API之上的两个浏览器。

即使没有 Content 模块，浏览器的开发者也可以在 Webkit的 Chrominum 移植上渲染网页内容，但是却没有办法获得沙箱模型、跨进程的 GPU加速机制、众多的 HTML5 功能，以为这些都是在 Content 层里实现的。

#### 多进程模型

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-5.png)

Chrominum 多进程模型特征：

- Browser 进程和页面的渲染时分开的，页面渲染进程的崩溃不会导致浏览器主界面的崩溃
- 每个网页是独立的进程，这保证了页面之间相互不影响
- 插件进程也是独立的，插件本身的问题不会影响到网页和主界面
- GPU 硬件加速进程也是独立的，且只有一个 GPU 进程

在 Android 平台上，GPU 进程演变成 Borwser 进程的一个线程。由于 Android 系统的局限性，Rneder 进程的数目会被严格限制，所以移动端浏览器引入了“影子”标签的概念，移动端浏览器会将后台的网页所使用的渲染设施都清除，当用户再次切换回来的时候，网页需要重新加载和渲染。

#### Browser 进程和 Render 进程

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-6.png)

Webkit 黏附层的出现主要是因为 Chrominum 中的一些类型和 Webkit 内部不一致，所以需要一个简单的桥阶层。

Render 进程主要处理进程间通信，接受来自 Browser 进程的请求，并**调用相应的 Webkit 接口层**。同时，将 Webkit 的处理结果发送回去。

在 Browser 进程中，与 Render 进程相应的就是 RenderHost，其目的是处理同 Render 进程之间的通信，用来给 Render 进程发送请求并接收来自 Render 进程的结果。

#### 多线程模型

每个进程内部都有很多线程，对于 Browser 进程，多线程的目的主要是为了保持用户界面的高度响应，保证 UI（Browser 主线程）线程不会被其他费时的操作阻碍从而影响对用户操作的响应。而在 Render 进程中，Chrominum 则不让其他操作阻止渲染线程的快速运行。

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-7.png)

网页渲染的过程在进程模型中的工作方式如下：

- Browser 进程收到用户的请求，首先有 UI 线程处理，而且将相应的任务给 IO 线程，IO 线程随即将该任务传递给 Render 进程。
- Render 进程的 IO 线程经过简单的解释后交给**渲染线程**。渲染线程接受请求，加载网页并渲染网页，这其中可能需要 Browser 进程获取资源和需要 GPU 进程来帮助渲染。最后 Render 进程将结果由 IO 线程传递给 Browser 进程。
- 最后，Browser 进程接收到结果并将结果绘制出来。

#### Content接口

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-10.png)

Content 接口提供了一层对多进程进行渲染的抽象接口，其目标是要支持所有的 HTML5 功能、GPU 硬件加速功能和沙箱机制，这可以让 Content 接口的使用者们不需要很多的工作即可得到强大的能力。

Content 接口的相关代码按照功能分成六个部分，每个部分的接口一般可分成两类，第一类是调用者（Chrominum 浏览器、Content Shell等）调用的接口，另一类是调用者应该实现的**回调接口**，被 Content 接口的内部实现所调用。

### Chrominum Content 接口代码结构

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-8.png)

## Webkit2

### Webkit2 架构及模块

Webkit2 的思想同 Chrominum 类似，就是将渲染过程放在单独的进程中来完成，独立于用户界面。

![](http://wanls4583.github.io/images/posts/webkit/webkit构架-9.png)

“Web 进程”对应于 Chrominum 中的 Browser 进程，主要是渲染网页。“UI 进程”对应于 Chrominum 中的 Render 进程，接口就暴露在该进程，应用程序（浏览器）调用该接口即可。

### Webkit 和 Webkit2 嵌入式接口

Webkit 提供嵌入式接口，该接口表示其他程序可以将网页渲染嵌入在程序中作为其中的一部分。

在 Webkit 项目中，狭义 Webkit 的接口主要是与移植相关的 ewk_view 文件中的相关类。其主要思想是将网页的渲染结果作为用户界面的一个窗口部件，从这个角度上看，这跟其他的部件没有什么不同，区别在于它用来显示网页的内容。总结这些接口，按功能大致可以把所有接口分成6种类型：

- 加载网页、获取加载进度、停止加载、重新加载等；
- 遍历前后浏览记录，可以前进、后退等；
- 网页的很多设置，例如缩放、主题、背景、编码等；
- 查找网页内容、高亮等；
- 触控事件、鼠标事件处理；
- 查看网页源代码、显示调试窗口等；

Webkit2 接口不同于 Webkit 的接口，它们是不兼容的，不过目的都差不多，都是提供嵌入式的应用接口。

### 比较 Webkit2 和 Chrominum 的多进程模型

Webkit2 的多进程模型参考了 Chrominum 的模型和框架，也提供了多进程之上的接口层，但二者还是有一些区别的：

- Chrominum 使用的仍然是 Webkit 接口，而不是 Webkit2 接口，也就是说 Chrominum 是在 Webkit 接口之上构建的多进程架构；
- Webkit2 的接口将多进程结构隐藏起来，这样可以让应用程序不必纠缠于内部的细节。但是，对 Chrominum 来说，它的主要目的时给 Chrominum 提供 Content 接口以便构建 Chrominum 浏览器，其本身目标不是提供嵌入式接口；
- Chrominum 中每个进程都是从相同的二进制可执行文件启动，而基于 Webkit2 的进程则未必如此；


