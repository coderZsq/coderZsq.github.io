---
layout: post
title: Animations 自定义转场现已加入豪华午餐
date: 2017.05.02 11:23
tag: 移动开发
---

> 经过了上周的基本动画和核心动画的理解, 我们今天来讲讲自定义转场动画, 自定义转场动画是iOS中比较高阶的动画形式了, 简单来说就是在切换控制器时发生的动画形式, 可以在Push/Pop ,Modal(present/dismiss)以及tabbar切换时进行动画. 代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-8f8d322fd1b24ddb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于Modal(present/dismiss)以及tabbar切换的自定义动画可以看我之前的博客:
 [Modal  请点击 --> iOS 狂霸酷炫拽之Button动效](http://www.jianshu.com/p/6106f5a08ec3)
[Tabbar 请点击 --> iOS 会跳舞的TabbarController](http://www.jianshu.com/p/c1a0cd2a348f)

本期我们来着重说说Push/Pop里如何进行自定义转场动画, 和之前的核心动画一样, 我们还是一如既往的先讲API, 再进行实战的演练, 对于自定义转场我们要分三步走:  1) 设置代理, 2)遵守协议, 3)实现代理方法.我们先从协议开始看起:

导航控制器代理方法:
```
public protocol UINavigationControllerDelegate : NSObjectProtocol

@available(iOS 7.0, *)
    optional public func navigationController(_ navigationController: UINavigationController, interactionControllerFor animationController: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning?

@available(iOS 7.0, *)
    optional public func navigationController(_ navigationController: UINavigationController, animationControllerFor operation: UINavigationControllerOperation, from fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning?
```
- interactionControllerFor 与用户交互时调用的代理方法
- animationControllerFor  自定义转场切换时调用的代理方法

自定义转场协议:
```
public protocol UIViewControllerAnimatedTransitioning : NSObjectProtocol
    public func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval
    public func animateTransition(using transitionContext: UIViewControllerContextTransitioning)
```
- transitionDuration 转场时长
- animateTransition 转场动画如何实现

自定义转场交互协议:
```
@available(iOS 7.0, *)
open class UIPercentDrivenInteractiveTransition : NSObject, UIViewControllerInteractiveTransitioning
    open func update(_ percentComplete: CGFloat)
    open func cancel()
    open func finish()
```
- update 更新转场 百分比
- cancel 取消转场
- finish 完成转场

自定义转场类型枚举:
```
public enum UINavigationControllerOperation : Int {
    case push
    case pop
}
```
- push 压栈
- pop 出栈

自定义转场上下文:
```
public protocol UIViewControllerContextTransitioning : NSObjectProtocol
@available(iOS 2.0, *)
    public var containerView: UIView { get }
    public var transitionWasCancelled: Bool { get }
@available(iOS 2.0, *)
    public func viewController(forKey key: UITransitionContextViewControllerKey) -> UIViewController?
@available(iOS 2.0, *)
    public func initialFrame(for vc: UIViewController) -> CGRect
@available(iOS 2.0, *)
    public func finalFrame(for vc: UIViewController) -> CGRect
```
- containerView 包装View, 用户处理自定义转场交互的View
- transitionWasCancelled 取消转场的操作
- viewControllerforKey 拿到起始或目标控制器方法
- initialFrame 初始Frame
- finalFrame 最终Frame

