---
layout: post
title: Animations 快速上手 iOS10 属性动画
date: 2017.05.03 11:04
tag: 移动开发
---

> 基础动画, 核心动画到自定义转场动画其实都不是什么新东西了, 所以我也是草草看一遍就能够读个大概, 但今天要说的UIViewPropertyAnimator, 是iOS10新的API, 其他的好处我还不太清楚, 但抽象动画逻辑和监控动画的进程上真的是方便很多.代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-8f8d322fd1b24ddb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于属性动画来说, 真的是个新知识, 想必会用的还不多, 我也是近期才有涉及, 理解不周还望大伙指点一二, 之前我们要做一些稍微高级的动画都会使用核心动画的方法, 但是核心动画有一个致命的弱点, 就是假象和被打断, 所以iOS10出了新的API真的是造福我们这些开发者. 不多说, 先了解新的API:

```
@available(iOS 10.0, *)
open class UIViewPropertyAnimator : NSObject, UIViewImplicitlyAnimating, NSCopying
  @NSCopying open var timingParameters: UITimingCurveProvider? { get }
  open var duration: TimeInterval { get }
  open var delay: TimeInterval { get }
  open var isUserInteractionEnabled: Bool
  open var isManualHitTestingEnabled: Bool
  open var isInterruptible: Bool
```
- timingParameters: 时间参数
- duration: 持续时间
- delay: 延迟时间
- isUserInteractionEnabled: 是否可交互
- isManualHitTestingEnabled: 是否启动手动测试
- isInterruptible: 是否可被打断

##### 构造方法:
```
public init(duration: TimeInterval, timingParameters parameters: UITimingCurveProvider)
```
- duration: 持续时间
- timingParameters: 时间参数

```
public convenience init(duration: TimeInterval, curve: UIViewAnimationCurve, animations: (@escaping () -> Swift.Void)? = nil)
```
- duration: 持续时间
- curve: 动画曲线
- animations: 动画执行闭包

```
public convenience init(duration: TimeInterval, controlPoint1 point1: CGPoint, controlPoint2 point2: CGPoint, animations: (@escaping () -> Swift.Void)? = nil)
```
- duration: 持续时间
- controlPoint1: 控制点1
- controlPoint2: 控制点2
- animations: 动画执行闭包

```
public convenience init(duration: TimeInterval, dampingRatio ratio: CGFloat, animations: (@escaping () -> Swift.Void)? = nil)
```
- duration: 持续时间
- dampingRatio: 减震比率
- animations: 动画执行闭包

##### 运行属性动画方法:
```
open class func runningPropertyAnimator(withDuration duration: TimeInterval, delay: TimeInterval, options: UIViewAnimationOptions = [], animations: @escaping () -> Swift.Void, completion: (@escaping (UIViewAnimatingPosition) -> Swift.Void)? = nil) -> Self
```
- duration: 持续时间
- options: 动画选项
- animations: 动画执行闭包
- completion: 动画完成闭包

##### 添加动画组方法:
```
open func addAnimations(_ animation: @escaping () -> Swift.Void, delayFactor: CGFloat)
```
- animation: 动画执行闭包
- delayFactor: 延迟比率, 按照百分比计算 范围0~1

##### 动画组完成方法:
```
open func addCompletion(_ completion: @escaping (UIViewAnimatingPosition) -> Swift.Void)
```
- completion: 完成执行闭包

##### 继续动画方法:
```
open func continueAnimation(withTimingParameters parameters: UITimingCurveProvider?, durationFactor: CGFloat)
```
- withTimingParameters: 时间参数
- durationFactor: 持续比率, 按照百分比计算 范围0~1

##### 动画状态:
```
@available(iOS 10.0, *)
public enum UIViewAnimatingState : Int {
    case inactive // The animation is not executing.
    case active // The animation is executing.
    case stopped // The animation has been stopped and has not transitioned to inactive.
}
```

##### 动画位置:
```
@available(iOS 10.0, *)
public enum UIViewAnimatingPosition : Int {
    case end
    case start
    case current
}
```

##### 动画协议:
```
public protocol UIViewAnimating : NSObjectProtocol
@available(iOS 10.0, *)
    public var state: UIViewAnimatingState { get }
    public var isRunning: Bool { get }
    public var isReversed: Bool { get set }
    public var fractionComplete: CGFloat { get set }
    public func startAnimation()
    public func startAnimation(afterDelay delay: TimeInterval)
    public func pauseAnimation()
    public func stopAnimation(_ withoutFinishing: Bool)
@available(iOS 10.0, *)
    public func finishAnimation(at finalPosition: UIViewAnimatingPosition)
```
- state: 动画状态
- isRunning: 动画是否在执行
- isReversed: 动画是否反向执行
- fractionComplete: 分段完成 范围0~1
- startAnimation: 开始执行动画
- startAnimation(afterDelay:): 延迟执行动画
- pauseAnimation: 暂停动画执行
- finishAnimation: 完成动画知悉在某个动画位置

##### 实战实例:

这次的API有点多, 因为本人也是第一次接触所以把每个都尝试了一遍, 我们就来使用这些API进行实战, 和之前一样, 我们就通过Stroyboard搭建界面 (由于Demo代码较多, 只列举具体的动画部分)

