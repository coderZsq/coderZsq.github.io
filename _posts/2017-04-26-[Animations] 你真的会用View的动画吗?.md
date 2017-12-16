---
layout: post
title: Animations 你真的会用View的动画吗?
date: 2017.04.26 11:58
tag: 移动开发
---

> RxSwift的学习我们先告一段落, 实在是太烧脑了, 今天我们换换脑子, 玩点轻松的动画, 去年我连载了一系列基于本人封装的Objective-C框架[SQExtension](https://github.com/coderZsq/coderZsq.project.oc)展开的项目实战, 里面关于动画方面的涉及还是蛮多的以至于受到了腾讯的关注. 这个系列我们就来细讲动画, 一起走上进阶之路. 代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-8f8d322fd1b24ddb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于动画方面除了最基本的View动画之外, 关于Layer的核心动画我都是看这篇[wiki](http://wiki.jikexueyuan.com/project/ios-core-animation/)学的, 像去年连载的一些自定义转场动画是从[Objc](https://www.objccn.io/)中学的. 这次买了Ray家的一系列教程, 就打算将动画学透, 不再只是碎片化的知识.和连载RxSwift不同, 这次我们先带大家熟悉API以便更好的理解.

这个应该是所有做动画最常用的方法了, 大多基本都是这个方法的变体.
```
@available(iOS 4.0, *)
    open class func animate(withDuration duration: TimeInterval, delay: TimeInterval, options: UIViewAnimationOptions = [], animations: @escaping () -> Swift.Void, completion: (@escaping (Bool) -> Swift.Void)? = nil)
```
- duration 持续时间 以秒作为单位
- delay 延迟调用时间 以秒作为单位
- options 动画选项 `[], .x, [.x, .x, ...]`等格式
- animations 动画代码块, 执行可动画属性的改变
- completion 动画完成后调用的尾随闭包

这个是7.0开始使用的动画, 弹簧效果
```
@available(iOS 7.0, *)
    open class func animate(withDuration duration: TimeInterval, delay: TimeInterval, usingSpringWithDamping dampingRatio: CGFloat, initialSpringVelocity velocity: CGFloat, options: UIViewAnimationOptions = [], animations: @escaping () -> Swift.Void, completion: (@escaping (Bool) -> Swift.Void)? = nil)
```
- duration 持续时间 以秒作为单位
- delay 延迟调用时间 以秒作为单位
- usingSpringWithDamping 减震比率
- initialSpringVelocity 初始速度
- options 动画选项 `[], .x, [.x, .x, ...]`等格式
- animations 动画代码块, 执行可动画属性的改变
- completion 动画完成后调用的尾随闭包

这个是关于UIView的转场动画, 可以对不可动画属性进行转场动画
```
@available(iOS 4.0, *)
    open class func transition(with view: UIView, duration: TimeInterval, options: UIViewAnimationOptions = [], animations: (@escaping () -> Swift.Void)?, completion: (@escaping (Bool) -> Swift.Void)? = nil)
```
- view 进行转场的View
- duration 持续时间 以秒作为单位
- options 动画选项 `[], .x, [.x, .x, ...]`等格式
- animations 动画代码块, 执行可动画属性的改变
- completion 动画完成后调用的尾随闭包

这个是从一个View转场至另一个View的动画
```
@available(iOS 4.0, *)
    open class func transition(from fromView: UIView, to toView: UIView, duration: TimeInterval, options: UIViewAnimationOptions = [], completion: (@escaping (Bool) -> Swift.Void)? = nil) // toView added to fromView.superview, fromView removed from its superview
```
- from 进行转场的View
- to 转场后的View
- duration 持续时间 以秒作为单位
- options 动画选项 `[], .x, [.x, .x, ...]`等格式
- completion 动画完成后调用的尾随闭包

关键帧动画:
```
@available(iOS 7.0, *)
    open class func animateKeyframes(withDuration duration: TimeInterval, delay: TimeInterval, options: UIViewKeyframeAnimationOptions = [], animations: @escaping () -> Swift.Void, completion: (@escaping (Bool) -> Swift.Void)? = nil)
```
- duration 持续时间 以秒作为单位
- delay 延迟调用时间 以秒作为单位
- options 动画选项 `[], .x, [.x, .x, ...]`等格式
- animations 动画代码块, 执行可动画属性的改变
- completion 动画完成后调用的尾随闭包

添加关键帧:
```
@available(iOS 7.0, *)
    open class func addKeyframe(withRelativeStartTime frameStartTime: Double, relativeDuration frameDuration: Double, animations: @escaping () -> Swift.Void) // start time and duration are values between 0.0 and 1.0 specifying time and duration relative to the overall time of the keyframe animation
```
- withRelativeStartTime 相对开始时间, 取值范围0~1, 按百分比取值
- relativeDuration 相对持续时间 取值范围0~1, 按百分比取值
- animations 动画代码块, 执行可动画属性的改变

API中涉及到的`options`, `UIViewAnimationOptions`, `UIViewKeyframeAnimationOptions`中的枚举效果还是一个一个试会有更深的体会.

讲完API, 我们就来使用这些API进行实战, 和之前一样, 我们就通过Stroyboard搭建界面:

![](http://upload-images.jianshu.io/upload_images/1229762-5da65a4d4bfafb70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过IBOutlet 进行关联
```
  @IBOutlet var bgImageView: UIImageView! //背景图片
  
  @IBOutlet var summaryIcon: UIImageView! //顶部图标
  @IBOutlet var summary: UILabel! //顶部文字
  
  @IBOutlet var flightNr: UILabel! //Flight文字
  @IBOutlet var gateNr: UILabel! //Gate文字
  @IBOutlet var departingFrom: UILabel! //出发地文字
  @IBOutlet var arrivingTo: UILabel! //目的地文字
  @IBOutlet var planeImage: UIImageView! //飞机图片
  
  @IBOutlet var flightStatus: UILabel! //状态文字
  @IBOutlet var statusBanner: UIImageView! //状态图片
```

首先我们进行图片的转场切换
```
  func fadeImageView(imageView: UIImageView, toImage: UIImage, showEffects: Bool) {

    UIView.transition(with: imageView, duration: 1.0, options: .transitionCrossDissolve,
      animations: {
        imageView.image = toImage
      },
      completion: nil
    )

    UIView.animate(withDuration: 1.0, delay: 0.0, options: .curveEaseOut,
      animations: {
        self.snowView.alpha = showEffects ? 1.0 : 0.0
      },
      completion: nil
    )
  }
```
- transitionCrossDissolve options枚举中的一个 交叉过度溶解
- curveEaseOut options枚举中的一个 快进慢出 类似刹车

接着我们来做立体转场的效果, 但这个并不是真正的立体效果, 而是通过两个View的动画进行模拟
```
  func cubeTransition(label: UILabel, text: String, direction: AnimationDirection) {

    let auxLabel = UILabel(frame: label.frame) //新建一个临时Label
    auxLabel.text = text
    auxLabel.font = label.font
    auxLabel.textAlignment = label.textAlignment
    auxLabel.textColor = label.textColor
    auxLabel.backgroundColor = label.backgroundColor

    let auxLabelOffset = CGFloat(direction.rawValue) * //设置偏移
      label.frame.size.height/2.0

    auxLabel.transform = CGAffineTransform(scaleX: 1.0, y: 0.1).concatenating( //这里的concatenating 类似于RxSwift的concat, 向后叠加.
      CGAffineTransform(translationX: 0.0, y: auxLabelOffset))

    label.superview!.addSubview(auxLabel) //添加View保持和Label平级

    UIView.animate(withDuration: 0.5, delay: 0.0, options: .curveEaseOut,
      animations: {
        auxLabel.transform = .identity //还原仿射
        label.transform =
          CGAffineTransform(scaleX: 1.0, y: 0.1).concatenating( //进行仿射缩放动画
          CGAffineTransform(translationX: 0.0, y: -auxLabelOffset)) //进行位移缩放动画
      },
      completion: {_ in
        label.text = auxLabel.text
        label.transform = .identity //还原仿射

        auxLabel.removeFromSuperview() //完成后删除临时Label
      }
    )
  }
```
进行移动动画:
```
  func moveLabel(label: UILabel, text: String, offset: CGPoint) {
    let auxLabel = UILabel(frame: label.frame) //同样新建一个临时Label
    auxLabel.text = text
    auxLabel.font = label.font
    auxLabel.textAlignment = label.textAlignment
    auxLabel.textColor = label.textColor
    auxLabel.backgroundColor = UIColor.clear

    auxLabel.transform = CGAffineTransform(translationX: offset.x, y: offset.y) //临时Label进行位移
    auxLabel.alpha = 0
    view.addSubview(auxLabel)

    UIView.animate(withDuration: 0.5, delay: 0.0, options: .curveEaseIn,
      animations: {
        label.transform = CGAffineTransform(translationX: offset.x, y: offset.y)
        label.alpha = 0.0
      },
      completion: nil
    )

    UIView.animate(withDuration: 0.25, delay: 0.1, options: .curveEaseIn,
      animations: {
        auxLabel.transform = .identity
        auxLabel.alpha = 1.0
      },
      completion: {_ in
        //clean up
        auxLabel.removeFromSuperview()
        label.text = text
        label.alpha = 1.0
        label.transform = .identity
      }
    )
  }

```
- curveEaseIn options枚举中的一个 慢进快出 类似加速度

给飞机添加上关键帧动画
```
func planeDepart() {
    let originalCenter = planeImage.center //保存飞机位置

    UIView.animateKeyframes(withDuration: 1.5, delay: 0.0, //进行关键帧动画
      animations: {
        //add keyframes
        UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.25, //添加关键帧
          animations: {
            self.planeImage.center.x += 80.0 //进行位移
            self.planeImage.center.y -= 10.0
          }
        )

        UIView.addKeyframe(withRelativeStartTime: 0.1, relativeDuration: 0.4) {
          self.planeImage.transform = CGAffineTransform(rotationAngle: -.pi / 8) //进行旋转
        }

        UIView.addKeyframe(withRelativeStartTime: 0.25, relativeDuration: 0.25) {
          self.planeImage.center.x += 100.0
          self.planeImage.center.y -= 50.0
          self.planeImage.alpha = 0.0 //淡出动画
        }

        UIView.addKeyframe(withRelativeStartTime: 0.51, relativeDuration: 0.01) {
          self.planeImage.transform = .identity //重置仿射动画
          self.planeImage.center = CGPoint(x: 0.0, y: originalCenter.y) //重置位置
        }

        UIView.addKeyframe(withRelativeStartTime: 0.55, relativeDuration: 0.45) {
          self.planeImage.alpha = 1.0 //淡入动画
          self.planeImage.center = originalCenter //回到原点
        }

      },
      completion: nil
    )
  }

```
最后我们将所有动画放在一起
```
  func changeFlight(to data: FlightData, animated: Bool = false) {
    
    // populate the UI with the next flight's data
    if animated {
      planeDepart()
      summarySwitch(to: data.summary)

      fadeImageView(imageView: bgImageView,
                    toImage: UIImage(named: data.weatherImageName)!,
                    showEffects: data.showWeatherEffects)

      let direction: AnimationDirection = data.isTakingOff ?
        .positive : .negative

      cubeTransition(label: flightNr, text: data.flightNr,  direction: direction)
      cubeTransition(label: gateNr, text: data.gateNr,  direction: direction)

      let offsetDeparting = CGPoint( //设定偏移量
        x: CGFloat(direction.rawValue * 80),
        y: 0.0
      )

      moveLabel(label: departingFrom, text: data.departingFrom,
                offset: offsetDeparting)

      let offsetArriving = CGPoint( /设定偏移量
        x: 0.0,
        y: CGFloat(direction.rawValue * 50)
      )

      moveLabel(label: arrivingTo, text: data.arrivingTo,
                offset: offsetArriving)

      cubeTransition(label: flightStatus, text: data.flightStatus,  direction: direction)

    } else {
      bgImageView.image = UIImage(named: data.weatherImageName)
      snowView.isHidden = !data.showWeatherEffects
      flightNr.text = data.flightNr
      gateNr.text = data.gateNr
      departingFrom.text = data.departingFrom
      arrivingTo.text = data.arrivingTo

      flightStatus.text = data.flightStatus
      summary.text = data.summary
    }
    
    // schedule next flight
    delay(seconds: 3.0) { //3秒后进行递归调用.
      self.changeFlight(to: data.isTakingOff ? parisToRome : londonToParis, animated: true)
    }
  }
```
演示效果:

![](http://upload-images.jianshu.io/upload_images/1229762-6c5bcf96d89c86d8.gif?imageMogr2/auto-orient/strip)

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
