---
author: wanls4583
comments: true
date: 2018-05-25
layout: post
title: webkit学习笔记-(2)HTML网页和结构
categories:
- webkit

tags:
- webkit
---

## 网页结构

### 框结构

框结构用来对网页的布局进行分割，将网页分成几个框，例如 frameset，frame，iframe 等元素都会形成新的框。每一个框结构包含一个 HTML 文档，最简单的框结构网页就是单一的框，当然，一个网页中也可以有多个框。

![](https://wanls4583.github.io/images/posts/webkit/网页结构-1.png)

### 层次结构

网页的层次结构是指网页中的元素可能分布在不同的层次中，也就是说某些元素可能不同于它的父元素所在的层，因为某些原因，Webkit 需要为该元素和它的子女建立一个新层。

## Webkit的网页渲染过程

在网页中，我们把当前可见的区域称之为视图。

Webkit的渲染过程如下图所示：

![](https://wanls4583.github.io/images/posts/webkit/网页结构-2.png)

加载完 HTML 文档后，渲染引擎开始创建DOM树，如果解析的过程中需要加载Javascript文件，则会停止当前 DOM 树的创建，直到 Javascript 文件加载并执行完成后才继续后面的DOM的创建。

HTML文档下载完成时会发出“DOMContent”事件，当HTML依赖的所有资源都加载完后会发出“onload”事件。

在解析 HTML 文档时，如果遇到 CSS 资源或者内联样式，Webkit 将会调用CSS解释器构建 CSSOM 树。随后，Webkit 在 DOM 树上附加上样式信息，构建出 RnederObject 树。RenderObject 树创建的同时，Webkit 会根据网页的层次结构创建 RenderLayer 树，同时创建一个虚拟的绘图上下文。

![](https://wanls4583.github.io/images/posts/webkit/网页结构-3.png)

图形上下文是一个与平台无关的抽象类，它将每个绘图操作桥接到不同的具体实现类（系统的图形库），最后由实现类来完成最终的绘制。