![](http://upload-images.jianshu.io/upload_images/1229762-601e06af1266b6c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们首先先将动画抽象到一个类中:
```
class AnimatorFactory {
    ...
}
```

添加缩放动画:
```
  static func scaleUp(view: UIView) -> UIViewPropertyAnimator {
    let scale = UIViewPropertyAnimator(duration: 0.33, curve: .easeIn) //属性动画初始化
    scale.addAnimations { //添加动画组
      view.alpha = 1.0
    }
    scale.addAnimations({ //添加动画组
      view.transform = .identity
    }, delayFactor: 0.33) //延迟33%执行
    scale.addCompletion {_ in //完成后打印
      print("ready")
    }
    return scale
  }
```
添加摇晃动画:
```
  @discardableResult //去除黄色警告
  static func jiggle(view: UIView) -> UIViewPropertyAnimator {
    return UIViewPropertyAnimator.runningPropertyAnimator(withDuration: 0.33, delay: 0, //直接执行属性动画
      animations: {

        UIView.animateKeyframes(withDuration: 1, delay: 0, //关键帧动画
          animations: {

            UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.25) {
              view.transform = CGAffineTransform(rotationAngle: -.pi/8)
            }
            UIView.addKeyframe(withRelativeStartTime: 0.25, relativeDuration: 0.75) {
              view.transform = CGAffineTransform(rotationAngle: +.pi/8)
            }
            UIView.addKeyframe(withRelativeStartTime: 0.75, relativeDuration: 1.0) {
              view.transform = .identity
            }
          },
          completion: nil
        )
      },
      completion: {_ in
        view.transform = .identity //完成后置空
      }
    )
  }

```

添加淡入淡出动画:
```
  @discardableResult
  static func fade(view: UIView, visible: Bool) -> UIViewPropertyAnimator {
    return UIViewPropertyAnimator.runningPropertyAnimator(withDuration: 0.5, delay: 0.1,
      options: [.curveEaseOut], 
      animations: {
        view.alpha = visible ? 1 : 0
      }
    )
  }
```
添加约束动画:
```
  @discardableResult
  static func animateConstraint(view: UIView, constraint: NSLayoutConstraint, by: CGFloat) -> UIViewPropertyAnimator {
    let spring = UISpringTimingParameters(dampingRatio: 0.55) //这个和之前的弹簧动画类似
    let animator = UIViewPropertyAnimator(duration: 1.0, timingParameters: spring)
    animator.addAnimations {
      constraint.constant += by //约束操作
      view.layoutIfNeeded() //刷帧
    }
    return animator
  }
```
添加闪烁动画:
```
  static func grow(view: UIVisualEffectView, blurView: UIVisualEffectView) -> UIViewPropertyAnimator {
    // 1
    view.contentView.alpha = 0
    view.transform = .identity

    // 2
    let animator = UIViewPropertyAnimator(duration: 0.5, curve: .easeIn)
    
    // 3
    animator.addAnimations {
      UIView.animateKeyframes(withDuration: 0.5, delay: 0.0, animations: {

        UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 1.0) {
          blurView.effect = UIBlurEffect(style: .dark)
          view.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
        }

        UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.5) {
          view.transform = view.transform.rotated(by: -.pi/8)
        }
      })
    }
    
    // 4
    animator.addCompletion {position in
      switch position {
        case .start:
          blurView.effect = nil
        case .end:
          blurView.effect = UIBlurEffect(style: .dark)
        default: break
      }
    }
    return animator
  }
```
进行属性动画的调用;

```
@IBAction func toggleShowMore(_ sender: UIButton) {

    self.showsMore = !self.showsMore

    let animations = { //创建动画组闭包
      self.widgetHeight.constant = self.showsMore ? 230 : 130
      if let tableView = self.tableView {
        tableView.beginUpdates() //进行tableView刷新动画
        tableView.endUpdates()
        tableView.layoutIfNeeded() //刷布局
      }
    }

    let textTransition = { //View转场
      UIView.transition(with: sender, duration: 0.25, options: .transitionCrossDissolve,
        animations: {
          sender.setTitle(self.showsMore ? "Show Less" : "Show More", for: .normal)
        },
        completion: nil
      )
    }

    if let toggleHeightAnimator = toggleHeightAnimator, toggleHeightAnimator.isRunning { 
      toggleHeightAnimator.addAnimations(animations)
      toggleHeightAnimator.addAnimations(textTransition, delayFactor: 0.5)
    } else {
      let spring = UISpringTimingParameters(mass: 30, stiffness: 1000, damping: 300, initialVelocity: CGVector.zero)
      toggleHeightAnimator = UIViewPropertyAnimator(duration: 0.0, timingParameters: spring) //初始化动画
      toggleHeightAnimator?.addAnimations(animations) //添加动画组
      toggleHeightAnimator?.addAnimations(textTransition, delayFactor: 0.5)
      toggleHeightAnimator?.startAnimation() //开始动画
    }

    widgetView.expanded = showsMore
    widgetView.reload()
  }
```
##### 演示效果:

![](http://upload-images.jianshu.io/upload_images/1229762-5be066f9e980f220.gif?imageMogr2/auto-orient/strip)

##### About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)

[百度云源码](http://www.demodashi.com/demo/10639.html)
