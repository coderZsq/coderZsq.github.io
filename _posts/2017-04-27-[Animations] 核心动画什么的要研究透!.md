---
layout: post
title: Animations 核心动画什么的要研究透!
date: 2017.04.27 10:44
tag: 移动开发
---

> 看过我去年文章的同学们一定知道对于动画方面还是有点心得体会的, 对于核心动画的学习, 之前我看的是[iOS Core Animation](http://www.amazon.com/iOS-Core-Animation-Advanced-Techniques-ebook/dp/B00EHJCORC/ref=sr_1_1?ie=UTF8&qid=1423192842&sr=8-1&keywords=Core+Animation+Advanced+Techniques)的[翻译版](http://wiki.jikexueyuan.com/project/ios-core-animation/), 看完感觉真的学到了不少东西, 不过这本书已经有点时日了, 我们需要与时俱进学习最前沿的技术.代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-8f8d322fd1b24ddb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

和之前的View的动画一样, 这次Layer的核心动画, 我们也是先细讲API, 再进行项目实战, 这样会比较容易理解, 新入行的同学不会望而却步.其实刚学习核心动画的时候还有有点难度的, 先提两个注意点: 1) 相对于`view`的`center`, 对应于`layer`的`anchorPoint`为`(0.5, 0.5)`的`position`, `layer`中的`position`和`anchorPoint`其实是同一个点.2) `layer`的动画其实是假象而且会被打断, `view`的动画是直接操作节点的. 好了, 我们来看今天所要掌握的API:

基础动画:
```
@available(iOS 2.0, *)
open class CABasicAnimation : CAPropertyAnimation 
    open var fromValue: Any?
    open var toValue: Any?
```
- fromValue 根据`keyPath`设定原始值
- toValue 根据`keyPath`设定目标值

关键帧动画:
```
@available(iOS 2.0, *)
open class CAKeyframeAnimation : CAPropertyAnimation 
    open var values: [Any]?
    open var path: CGPath?
    open var keyTimes: [NSNumber]?
    open var timingFunctions: [CAMediaTimingFunction]?
```
- values 关键帧 的值 [x, x, x, ...]
- path 关键帧路径, 贝塞尔曲线
- keyTimes 每帧的耗时, 以秒作为计量单位
- timingFunctions 缓冲函数

动画组:
```
@available(iOS 2.0, *)
open class CAAnimationGroup : CAAnimation
    open var animations: [CAAnimation]?
```
- animations 动画组数组, 将`CAAnimation`填入数组 进行组动画

弹簧动画:
```
@available(iOS 9.0, *)
open class CASpringAnimation : CABasicAnimation
    open var mass: CGFloat 
    open var stiffness: CGFloat
    open var damping: CGFloat
    open var initialVelocity: CGFloat
```
- mass 物体的质量
- stiffness 弹簧的刚度
- damping 缓冲率 也称阻尼值
- initialVelocity 初始速度

动画代理, 进行操作监听
```
public protocol CAAnimationDelegate : NSObjectProtocol
    @available(iOS 2.0, *)
    optional public func animationDidStart(_ anim: CAAnimation)
    @available(iOS 2.0, *)
    optional public func animationDidStop(_ anim: CAAnimation, finished flag: Bool)
```
- animationDidStart 动画开始前调用
- animationDidStop 动画结束后调用

CAMediaTiming 协议
```
public protocol CAMediaTiming 
    public var beginTime: CFTimeInterval { get set }
    public var duration: CFTimeInterval { get set }
    public var speed: Float { get set }
    public var repeatCount: Float { get set }
    public var autoreverses: Bool { get set }
    public var fillMode: String { get set }
```
- beginTime 动画开始时间, 对比view动画的`delay`
- duration 动画持续时间
- speed 动画速度 以倍率计算, 父层设置子层叠加
- repeatCount 重复数量
- autoreverses 自动翻转
- fillMode 填充方式, 是否显示第一帧和最后一帧

