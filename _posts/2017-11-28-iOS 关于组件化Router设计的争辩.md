---
layout: post
title: iOS 关于组件化Router设计的争辩
date: 2017.11.28 20:16
tag: 移动开发
---

> 本文记录了与一位同学关于Router设计的争论, 对于url router 和 protocol router 的争论, 架构并没有孰优孰劣, 只有适合与否, 希望能有更多的同学一起踊跃探讨.

![](http://upload-images.jianshu.io/upload_images/1229762-edb8e045f1b4632c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


对于组件化, 相信大家一定不陌生, 但针对组件化的方案及思路, 大家或多或少都有一些自己的想法, 如果不清楚组件化的同学可以先通过以下文章预习一下.

##### 参考阅读:

[alibaba/BeeHive](https://github.com/alibaba/BeeHive)

[蘑菇街 App 的组件化之路](http://limboy.me/tech/2016/03/10/mgj-components.html)

[蘑菇街 App 的组件化之路·续](http://limboy.me/tech/2016/03/14/mgj-components-continued.html)

[iOS应用架构谈 组件化方案](https://casatwy.com/iOS-Modulization.html)

[iOS组件化方案](http://mrpeak.cn/blog/module/)

[组件化架构漫谈](http://www.jianshu.com/p/67a6004f6930)

[iOS 组件化 —— 路由设计思路分析](http://www.jianshu.com/p/76da56b3bd55)

[iOS VIPER架构实践(三)：面向接口的路由设计](https://juejin.im/post/59cb629c5188253cb5016322)

[Hybird 搭建路由Router实现组件化](http://www.jianshu.com/p/36314d0c0032)

#### 起源

一切源于我将 -- 架构初探系列 -- 发布到掘金上和一位大佬对于Router设计展开的激烈的辩论, 作为一个iOS初级玩家, 觉得应该将争辩的内容分享出来让大家一起探讨才能够有所提升.

以下是我对于Router的简单设计, 这个在之前的[文章](http://www.jianshu.com/p/36314d0c0032)里也有写到.


```
//
//  Router.swift
//  RouterPatterm
//
//  Created by 双泉 朱 on 17/4/12.
//  Copyright © 2017年 Doubles_Z. All rights reserved.
//

import UIKit

class Router {
    static let shareRouter = Router()
    var params: [String : Any]?
    var routers: [String : Any]?
    fileprivate let map = ["J1" : "Controller"]
    
    func guardRouters(finishedCallback : @escaping () -> ()) {
        
        Http.requestData(.get, URLString: "http://localhost:3001/api/J1/getRouters") { (response) in
            guard let result = response as? [String : Any] else { return }
            guard let data:[String : Any] = result["data"] as? [String : Any] else { return }
            guard let routers:[String : Any] = data["routers"] as? [String : Any] else { return }
            self.routers = routers
            finishedCallback()
        }
    }
}

extension Router {
    
    func addParam(key: String, value: Any) {
        params?[key] = value
    }
    
    func clearParams() {
        params?.removeAll()
    }
    
    func push(_ path: String) {
        
        guardRouters {
            guard let state = self.routers?[path] as? String else { return }
            
            if state == "app" {
                guard let nativeController = NSClassFromString("RouterPatterm.\(self.map[path]!)") as? UIViewController.Type else { return }
                currentController?.navigationController?.pushViewController(nativeController.init(), animated: true)
            }
            
            if state == "web" {
                
                let host = "http://localhost:3000/"
                var query = ""
                let ref = "client=app"
                
                guard let params = self.params else { return }
                for (key, value) in params {
                    query += "\(key)=\(value)&"
                }
                
                self.clearParams()
                
                let webViewController = WebViewController("\(host)\(path)?\(query)\(ref)")
                currentController?.navigationController?.pushViewController(webViewController, animated: true)
            }
        }
    }
}

```

使用的时候只需要如下操作:

```
Router.shareRouter.params = [
    "text" : "app端 传入数据",
    "code" : 1001
]
Router.shareRouter.push("J1")
```
```
override func viewDidLoad() {
    super.viewDidLoad()
	self.text = Router.shareRouter.params["text"]
}
```
以上就是我理解router的本质, 最为精简的功能, 可选添加参数, AOP清除参数, 跳转. 当然要路由是要根据业务来设计的, 这里只是实现一个最基础的功能.

接着就有一位同学[@黑超熊猫zuik](http://www.jianshu.com/u/4c88d7c03e0b)在我的[文章](https://juejin.im/post/5a183f38f265da432528fefc)后进行了评论, [@黑超熊猫zuik](http://www.jianshu.com/u/4c88d7c03e0b)是 [iOS Developer](http://www.jianshu.com/c/3233d1a249ca)专栏的管理员(有30k+关注), 之后我们就router的 map 和 protocol 展开了交流, 以下就是交流的过程.

#### 辩论

顽主范儿

请教一下，Router的push方法怎么往目标viewController中传递参数


Castie1

回复顽主范儿：我这个router设计的比较简单, 作为一个单例持有一个参数字典, 如果需要传递参数的时候调用 addParam, 因为并不是所有跳转都需要传参数的, 使用aop在目的页面hook生命周期删除参数字典就可以了. 当然router如何设计很大程度与公司项目制定的需求及规范有关, 所以这块没有办法脱离业务来谈, 只能描述下本质.

顽主范儿

回复Castie1：我们之前也考虑过组件化，但是考虑到这个map还有整个架构都依赖于字符串，记得还有其他一些局限性，看了链家蘑菇街等分享的方案，依然没有解决这些问题，所以最后放弃了ᕙ(⇀‸↼‵‵)ᕗ

Castie1

回复顽主范儿：依赖于字符串并不是什么坏事, 因为字符串错误的问题在开发环境上就会解决了, 使用extern也可以减少这种事情的发生,  https://casatwy.com/iOS-Modulization.html 这篇文章的思想就是完全的去模型化, 可能比较激进, 但供你参考.

顽主范儿
回复Castie1：多谢，之前也看过这篇文章，由于我们没有服务器配置router的需求，只是单纯的想把各个大模块解耦，这种情况下考虑到组件化带来的利并没有特别大于弊，所以还是没有采用，再次感谢😁

黑超熊猫zuik

回复顽主范儿：这篇文章探讨的更多的是Hybird架构。如果想要更加原生的路由方式，可以参考阿里的BeeHive，里面有一个用原生的protocol寻找模块的方法，不过实现得很简单。更彻底的方式可以参考我的这篇文章：https://juejin.im/post/59cb629c5188253cb5016322

Castie1

回复黑超熊猫zuik：使用protocol寻址的方法是不是意味着对于每个vc都需要维护一套自己的参数接口并通过多态的方式跳转, 那这个和直接hook头文件的参数列表并赋值的好处在哪, 把map替换成protocol映射这一步还是逃不掉, 对于动态的h5活动页以及降级方案和urlscheme的跳转都需要额外配置. 对于Router这块还是要看业务场景 以上个人感觉.

黑超熊猫zuik

回复Castie1：的确是先看业务场景，Hybird app用URL router更简单直接，原生app用protocol router更安全高效，两种router在我的文章里也有比较。
我的protocol router实现没有用hook，是用了抽象工厂模式，每个需要路由的模块创建对应的router子类，注册可用于路由的protocol。使用时用protocol获取router子类，再执行路由获取模块，可以用注册过的protocol向模块传参和调用方法。
好处是：

1. 开发和重构时可以让编译器检查错误，充分保证安全。让开发者明确地知道所使用的模块有哪些接口，在接口改变时修改也更高效。

2. 实现路由时，无需要求模块做出修改，只需要模块实现了所注册的protocol即可，因此所映射的模块也可以随时被另一个实现了相同protocol的模块替换，解耦能力更好。

3. URL router更适合界面模块，protocol router则适用于所有功能模块。

4. 可以在router子类里进行更多自定义操作，为模块注入依赖。
对于页面降级，其实都可以通过修改映射来实现，两者差不多。

缺点是：

1. URL scheme在protocol router里需要额外支持。不过从职责上来讲这本来就应该是一个独立于router的功能。

2. protocol router在统一多端路由规则时会遇到麻烦。这时候还是需要再封装一层URL router，把PC、iOS、Android的路由表都统一起来。

Castie1

回复黑超熊猫zuik：0. 很高兴进行技术交流, 对您的protocal的思想也没有任何攻击的意思, 可以看到我这篇文章Router这块仅占很小一部分, 而其他大部分也是根据protocal进行设计的, 所以对于protocal抽象的好处你我是有共识的.

但针对Router这块, 对于原生的跳转基于protocal还有一点小小的疑问.

1. 首先明确一下名词定位, 我所理解的router就是界面模块的router, 而您指的功能模块的router在我看来就是一个中间件的服务, middleware, 和前端中router的概念完全不一样, 参照 react-router / vue-router.

2. 对于router的设计来说, 引用您说的 "每个需要路由的模块创建对应的router子类，注册可用于路由的protocol", 在我粗浅的理解就是业务方每写一个vc的时候就需要维护一套router接口, 其实就是变相的adapter模式, 造成了额外的objc文件, 会不会延缓启动速度.

3. "当接口改变的时候修改高效", 对于单个protocal接口修改的确高效, 因为是离散型的设计, 但和map集聚在router一个文件内的接口统筹修改的扩展性并没有体现出来. 和之前讲的一样, 只不过是改变了映射方式. 而使用Router.map的好处在于可以将所有路由逻辑写在一个文件内方便统一维护.

4. "开发和重构时可以让编译器检查错误，充分保证安全。"这一点保证安全, 仅仅是保证字符串拼写错误的安全, 而不是野指针级别的安全, 而依赖于字符串并不是什么坏事, 因为字符串错误的问题在开发环境上就会解决了.

我的理解:

我对于router的本质理解并不是蘑菇街这种复杂的设计, 而仅仅作为一个单例, 持有map映射, 有加参数和清参数的方法以及跳转逻辑, 我觉得这样已经够了, 这样对于业务方写vc的时候只需要对路由添加参数就够了然后跳转就够了, 而map和参数均可以服务器动态配置, 增加其扩展性和活动性, 此处参照vue-router的设计.


一点疑问: 

看了您的github的demo的用法使用了很多的block回调, 会不会加剧内存占用而不能提前释放的问题. 如果使用delegate的设计会不会减少内存占用, 有没有对比测试过?

黑超熊猫zuik

回复Castie1：嗯嗯，我是看到这条评论提到了这方面所以讨论一下，很高兴博主这么认真地互相探讨。
关于你的疑问：
1. 由于是protocol router，因此也就自然地实现了任意模块获取的功能，作用就是获取一个实现了某个指定接口的模块，对于界面路由只是再封装了一层界面跳转的方法。我对前端了解有限，不知道前端是怎么实现这种模块解耦的。

2. router子类会导致类的数量增多，这个是正常的。这个router是在实践VIPER的时候开发的，VIPER里为每一个界面模块额外创建了许多类，各部分都遵守单一职责，而相对于类的数量，我认为解耦能力在大型app里更加重要。类数量过多的确会影响objc的启动速度，不过影响其实是很有限的。微信这样的app里，view controller也只有6百多个，而再多6百个router类并不会对性能产生影响。对于一些简单的不需要传参的界面，也可以全部放到同一个router子类里，动态地返回不同的界面。

3. 为router添加统一接口。router子类在使用时是不可见的，我的实现里维护的是一个protocol和router子类的映射，通过protocol查找到对应的router子类，而使用时调用的都是抽象router父类上的统一接口。如果要为所有router添加统一的接口，可以在router父类上添加。

4. 编译检查不仅仅是为了避免字符串错误，更重要的是为了更安全地传递参数，避免字典传参。接口改变时，参数名字和类型都有可能改变，而字典传参会导致无法检测参数的正确性，可能会传入错误的参数类型。有些模块会有一些必须传入的参数，也需要从接口上体现出来。URL router这时候只能查找文档。从解耦上来看，字典传参需要模块主动解析收到的参数，调用者也需要从文档中查找可以传递哪些参数，字符串名字在字典传参时会造成耦合，当需要替换成另一个相同模块，而这个模块的参数key名字不同时，重构就很麻烦了。因此在大部分时候应该尽可能避免字典传参。

5. block的问题。在使用时不会导致内存问题，因为在执行路由时都是立即同步地调用block的。在这里使用block比delegate更合适，因为每一次调用都是和一个protocol相对应的，可以在block直接使用protocol，而使用delegate就又需要很多额外判断，使用起来会很麻烦。

Castie1

回复黑超熊猫zuik：感谢大佬详细的解答, 花了点时间拜读了下你的源码.

的确protocol router的方案的确对native端更加的友好, 使得native的组件间传值更为清晰, 

1.对于您"接口改变时，参数名字和类型都有可能改变，而字典传参会导致无法检测参数的正确性，可能会传入错误的参数类型",

```
Router.shareRouter.params = [
    "text" : "app端 传入数据",
    "code" : 1001
]
Router.shareRouter.push("J1")
```

的确, 以字典形式传参是做不到对类型的控制, 这点的确是弊端, 但这仅仅是开发阶段就能够发现的错误, 有点开发经验的人基本都会看文档, 这其实并不会造成多少开销.

所以这一点上我已经接受protocol形式的路由, 其实map其实也只是针对native的, 设计层面, protocol router和url router 应该是并行的关系, 而 urlscheme是一个额外的模块, 在这一点上已经达成了共识.

2.而对于"而这个模块的参数key名字不同时，重构就很麻烦了。因此在大部分时候应该尽可能避免字典传参" 

```
#define ZIKInfoViewProtocol_routable @protocol(ZIKInfoViewProtocol)
@protocol ZIKInfoViewProtocol <ZIKViewRoutable>
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, weak) id<ZIKInfoViewDelegate> delegate;
@end

self.infoViewRouter = [[ZIKViewRouter.toView(ZIKInfoViewProtocol_routable) alloc]
                           initWithConfiguring:^(ZIKViewRouteConfiguration * _Nonnull config) {
                               config.source = self;
                               config.routeType = ZIKViewRouteTypePush;
                               
                               //prepareDestination is hold in configuration, should be careful about retain cycle if this view controller will hold the router. Same with routeCompletion, successHandler, errorHandler, stateNotifier.
                               config.prepareDestination = ^(UIViewController<ZIKInfoViewProtocol> *destination) {
                                   NSLog(@"provider: prepare destination");
                                   destination.name = @"Zuik";
                                   destination.age = 18;
                                   destination.delegate = weakSelf;
                               };

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    
    self.nameLabel.text = self.name;
    self.ageLabel.text = [@(self.age) stringValue];
}
```

如果需要改变参数或类型, 及需要重构的情况下, 按您的demo所示, 也是需要修改至少三处, 所谓的解决重构麻烦的问题并没有实质性的帮助.

3.当使用cocoapod-sepc进行组件化的时候各个组件的粒度是如何分配, 如果是以页面为基础进行的划分, 在当前页和目的页都需要依赖protocol, 那是否意味着所有的protocol分为独立一个组件进行依赖, 但这样显而易见是有问题的, 请告知实际该如何划分.

4.对于业务方需要学习router之间如何配置, 由于您的框架比较复杂, 对于不纯熟的业务方的使用可能并不是那么友好, 增加学习成本, 而有可能对于框架的认知不到位, 是否会引起其他的问题?  

```
override func viewDidLoad() {
    super.viewDidLoad()
	self.text = Router.shareRouter.params["text"]
}

```

对于这点, 使用我这个简单demo的思路, 业务方直接就可以在viewdidload的方法直接获取需要的参数, 根本不用暴露头文件, 学习成本非常低. 由于页面跳转都是主线程, 也不会有线程安全的问题, 而且重构改动并没您所说的麻烦.

您的源码中有好多处理异常的操作, 其实并不是很理解为什么一个Router需要设计的那么复杂, 可能是我考虑的不够全面, 请大佬解惑.

黑超熊猫zuik

回复Castie1：多谢大佬看得这么仔细。重构的时候肯定是需要修改代码的，但是protocol调用依赖的是编译器检查，在修改时可以百分之百地保证所有传递的参数都是正确的。如果是字典传参，需要人工检查几十上百处参数类型时，是无法保证没有遗漏的。这个会影响开发效率，虽然修改量是相同的，但是花费的时间会相差很多。
protocol还用于描述这个模块的功能，直接在头文件里描述，无需查找文档，也是为了提升效率，并且这也是模块间解耦必须用到的。

如果要用cocoapods封装一些通用的界面组件，是可以不依赖于某一个固定的protocol的，可以让多个protocol指向同一个router子类。组件提供view controller和router子类，router子类注册一个provided protocol。在使用这个组件的app里，用另一个接口类似的required protocol获取这个组件，而app只需要再把required protocol和对应的router子类进行映射即可。
在这里，required protocol是使用者需要用到的模块接口，provided protocol是模块实际提供的接口。如果provided protocol和required protocol是一样的，那么可以无缝兼容使用，如果接口提供的功能是相同的，但是接口命名有差异，可以用扩展、NSProxy等方式让组件支持required protocol，也可以让required protocol指向一个新的router子类，而这个router子类负责提供一个实现了required protocol的中介者，把required protocol的调用转为provided protocol。
这部分接口兼容的工作是有耦合的，由app context负责，虽然是在同一个app内，但是调用者对此是不知情的，调用者可能也是另一个独立的组件。这样就可以实现不依赖于固定的protocol，从而完成彻底的模块间解耦。因此，如果要实现接口兼容，protocol是必须的。
关于这部分可以看看我的VIPER的实践：https://juejin.im/post/599a43035188252432172045，以及这个demo：https://github.com/Zuikyo/ZIKViper

关于工具的学习问题，这个工具的复杂度是一步步上升的，只是简单地为某个模块添加路由支持时，只需要三步：1. 创建子类 2.在子类里注册模块和protocol 3.在子类里返回模块实例。这时看个demo就可以很快明白。当需要其他更复杂的功能，例如为模块添加依赖注入、接口兼容、移除路由等，可以再按需学习。想要解耦，就肯定是要有一些学习成本的。

关于使用中有可能遇到的问题，应该只有一个需要注意的地方。执行路由后会返回一个router实例，可以让使用者移除已执行的路由，比如消除已经显示的界面。如果调用者持有这个router实例，则需要注意在一些block里用weak self防止循环引用。

那些异常断言检查是为了辅助开发：1. 检查router的实现是否正确，注册的模块是否已经实现了对应的protocol等 2. 检查运行时的调用是否正确，比如是否传入了错误的protocol，这个主要是因为OC是动态语言，而在Swift里可以保证调用都是正确的 3. UIKit的界面跳转有很多情况下会失败，比如在一个没有navigation的界面上执行push，就会毫无反应，这时就可以记录log，并且触发断言错误。

分了两次回复，应该没有遗漏什么吧……其实最主要是字典传参需要调用者和模块都使用同一套字符串key，会导致一定的耦合，无法无缝替换另一个相同功能的模块，也很难进行接口兼容。在ViewDidLoad里用key获取参数时，又会导致模块和router工具产生耦合。如果要想多个app共享通用组件，还是尽量寻找一个更解耦的方式，如果只是在自己的app内部使用，那可以随意点，以后有需要再改。

#### 思考

的确, 对于组件化的设计思想真是各有不同, 一个人的思维都会窄化, 所以发出来大家集思广益吧.

对于第三方库我一直是持保留态度的, 原因如下, 1. 你并不完全知道三方库的具体实现, 就算你看了源码, 2. 三方库往往对于项目时过度设计的, 根本用不到那么多冗余的方法, 3. 对于app瘦身, 虽然资源占很大比例, 但是少用三方库也是一个方面. 4. 了解一个东西的本质, 自己造个轮子会学到更多的东西, 你说呢?

希望大家也能够对于Router的设计展开激烈的探讨!! 说出你的观点!!

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
