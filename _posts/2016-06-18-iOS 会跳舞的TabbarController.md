---
layout: post
title: iOS 会跳舞的TabbarController
date: 2016.06.18 14:19
tag: 移动开发
---

> 上周的<投机流实现 无限轮播图> 收到大家一致的好评和转载, 小弟心中感到很是欣慰, 所以今天再此决定分享一个干货给各位小伙伴们!!

###### 参考链接:

- [iOS 做好开工前的准备](http://www.jianshu.com/p/a3e1b54c73d6)
- [iOS 集成Reveal UI调试利器](http://www.jianshu.com/p/861c9c916b2a)
- [UI/UX 产品原型 从Axure开始](http://www.jianshu.com/p/440bdc425c02)
- [UI/UX 使用Ps 进行视觉设计](http://www.jianshu.com/p/56eb4917f956)
- [iOS 投机流实现 无限轮播图](http://www.jianshu.com/p/e42db267d5f1)

以下内容在上述文章基础上进行, 请事先查阅.

具体是什么干货呢, 今天要分享的是会跳舞的TabbarController, 什么是会跳舞的TabbarController? 很简单,就是使用自定义转场给Tabbar切换加上动画!! 鄙人会在能力范围内尽力的用最简练的语句和精简的方法给大家解释清楚!!

##### 自定义转场
那什么是自定义转场呢, iOS7中的视图控制器切换分为两种, 自定义切换动画和交互式切换动画, 蒙圈的小伙伴别着急,且往下看:

在iOS中 控制器转场分为 Push/Pop流 ,Modal流(present/dismiss流), 但是最会被大家忽视的就是TabbarController的切换, Tabbar的切换也能做动画吗?? 是的他可以!! 话不多说, 直接进入我们的项目中来!!

##### 实现方法
首先我们先进入`SQTabBarController.m`这是之前在我SQExtension中封装的框架的一部分, 找不到的同学可以通过(`CMD+shift+O`进行搜索) 其实自定义转场就是通过调用系统的代理方法实现的! 设置代理三步走 走起~~

###### 设置代理
```
- (void)tabBar:(SQTabBar *)tabBar didSelectItemFrom:(NSInteger)preIndex to:(NSInteger)selectedIndex {

    self.selectedIndex = selectedIndex;
    self.delegate = self;
}
```
这里有一个不能叫天坑,简直能被称为神坑的坑!! 我使用最新Xcode7时, 如果在TabbarController实例化后马上设置代理为self, 那个时候的代理将会是nil,
届时代理方法就不会响应!! 而最坑的是,当我将开发环境切换到Xcode5的时候, 相同的代码居然连TabbarController得子控制器都不能进行渲染(不走viewDidLoad等生命周期方法) 

`经过一段时间的调试,只有将代理设在响应代理方法(如 pushViewController:/presentViewController:/selectedIndex)之后 才不会出现上述问题!! 在此深深的为iOS底层API着了一把小急…`


###### 遵守协议
```
@interface SQTabBarController () <SQTabBarDelegate,UITabBarControllerDelegate>
```

###### 实现代理方法
```
- (id <UIViewControllerAnimatedTransitioning>)tabBarController:(UITabBarController *)tabBarController animationControllerForTransitionFromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC {
    return [NSObject new];
}
```
在这里需要返回的是id <UIViewControllerAnimatedTransitioning>也就是遵守UIViewControllerAnimatedTransitioning协议的任意对象, 我们为了让程序正常运行 先返回一个NSObject.

###### 实现id <UIViewControllerAnimatedTransitioning> 对象
这个就是自定义转场的关键, 正如UICollectionViewLayout一样重要!! 我们先创建一个继承与NSObject的类,我将其取名SQTabbarControllerAnimatedTransitioning并遵守协议.

```
@interface SQTabbarControllerAnimatedTransitioning : NSObject <UIViewControllerAnimatedTransitioning>
```

这里需要实现两个方法

`转换效果持续时间`
- (NSTimeInterval)transitionDuration:(id<UIViewControllerContextTransitioning>)transitionContext;

`转换效果的定义`
- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext;


```
static CGFloat const kPadding  = 10;
static CGFloat const kDamping  = 0.75;
static CGFloat const kVelocity = 2;

- (NSTimeInterval)transitionDuration:(id<UIViewControllerContextTransitioning>)transitionContext {
    return 0.75f;
}

- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
    
    UIViewController * toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    UIViewController * fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    UIView * containerView = [transitionContext containerView];
    
    CGFloat translation = containerView.bounds.size.width + kPadding;
    CGAffineTransform transform = CGAffineTransformMakeTranslation (translation, 0);
    toViewController.view.transform = CGAffineTransformInvert (transform);
    [containerView addSubview:toViewController.view];
    
    [UIView animateWithDuration:[self transitionDuration:transitionContext] delay:0 usingSpringWithDamping:kDamping initialSpringVelocity:kVelocity options:UIViewAnimationOptionCurveEaseInOut animations:^{
        fromViewController.view.transform = transform;
        toViewController.view.transform = CGAffineTransformIdentity;
    } completion:^(BOOL finished) {
        fromViewController.view.transform = CGAffineTransformIdentity;
        [transitionContext completeTransition:![transitionContext transitionWasCancelled]];
    }];
}
```
我们来分析一下这串代码, 这个代理方法传了一个(id<UIViewControllerContextTransitioning>)transitionContext, 我相信使用过Quartz2D的同学们对上下文应该不会太陌生. 我先说说这个类的必须了解的三个方法:

`viewController切换所发生的view容器,开发者应该将切出的view移除, 将切入的view加到该view容器中`

- (UIView *)containerView; 

`提供一个key返回对应的viewController`

- (UIViewController *)viewControllerForKey:(NSString *)key;

`向这个上下文报告切换已经完成`

- (void)completeTransition:(BOOL)didComplete;

`这三个方法必须实现 不然将不会进行转场!! 谨记!!`

实现了上述方法 我们来自定义动画, 这里我使用的是UIView的弹簧动画, 这个不多讲,大家随便设值就能了解个大概.

最后我们将这个类返回给代理方法

```
- (id <UIViewControllerAnimatedTransitioning>)tabBarController:(UITabBarController *)tabBarController animationControllerForTransitionFromViewController:(UIViewController *)fromVC toViewController:(UIViewController *)toVC {
    return [SQTabbarControllerAnimatedTransitioning new];
}
```

##### 解决冲突

来 我们运行一下程序, 发现切换tabbar的时候已经有过度动画了, 是不是很神奇!! 不过细心的同学已经发现了, 当进行自定义动画的切换的时候会和之前的轮播图产生冲突而引起Bug, 来, 我们先来解决这个Bug.

来到我们`SQLifestyleBannerCell.m`替换成collectionView的动画效果即可!

```
- (void)updateTimer {
//    [self.collectionView setContentOffset:CGPointMake(self.collectionView.contentOffset.x + self.contentView.frame.size.width, 0) animated:YES];

    NSInteger item = (self.collectionView.contentOffset.x + self.contentView.frame.size.width) * (kCounts * kMultiply)/ self.collectionView.contentSize.width;
    [self.collectionView scrollToItemAtIndexPath:[NSIndexPath indexPathForItem:item inSection:0] atScrollPosition:UICollectionViewScrollPositionCenteredHorizontally animated:YES];
}
```

##### 投机流 改善用户体验

现在我们发现现在的转场动画都是往一边的, 感觉用户体验不好, 能不能实现我切换的时候能够识别tabbarButton的顺序呢? 我看了下喵神的博客, 发现他是实现了自定义的上下文然后使用上下文的以下方法:

```
// The frame's are set to CGRectZero when they are not known or
// otherwise undefined.  For example the finalFrame of the
// fromViewController will be CGRectZero if and only if the fromView will be
// removed from the window at the end of the transition. On the other
// hand, if the finalFrame is not CGRectZero then it must be respected
// at the end of the transition.
- (CGRect)initialFrameForViewController:(UIViewController *)vc;
- (CGRect)finalFrameForViewController:(UIViewController *)vc;
```

但是正如我上篇文章所述, 能够投机的就不用搞得那么复杂嘛…

来, 我们先创建个全局类来记录对应的Index下标.

.h
```
extern NSInteger _preIndex;
extern NSInteger _selectedIndex;
```

.m
```
NSInteger _preIndex = 0;
NSInteger _selectedIndex = 0;
```

在`SQTabBarController.m`中记录下标

```
- (void)tabBar:(SQTabBar *)tabBar didSelectItemFrom:(NSInteger)preIndex to:(NSInteger)selectedIndex {

                         _preIndex      = preIndex;
    self.selectedIndex = _selectedIndex = selectedIndex;
    self.delegate      = self;
}
```
在自定义转场类中做判断

```
- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
    
    UIViewController * toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    UIViewController * fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    UIView * containerView = [transitionContext containerView];
    
    CGFloat translation = containerView.bounds.size.width + kPadding;
    CGAffineTransform transform = CGAffineTransformMakeTranslation ((_preIndex > _selectedIndex ? YES : NO) ? translation : -translation, 0);
    toViewController.view.transform = CGAffineTransformInvert (transform);
    [containerView addSubview:toViewController.view];
    
    [UIView animateWithDuration:[self transitionDuration:transitionContext] delay:0 usingSpringWithDamping:kDamping initialSpringVelocity:kVelocity options:UIViewAnimationOptionCurveEaseInOut animations:^{
        fromViewController.view.transform = transform;
        toViewController.view.transform = CGAffineTransformIdentity;
    } completion:^(BOOL finished) {
        fromViewController.view.transform = CGAffineTransformIdentity;
        [transitionContext completeTransition:![transitionContext transitionWasCancelled]];
}
   
```
这样就大功告成了!! 这个自定义转场类不仅可以为Tabbar我转成, 如本文之前所说, 任何控制器转场都可以使用!!欲知Push/Pop流 ,Modal流(present/dismiss流)以及交互式切换, 后期有机会会为大家讲解!!

来我们看下实现的效果:
![](http://upload-images.jianshu.io/upload_images/1229762-53e832042f558ff7.gif?imageMogr2/auto-orient/strip)

具体源码及SQExtension方法信息 请到[github](https://github.com/coderZsq)上进行下载! 喜欢的朋友请点一下左上角的小星星哦!!
