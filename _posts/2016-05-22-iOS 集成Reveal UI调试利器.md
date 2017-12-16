---
layout: post
title: iOS 集成Reveal UI调试利器
date: 2016.05.22 11:51
tag: 移动开发
---

Tools Reveal 1.5.1

在 iOS 开发中，我们有时很希望有一款类似 Web 开发中的 UI Debug 工具（例如：Firebug），让我们能够实时查看 UI 的结构，还可以实时更改某个 UIView 的位置和大小的相关属性值查看效果。这里我们发现原来真有这么一款强大的工具存在，他就是 Reveal。(虽然Xcode6 之后有「Capture View Hierarchy」功能，但支持的功能还只是基础的查看 UI 结构，对比 Reveal 来说，就显得逊色多了）。

接着上一篇,  我们先在项目中创建一些Controller,  由于是1.0版本,  就先创建3个Controller 继承UIViewController,  其实也可以按照需求继承UITableViewController 或 UICollectionViewController. 在SQBundleVersionManager.m 中导入Controller的头文件, 并添加对应的title. icon图片先填空字符串即可.

![](http://upload-images.jianshu.io/upload_images/1229762-574399e90f515151.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


配置Controller 信息
此时我们发现ProfileViewController不显示图标和不支持切换, 难道是SQExtension 的框架有问题吗? 并不是!! 由于之前的项目有用到像新浪微博的中间式弹出Controller 故我们先将此功能关闭!!

![](http://upload-images.jianshu.io/upload_images/1229762-f6a4109212a9123b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


将centerButton的 hidden属性设为YES
设置完成之后我们就能够发现ProfileViewController已经可以切换显示了, 接下来我们在LifestyleViewController中创建TableView 设置Delegate 实现创建代理方法.

![](http://upload-images.jianshu.io/upload_images/1229762-0f3643bfafa9b58e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


创建TableView
这里分享给大家一个小技巧 将代码拖入代码库 在想要输入的地方按照<#input#>格式输入,Completion Shortcut中输入对应的快捷键 点击Done 按钮即可. 下图为懒加载的固定写法 写入代码库既可以提高开发效率,又可以保持代码整洁有规范.

![](http://upload-images.jianshu.io/upload_images/1229762-4a8e1d06decf0a00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码库配置
接下来我们就要在项目中集成Reveal UI调试利器. 其实Reveal不仅能够实时查看UI结构 Vim配置及导入静态库 Other Linker Flags项增加-ObjC -framework Reveal之后 在越狱的iOS设备下 还可以逆向工程(即可以看到任何App 的UI层次结构来推算实现方法) 总之狂霸酷炫屌炸天!!

今天要分享的是不修改Xcode工程并加载Reveal(此方法仅适用于在iOS模拟器上运行的应用) 通过不修改Xcode工程文件来加载Reveal的方式，您可以检视任何一个您正在开发的iOS应用，而不需要对这些应用的工程做任何修改。另一个好处就是，您不需要再担心，犯下一不小心将Reveal库连接到应用中发布了的错误。

1.打开您的iOS工程，选择View → Navigators → Show Breakpoint Navigator

![](http://upload-images.jianshu.io/upload_images/1229762-763df3b13a429de1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开您的iOS工程，选择View → Navigators → Show Breakpoint Navigator
2.在面板左下角，点击+按钮并选择**Add Symbolic Breakpoint**  

3.在Symbol输入区内输入UIApplicationMain 点击Add Action按钮, 确认Action被设置为Debugger Command。

4.将 expr (Class)NSClassFromString(@"IBARevealLoader") == nil ? (void *)dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib", 0x2) : ((void*)0) 拷贝至action 的输入区

5.选中Automatically continue after evaluating actions选项。

![](http://upload-images.jianshu.io/upload_images/1229762-276560e372a80833.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意: 请确认Reveal.app的路径信息符合您Mac的实际位置。

6.右击刚才新创建的断点，选择Move Breakpoint To → User

![](http://upload-images.jianshu.io/upload_images/1229762-08b8a4db16d8781e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


您可以像其他断点一样，禁用或启用此断点。用户级别断点在所有的Xcode工程中都可以使用。
7.编译程序 在Reveal中选择对应应用 届时已经能看到层次分明的应用啦

![](http://upload-images.jianshu.io/upload_images/1229762-665253083b030877.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在Reveal 中显示应用
最终在模拟器中如下图显示

![](http://upload-images.jianshu.io/upload_images/1229762-042700036759d139.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终显示
具体源码及SQExtension方法信息 请到[github](https://github.com/coderZsq/coderZsq.project.oc)上进行下载!