讲完API, 我们就来使用这些API进行实战, 和之前一样, 我们就通过Stroyboard搭建界面:
![](http://upload-images.jianshu.io/upload_images/1229762-e3c718347ac66709.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过 @IBOutlet 进行关联
```
    @IBOutlet var loginButton: UIButton! //登录按钮
    @IBOutlet var heading: UILabel! //顶部标签
    @IBOutlet var username: UITextField! //用户名输入框
    @IBOutlet var password: UITextField! //密码输入框
    
    @IBOutlet var cloud1: UIImageView! // 四片云
    @IBOutlet var cloud2: UIImageView!
    @IBOutlet var cloud3: UIImageView!
    @IBOutlet var cloud4: UIImageView!
```

在控制器的view将要显示的时候进行动画设置:
```
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        let formGroup = CAAnimationGroup() //创建动画组
        formGroup.duration = 0.5 //设置组动画时长为0.5秒
        formGroup.fillMode = kCAFillModeBackwards //将填充模式设置为显示最后一帧
        
        let flyRight = CABasicAnimation(keyPath: "position.x") //创建基础动画, keyPath设置为横向位移
        flyRight.fromValue = -view.bounds.size.width/2 //设置keyPath: "position.x"的起始位置
        flyRight.toValue = view.bounds.size.width/2 //设置keyPath: "position.x"的目标位置
        
        let fadeFieldIn = CABasicAnimation(keyPath: "opacity") //创建基础动画, keyPath设置为透明率
        fadeFieldIn.fromValue = 0.25 设置keyPath: "opacity"的起始值
        fadeFieldIn.toValue = 1.0 设置keyPath: "opacity"的目标值
        
        formGroup.animations = [flyRight, fadeFieldIn] //将横向位移和透明率动画加入动画组
        heading.layer.add(formGroup, forKey: nil) //将动画添加到layer层上
        
        formGroup.delegate = self //设置代理, 监听动画进程
        formGroup.setValue("form", forKey: "name") //KVC设置属性
        formGroup.setValue(username.layer, forKey: "layer") //KVC设置属性
        
        formGroup.beginTime = CACurrentMediaTime() + 0.3 //动画开始时间, 从现在+0.3秒
        username.layer.add(formGroup, forKey: nil) //将动画添加到layer层上
        
        formGroup.setValue(password.layer, forKey: "layer") //KVC设置属性
        formGroup.beginTime = CACurrentMediaTime() + 0.4 //动画开始时间, 从现在+0.4秒
        password.layer.add(formGroup, forKey: nil) //将动画添加到layer层上
        
        let fadeIn = CABasicAnimation(keyPath: "opacity") 
        fadeIn.fromValue = 0.0
        fadeIn.toValue = 1.0
        fadeIn.duration = 0.5
        fadeIn.fillMode = kCAFillModeBackwards
        fadeIn.beginTime = CACurrentMediaTime() + 0.5
        cloud1.layer.add(fadeIn, forKey: nil)
        
        fadeIn.beginTime = CACurrentMediaTime() + 0.7
        cloud2.layer.add(fadeIn, forKey: nil)
        
        fadeIn.beginTime = CACurrentMediaTime() + 0.9
        cloud3.layer.add(fadeIn, forKey: nil)
        
        fadeIn.beginTime = CACurrentMediaTime() + 1.1
        cloud4.layer.add(fadeIn, forKey: nil)
    }
```
在控制器的view已经显示的时候进行动画设置:

```
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        
        let groupAnimation = CAAnimationGroup()
        groupAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn) //添加缓冲函数
        groupAnimation.beginTime = CACurrentMediaTime() + 0.5
        groupAnimation.duration = 0.5
        groupAnimation.fillMode = kCAFillModeBackwards
        
        let scaleDown = CABasicAnimation(keyPath: "transform.scale")
        scaleDown.fromValue = 3.5
        scaleDown.toValue = 1.0
        
        let rotate = CABasicAnimation(keyPath: "transform.rotation")
        rotate.fromValue = CGFloat(M_PI_4)
        rotate.toValue = 0.0
        
        let fade = CABasicAnimation(keyPath: "opacity")
        fade.fromValue = 0.0
        fade.toValue = 1.0
        
        groupAnimation.animations = [scaleDown, rotate, fade]
        loginButton.layer.add(groupAnimation, forKey: nil)
        
        animateCloud(layer: cloud1.layer)
        animateCloud(layer: cloud2.layer)
        animateCloud(layer: cloud3.layer)
        animateCloud(layer: cloud4.layer)
        
        let flyLeft = CABasicAnimation(keyPath: "position.x")
        flyLeft.fromValue = info.layer.position.x + view.frame.size.width
        flyLeft.toValue = info.layer.position.x
        flyLeft.duration = 5.0
        flyLeft.repeatCount = 2.5
        flyLeft.autoreverses = true
        info.layer.add(flyLeft, forKey: "infoappear")
        
        let fadeLabelIn = CABasicAnimation(keyPath: "opacity")
        fadeLabelIn.fromValue = 0.2
        fadeLabelIn.toValue = 1.0
        fadeLabelIn.duration = 4.5
        info.layer.add(fadeLabelIn, forKey: "fadein")
        
        username.delegate = self
        password.delegate = self
    }
```

当登录按钮点击的时候设置动画
```
    @IBAction func login() {
        view.endEditing(true)
        
        ...

        let balloon = CALayer()
        balloon.contents = UIImage(named: "balloon")!.cgImage //layer寄宿图设置
        balloon.frame = CGRect(x: -50.0, y: 0.0, width:
            50.0, height: 65.0)
        view.layer.insertSublayer(balloon, below: username.layer)
        
        let flight = CAKeyframeAnimation(keyPath: "position") //添加关键帧动画
        flight.duration = 12.0
        flight.values = [ //NSValue结构值
            CGPoint(x: -50, y: 0.0),
            CGPoint(x: view.frame.width + 50.0, y: 160.0),
            CGPoint(x: -50.0, y: loginButton.center.y)
            ].map { NSValue(cgPoint: $0)} //进行map映射
        flight.keyTimes = [0.0, 0.5, 1.0]
        balloon.add(flight, forKey: nil)
        balloon.position = CGPoint(x: -50.0, y: loginButton.center.y)
    }
```

对`CAAnimationDelegate`代理方法进行监听: 
```
extension ViewController: CAAnimationDelegate {
    func animationDidStop(_ anim: CAAnimation, finished flag: Bool) { //当动画结束后调用
        print("animation did finish")
        
        if let name = anim.value(forKey: "name") as? String {
            if name == "form" { //拿到KVC的值
                //form field found
                let layer = anim.value(forKey: "layer") as? CALayer
                anim.setValue(nil, forKey: "layer") //清空KVC设值
                
                let pulse = CASpringAnimation(keyPath: "transform.scale") //添加弹簧动画
                pulse.damping = 7.5 //阻尼率为7.5
                pulse.fromValue = 1.25
                pulse.toValue = 1.0
                pulse.duration = pulse.settlingDuration
                layer?.add(pulse, forKey: nil)
            }
            
            if name == "cloud" {
                guard let layer = anim.value(forKey: "layer") as? CALayer else {
                    return
                }
                
                anim.setValue(nil, forKey: "layer")
                
                layer.position.x = -layer.bounds.width/2
                delay(seconds: 0.5) {
                    self.animateCloud(layer: layer)
                }
            }
        }
    }
}
```
对`UITextFieldDelegate`代理方法进行监听:
```
extension ViewController: UITextFieldDelegate {
    func textFieldDidBeginEditing(_ textField: UITextField) {
        print(info.layer.animationKeys())
        info.layer.removeAnimation(forKey: "infoappear") //当用户输入的时候移除动画
    }
    
    func textFieldDidEndEditing(_ textField: UITextField) {
        guard let text = textField.text else { return }
        if text.characters.count < 5 {
            // add animations here
            
            let jump = CASpringAnimation(keyPath: "position.y") //当用户输入完成后添加弹簧动画
            jump.initialVelocity = 100.0 //初始速度为100
            jump.mass = 10.0 //layer的质量为10
            jump.stiffness = 1500.0 //弹簧刚度为1500
            jump.damping = 50.0 //阻尼率为50
            jump.fromValue = textField.layer.position.y + 1.0
            jump.toValue = textField.layer.position.y
            jump.duration = jump.settlingDuration //持续时间设置为系统认为合理的时间
            textField.layer.add(jump, forKey: nil)
            
            textField.layer.borderWidth = 3.0
            textField.layer.borderColor = UIColor.clear.cgColor
            
            let flash = CASpringAnimation(keyPath: "borderColor")
            flash.damping = 7.0
            flash.stiffness = 200.0
            flash.fromValue = UIColor(red: 1.0, green: 0.27, blue: 0.0, alpha: 1.0).cgColor
            flash.toValue = UIColor.white.cgColor
            flash.duration = flash.settlingDuration
            textField.layer.add(flash, forKey: nil)
        }
    }
}
```
演示效果:

![](http://upload-images.jianshu.io/upload_images/1229762-d737971df1e1fdd9.gif?imageMogr2/auto-orient/strip)

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