讲完API, 我们就来使用这些API进行实战, 和之前一样, 我们就通过Stroyboard搭建界面:
![](http://upload-images.jianshu.io/upload_images/1229762-0ec1f9c441fc9f99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

跳过@IBOutlet 关联这步, 我们先来创建自定义转场对象:
```
class RevealAnimator: UIPercentDrivenInteractiveTransition, UIViewControllerAnimatedTransitioning, CAAnimationDelegate { //遵循协议

  let animationDuration = 2.0 //动画时长
  var operation: UINavigationControllerOperation = .push //转场类型
  weak var storedContext: UIViewControllerContextTransitioning? //保存转场上下文
  var interactive = false //是否可交互

  func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
    return animationDuration //返回动画时长
  }

  func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    storedContext = transitionContext //保存上下文
    ... 
  }
}
```
进行动画的自定义:
```
  func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
    storedContext = transitionContext

    if operation == .push { //判断转场类型 为push

      let fromVC = transitionContext.viewController(forKey: .from) as! MasterViewController //拿到初始控制器
      let toVC = transitionContext.viewController(forKey: .to) as! DetailViewController //拿到目标控制器

      transitionContext.containerView.addSubview(toVC.view) //将目标控制器的View添加到上下文的包装View中
      toVC.view.frame = transitionContext.finalFrame(for: toVC) //设置目标控制器View为finalFrame

      let animation = CABasicAnimation(keyPath: "transform") //进行核心动画
      animation.fromValue = NSValue(caTransform3D: CATransform3DIdentity) 
      animation.toValue = NSValue(caTransform3D:
        CATransform3DConcat( //concat与RxSwift中相同为向后拼接, 此处为叠加3D仿射动画
          CATransform3DMakeTranslation(0.0, -10.0, 0.0),
          CATransform3DMakeScale(150.0, 150.0, 1.0)
        )
      )

      animation.duration = animationDuration 
      animation.delegate = self 
      animation.fillMode = kCAFillModeForwards 
      animation.isRemovedOnCompletion = false
      animation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)
      
      let maskLayer: CAShapeLayer = RWLogoLayer.logoLayer() //maskLayer为遮罩层
      maskLayer.position = fromVC.logo.position 
      maskLayer.add(animation, forKey: nil)
      toVC.view.layer.mask = maskLayer //将遮罩层Layer添加到显示层Layer的Mask属性上
      
      fromVC.logo.add(animation, forKey: nil)

      let fadeIn = CABasicAnimation(keyPath: "opacity")
      fadeIn.fromValue = 0.0
      fadeIn.toValue = 1.0
      fadeIn.duration = animationDuration
      toVC.view.layer.add(fadeIn, forKey: nil)
    } else {

      let fromView = transitionContext.view(forKey: .from)!
      let toView = transitionContext.view(forKey: .to)!

      transitionContext.containerView.insertSubview(toView, belowSubview: fromView)

      UIView.animate(withDuration: animationDuration, delay: 0.0, options: .curveEaseIn,
        animations: {
          fromView.transform = CGAffineTransform(scaleX: 0.01, y: 0.01)
        },
        completion: {_ in
          transitionContext.completeTransition(!transitionContext.transitionWasCancelled) //完成转场
        }
      )
    }
```

核心动画代理:
```
  func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {
    if let context = storedContext {
      context.completeTransition(!context.transitionWasCancelled) //完成转场
      //reset logo
      let fromVC = context.viewController(forKey: .from) as! MasterViewController
      fromVC.logo.removeAllAnimations() //删除所有动画
      
      let toVC = context.viewController(forKey: .to) as! DetailViewController
      toVC.view.layer.mask = nil //动画停止 清空遮罩层
    }
    storedContext = nil
  }
```

pan手势转场交互:
```
  func handlePan(recognizer: UIPanGestureRecognizer) {
    let translation = recognizer.translation(in: recognizer.view!.superview!) //拿到转换坐标
    var progress: CGFloat = abs(translation.x / 200.0) //拿到x轴进度
    progress = min(max(progress, 0.01), 0.99) //给定进度边界

    switch recognizer.state { //控制手势状态
    case .changed: //改变时
      update(progress) //更新进度
    case .cancelled, .ended: //取消或结束时
      let transitionLayer = storedContext!.containerView.layer //拿到包装Layer层
      transitionLayer.beginTime = CACurrentMediaTime()
      if progress < 0.5 { //控制进度
        cancel() //取消转场
        transitionLayer.speed = -1.0 //进行隐式动画
      } else {
        transitionLayer.speed = 1.0 //进行隐式动画
        finish() //完成转场
      }
      interactive = false //交互设置为不可交互
    default:
      break
    }
  }
```
完成自定义转场类后, 我们进行三步走: 
```
1)  设置代理:
class MasterViewController: UIViewController {
  
  let logo = RWLogoLayer.logoLayer() 
  let transition = RevealAnimator() //自定义转场类

  override func viewDidLoad() {
    super.viewDidLoad()
    title = "Start"
    navigationController?.delegate = self //设置代理
  }
}
```
```
2) 遵守协议并实现代理方法:
extension MasterViewController: UINavigationControllerDelegate {
  func navigationController(_ navigationController: UINavigationController, animationControllerFor operation: UINavigationControllerOperation, from fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
    transition.operation = operation
    return transition
  }

  func navigationController(_ navigationController: UINavigationController, interactionControllerFor animationController: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
    if !transition.interactive { //当上下文不可交互时返回空
      return nil
    }
    return transition
  }
}
```
```
3) 添加手势:
  override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    
    let pan = UIPanGestureRecognizer(target: self, action: #selector(didPan))
    view.addGestureRecognizer(pan)

    // add the logo to the view
    logo.position = CGPoint(x: view.layer.bounds.size.width/2,
      y: view.layer.bounds.size.height/2 - 30)
    logo.fillColor = UIColor.white.cgColor
    view.layer.addSublayer(logo)
  }

  func didPan(recognizer: UIPanGestureRecognizer) {
    switch recognizer.state {
    case .began:
      transition.interactive = true //设置转场可交互
      performSegue(withIdentifier: "details", sender: nil)
    default:
      transition.handlePan(recognizer: recognizer) //调用自定义转场类中的手势操作更新转场

    }
  }
```
演示效果:
![](http://upload-images.jianshu.io/upload_images/1229762-f2c856c0d61b0117.gif?imageMogr2/auto-orient/strip)
About: 

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
