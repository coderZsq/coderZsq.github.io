---
layout: post
title: UI/UX 产品原型 从Axure开始
date: 2016.05.28 15:40
tag: UI设计
---

Tools Axure RP 7.0

俗话说的好, 不懂产品的设计师 不是一个好开发者! 作为一个iOS开发者, 也势必随波逐流的力争跻身全栈的行列, 话说产品是什么? 什么是产品? 什么是好产品? 好产品能为我们带来什么? 作为产品小白 还知知甚浅.. 但是我相信人人都是产品经理, 每个开发者都有一个做产品的心, 因为一个好产品背后带来的效应和效益是巨大的.

接着上周的项目 上期我们已经把项目的基本框架给搭出来了,也集成了Reveal 可以实时监控UI的层次的变化 那Lifestyle到底是个什么项目呢, 这一切还得从原型说起.

产品介绍: Lifestyle 是一款发现和分享的创意图区App

v1.0 的需求分析: 实现基本的发推功能, 实现基本的搜索功能, 实现基本登录注册功能, 实现基本的收藏功能.

那什么是原型呢?

产品原型简单的说就是产品设计成形之前的一个简单框架，对App来讲，就是将页面模块、元素进行粗放式的排版和布局，深入一些，还会加入一些交互性的元素，使其更加具体、形象和生动。就我个人而言，目前使用频率最多，最高效，交互效果最好的原型工具无疑是Axure。

![](http://upload-images.jianshu.io/upload_images/1229762-4a363c5476b02861.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/625)


1.主要页面原型

作为一个开发者, 很负责任的告诉各位产品 你们做的那些高保真原型图 不仅浪费时间还很少有人会看! 对于本人这样客户端的开发 基本就看设计师给的设计图(图上有精确的 颜色 字体 间距 精确到px), 对于复杂的业务逻辑再看下产品需求, 和后端定义的接口文档.

以下为Lifestyle 项目的一些主要页面原型 (低保真原型图)

![Lifestyle 页面](http://upload-images.jianshu.io/upload_images/1229762-cfd8cd134724265c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

![Discover 页面](http://upload-images.jianshu.io/upload_images/1229762-2a9d96195369b7a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

![Profile 页面](http://upload-images.jianshu.io/upload_images/1229762-50af5e9ebec2b0dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

2.页面流程图

在确定主要页面之后，我们可以开始细化页面流程了。页面流程图有利于向大家展示自己的想法，也有利于思路的整理。毕竟Axure上面的交互点是散的，通过页面流程图，我们可以整理所有的页面上的交互行为，避免遗漏；在向他人展示的时候，也可以一目了然的看出需要的操作步骤是多少。

以下为Lifestyle 项目的页面流程图 (低保真原型图)

![](http://upload-images.jianshu.io/upload_images/1229762-8c5c276eaaaf3b98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

页面流程图
3.完善原型

页面的主要页面和页面流程确定之后，就可以完善原型了。为每个页面加上点击事件 这样能够更加直观的让开发人员了解到产品的业务逻辑 当然复杂的逻辑还是需要文档标注的 个人建议所有的页面交互的事件 都需要把跳转交互的可能性做标注 这样可以极大的减少沟通成本 大大加快了开发人员的工作效率!

建议如下图标注显示

![](http://upload-images.jianshu.io/upload_images/1229762-30cc460a07e5c267.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

Post页面
总结:
在制作原型的初期请把所有精力都放在流程的优化和布局设计上面吧，不要把时间浪费在视觉设计上，那样绝对是得不偿失。因为你们的原型方案还没有最终通过，肯定要通过多次迭代才能确定方案，太多的视觉设计就是浪费时间；再者，如果原型做的十分逼真，在产品讨论会上，视觉的元素会很快抓住大家的眼球，到时就会有领导来质疑你的界面是不是该换成蓝色，按钮是不是再精致一些这样的问题。没有人会专注于你的交互设计了。

Axure其实只是一种交互工具而已，交互设计最重要的还是想法，工具只是来帮你表现想法的。不必过于追求技术，不必过于追求视觉表现。我们在把握好整个产品方向的同时，应专注于交互流程、页面内交互、布局结构的创新和优化。

对于只学了1天Axure的码农的拙见, 各位产品大大看过笑过就好 有好的建议也请不吝惜提出 小弟在这先谢过了~

具体原型rp文件 请到[github](https://github.com/coderZsq/coderZsq.project.oc)上进行下载!



