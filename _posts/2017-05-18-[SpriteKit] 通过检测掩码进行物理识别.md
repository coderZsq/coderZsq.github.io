---
layout: post
title: SpriteKit 通过检测掩码进行物理识别
date: 2017.05.18 16:38
tag: 移动开发
---

> 上周忙着写代码生成器了, 回到我们的学习中来, 今天我们来继续学习SpriteKit, 上篇我们学习了一些2D游戏场景, 精灵, 摄像头, 行动的基本概念, 今天我们就来进入物理的世界, 感受碰撞所带来的乐趣.代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-b5433f0a638b1a15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

iOS7有出过UIDynamic, 用于UIVIew的物理动画效果, 同期Apple也出了SpriteKit框架, 到现在也已经3个年头了, 我们学习SpriteKit的目的也在于更好的对UIKit进行了解. 也算是一条技术进阶的路吧, 我们先来看一下学习物理世界所需要熟悉的API.

##### SKPhysicsWorld
```
open class SKPhysicsWorld : NSObject, NSCoding
    open var gravity: CGVector
    open var speed: CGFloat
    unowned(unsafe) open var contactDelegate: SKPhysicsContactDelegate?

    open func add(_ joint: SKPhysicsJoint)
    open func remove(_ joint: SKPhysicsJoint)
    open func removeAllJoints()
    open func body(in rect: CGRect) -> SKPhysicsBody?
    open func enumerateBodies(in rect: CGRect, using block: @escaping (SKPhysicsBody, UnsafeMutablePointer<ObjCBool>) -> Swift.Void)
```
- `gravity` 物理世界的重力
- `speed `物理世界的速度
- `contactDelegate ` 物理世界的代理
-  `add(_ joint: SKPhysicsJoint)` 给物理世界添加一个连接
-  `remove(_ joint: SKPhysicsJoint)` 删除物理世界中的一个连接
- `removeAllJoints()` 删除物理世界的所有连接
- `body(in rect: CGRect) -> SKPhysicsBody?` 返回物理世界中规定矩形内的物理体
- `enumerateBodies(in rect: CGRect, using block:` 遍历物理世界中所有的物理体

##### SKPhysicsBody
```
open class SKPhysicsBody : NSObject, NSCopying, NSCoding
    public /*not inherited*/ init(circleOfRadius r: CGFloat, center: CGPoint)
    public /*not inherited*/ init(rectangleOf s: CGSize, center: CGPoint)
    public /*not inherited*/ init(polygonFrom path: CGPath)
    public /*not inherited*/ init(edgeLoopFrom rect: CGRect)
@available(iOS 8.0, *)
    public /*not inherited*/ init(texture: SKTexture, size: CGSize)
    public /*not inherited*/ init(bodies: [SKPhysicsBody])

    open var isDynamic: Bool
    open var restitution: CGFloat
    open var density: CGFloat
    open var mass: CGFloat
    open var categoryBitMask: UInt32
    open var collisionBitMask: UInt32
    open var contactTestBitMask: UInt32
    open var joints: [SKPhysicsJoint] { get }
    weak open var node: SKNode? { get }
    open var velocity: CGVector
    open var angularVelocity: CGFloat

    open func applyForce(_ force: CGVector, at point: CGPoint)
    open func applyTorque(_ torque: CGFloat)
    open func applyImpulse(_ impulse: CGVector, at point: CGPoint)
    open func allContactedBodies() -> [SKPhysicsBody]
```
- `init(circleOfRadius r: CGFloat, center: CGPoint)` 通过半径和原点创建圆形物理体
- `init(rectangleOf s: CGSize, center: CGPoint)` 通过中点和尺寸创建矩形物理体
- `init(polygonFrom path: CGPath)` 通过路径创建多边形不规则物理体
-  `init(edgeLoopFrom rect: CGRect)` 通过矩形创建边缘物理体, 这个不好描述, 就是向内的物理体
- `init(texture: SKTexture, size: CGSize)`通过纹理和尺寸创建以Alpha通道的物理体
-  `init(bodies: [SKPhysicsBody])` 根据物理体创建可合并的物理体
- `isDynamic ` 物理体是否进入物理世界
-  `restitution ` 物理体的弹性, 姑且叫弹性吧 =. = 
-  `density ` 物理体的密度
- `mass ` 物理体的质量
- `categoryBitMask ` 物理体的类型标识
-  `collisionBitMask ` 物理体的碰撞标识
- `contactTestBitMask ` 物理体的触碰标识
- `joints` 物理体的连接点
- `node ` 物理体本身的节点
-  `velocity ` 物理体的速度
- `angularVelocity ` 物理体的角速度
- `applyForce(_ force: CGVector, at point: CGPoint)` 通过向量和点给物理体添加一个力
- `applyTorque(_ torque: CGFloat) `给物理体添加一个扭力
-  `applyImpulse(_ impulse: CGVector, at point: CGPoint)` 通过向量和点给物理体添加一个脉冲力
- `allContactedBodies() -> [SKPhysicsBody]` 返回所有与之接触的物理体

##### SKPhysicsContactDelegate
```
public protocol SKPhysicsContactDelegate : NSObjectProtocol
    optional public func didBegin(_ contact: SKPhysicsContact)
    optional public func didEnd(_ contact: SKPhysicsContact)
```
- `didBegin(_ contact: SKPhysicsContact)` 物理世界中的物理体开始接触的时候调用
- `didEnd(_ contact: SKPhysicsContact)` 物理世界中的物理体结束接触的时候调用

##### SKPhysicsContact
```
open class SKPhysicsContact : NSObject
    open var bodyA: SKPhysicsBody { get }
    open var bodyB: SKPhysicsBody { get }
    open var contactPoint: CGPoint { get }
```
- `bodyA `只读属性, 物理接触的物理体A
- ` bodyB `只读属性, 物理接触的物理体B
- `contactPoint` 只读属性, 物理体A和物理体B的连接点

##### SKPhysicsJoint
```
open class SKPhysicsJoint : NSObject, NSCoding
    open var bodyA: SKPhysicsBody
    open var bodyB: SKPhysicsBody

open class SKPhysicsJointPin : SKPhysicsJoint
    open class func joint(withBodyA bodyA: SKPhysicsBody, bodyB: SKPhysicsBody, anchor: CGPoint) -> SKPhysicsJointPin

open class SKPhysicsJointSpring : SKPhysicsJoint
    open class func joint(withBodyA bodyA: SKPhysicsBody, bodyB: SKPhysicsBody, anchorA: CGPoint, anchorB: CGPoint) -> SKPhysicsJointSpring

open class SKPhysicsJointFixed : SKPhysicsJoint
    open class func joint(withBodyA bodyA: SKPhysicsBody, bodyB: SKPhysicsBody, anchor: CGPoint) -> SKPhysicsJointFixed

open class SKPhysicsJointSliding : SKPhysicsJoint
    open class func joint(withBodyA bodyA: SKPhysicsBody, bodyB: SKPhysicsBody, anchor: CGPoint, axis: CGVector) -> SKPhysicsJointSliding

open class SKPhysicsJointLimit : SKPhysicsJoint
    open class func joint(withBodyA bodyA: SKPhysicsBody, bodyB: SKPhysicsBody, anchorA: CGPoint, anchorB: CGPoint) -> SKPhysicsJointLimit
```
- `bodyA` 物理链接的物理体A
- `bodyB` 物理链接的物理体B
- `SKPhysicsJointPin ` 针连接
- `SKPhysicsJointSpring ` 弹簧连接
- `SKPhysicsJointFixed ` 固定连接
- `SKPhysicsJointSliding` 滑动链接
- `SKPhysicsJointLimit` 限制链接

##### SKConstraint
```
@available(iOS 8.0, *)
open class SKConstraint : NSObject, NSCoding, NSCopying
    open var enabled: Bool
    open class func orient(to node: SKNode, offset radians: SKRange) -> Self
    open class func zRotation(_ zRange: SKRange) -> Self
    open class func distance(_ range: SKRange, to node: SKNode) -> Self
    open class func positionX(_ xRange: SKRange, y yRange: SKRange) -> Self

open class SKRange : NSObject, NSCoding, NSCopying 
    public init(lowerLimit lower: CGFloat, upperLimit upper: CGFloat)
```
- `enabled` 是否进行对节点的约束
- `orient(to node: SKNode, offset radians: SKRange)`对节点的方向进行约束
- `zRotation(_ zRange: SKRange)` 对节点的旋转进行约束
-  `distance(_ range: SKRange, to node: SKNode)` 对节点的距离进行约束
- `positionX(_ xRange: SKRange, y yRange: SKRange) ` 对节点的位置进行约束
- `SKRange` 节点约束的范围

#### 实战: 
API, 了解一些基本的就够了, 如果要深究可以打开头文件逐个尝试, 我们现在就来实现一个小游戏, 这个游戏中包含了6个场景, 就像通关游戏一样, 破坏物体使猫睡在床上即为闯关成功反之则为失败, 成功后进入下一关, 我们着手进行游戏的开发吧!


![](http://upload-images.jianshu.io/upload_images/1229762-c989ff9849ff42a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- step1 创建游戏后自动生成GameViewController, 从可视化文件中读取场景

```
  override func viewDidLoad() {
    super.viewDidLoad()

    if let view = self.view as! SKView? {
      // Load the SKScene from 'GameScene.sks' //从可视化文件中读取场景
      if let scene = GameScene.level(levelNum: 1) {
        // Set the scale mode to scale to fit the window
        scene.scaleMode = .aspectFill

        // Present the scene
        view.presentScene(scene)
      }

      view.ignoresSiblingOrder = false //不忽略同级节点

      view.showsFPS = true
      view.showsNodeCount = true
      view.showsPhysics = true //显示物理体
    }
```

- step2 动态化配置场景文件

```
  //1
  var currentLevel: Int = 0 //当前场景等级

  //2
  class func level(levelNum: Int) -> GameScene? {
    let scene = GameScene(fileNamed: "Level\(levelNum)")! //根据等级读取可视化文件场景
    scene.currentLevel = levelNum //赋值给当前等级
    scene.scaleMode = .aspectFill
    return scene
  }
```

- step3 设置各物理体标识, 根据左移来进行判断

```
struct PhysicsCategory {
  static let None:  UInt32 = 0 //不表示物理体
  static let Cat:   UInt32 = 0b1 // 1 //猫物理体标识
  static let Block: UInt32 = 0b10 // 2 //砖块物理体标识
  static let Bed:   UInt32 = 0b100 // 4 //床物理体标识
  static let Edge:  UInt32 = 0b1000 // 8 //边缘物理体标识
  static let Label: UInt32 = 0b10000 // 16 //标签物理体标识
  static let Spring:UInt32 = 0b100000 // 32 //弹簧物理体标识
  static let Hook:  UInt32 = 0b1000000 // 64 //钩子物理体标识
}
```
- step4 设置精灵节点的事件和交互协议

```
protocol EventListenerNode {
  func didMoveToScene() //当节点进入场景时调用
}
protocol InteractiveNode {
  func interact() //用户交互时调用
}
```
- step5 猫节点的设置

```
class CatNode: SKSpriteNode, EventListenerNode, InteractiveNode { //遵守事件和交互协议

  static let kCatTappedNotification = "kCatTappedNotification"

  func didMoveToScene() {
    print("cat added to scene")

    let catBodyTexture = SKTexture(imageNamed: "cat_body_outline") //读取物理体纹理
    parent!.physicsBody = SKPhysicsBody(texture: catBodyTexture, size: catBodyTexture.size()) //通过纹理创建物理体

    parent!.physicsBody!.categoryBitMask = PhysicsCategory.Cat //将该节点设置为猫物理体标识
    parent!.physicsBody!.collisionBitMask = PhysicsCategory.Block | PhysicsCategory.Edge | PhysicsCategory.Spring //猫物理体标识将和边缘物理体, 砖块物理体, 弹簧物理体碰撞
    parent!.physicsBody!.contactTestBitMask = PhysicsCategory.Bed | PhysicsCategory.Edge //猫物理体将会和床物理体, 边缘物理体接触

    isUserInteractionEnabled = true //猫节点可以和用户进行交互
  }

  func wakeUp() { //醒来
    // 1
    for child in children {
      child.removeFromParent() //遍历移除所有的子节点
    }
    texture = nil //置空纹理
    color = SKColor.clear //设置透明色
    // 2
    let catAwake = SKSpriteNode(fileNamed: "CatWakeUp")!.childNode(withName: "cat_awake")! //获取可视化场景中的name为cat_awake的节点
    // 3
    catAwake.move(toParent: self) //移动节点到上层
    catAwake.position = CGPoint(x: -30, y: 100) //设置位置
  }

  func curlAt(scenePoint: CGPoint) { //蜷缩
    parent!.physicsBody = nil //置空物理体
    for child in children {
      child.removeFromParent() /
    }
    texture = nil 
    color = SKColor.clear 

    let catCurl = SKSpriteNode(fileNamed: "CatCurl")!.childNode(withName: "cat_curl")!
    catCurl.move(toParent: self)
    catCurl.position = CGPoint(x: -30, y: 100) 

    var localPoint = parent!.convert(scenePoint, from: scene!) //从场景中转换坐标
    localPoint.y += frame.size.height/3 

    run(SKAction.group([ //进行组动画
      SKAction.move(to: localPoint, duration: 0.66),
      SKAction.rotate(toAngle: -parent!.zRotation, duration: 0.5)
      ]))
  }

  func interact() {
    NotificationCenter.default.post(Notification(name: //发送通知
      NSNotification.Name(CatNode.kCatTappedNotification), object: nil))

    if DiscoBallNode.isDiscoTime && !isDoingTheDance { 
      isDoingTheDance = true
      let move = SKAction.sequence([
        SKAction.moveBy(x: 80, y: 0, duration: 0.5),
        SKAction.wait(forDuration: 0.5),
        SKAction.moveBy(x: -30, y: 0, duration: 0.5)
        ])
      let dance = SKAction.repeat(move, count: 3)
      parent!.run(dance, completion: {
        self.isDoingTheDance = false
      })
    }
  }

  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    interact()
  }

  private var isDoingTheDance = false


}
```
- step6 石头节点的设置

```
class StoneNode: SKSpriteNode, EventListenerNode, InteractiveNode {
  func didMoveToScene() {
    guard let scene = scene else { //对场景进行校验
      return
    }

    if parent == scene {
      scene.addChild(StoneNode.makeCompoundNode(in: scene)) //添加石头节点进入场景
    }
  }

  func interact() {
    isUserInteractionEnabled = false //不可交互
    run( SKAction.sequence([
      SKAction.playSoundFileNamed("pop.mp3",
                                  waitForCompletion: false),
      SKAction.removeFromParent()
      ]))
  }

  static func makeCompoundNode(in scene: SKScene) -> SKNode { //创建复合节点
    let compound = StoneNode() 
    for stone in scene.children.filter({ node in node is StoneNode}) { //过滤节点
        stone.removeFromParent()
        compound.addChild(stone)
    }

    let bodies = compound.children.map({ node in //映射节点
      SKPhysicsBody(rectangleOf: node.frame.size, center: node.position) //设置物理体
    })

    compound.physicsBody = SKPhysicsBody(bodies: bodies) //合并物理体
    compound.physicsBody!.collisionBitMask = PhysicsCategory.Edge  | PhysicsCategory.Cat | PhysicsCategory.Block //设置碰撞标识
    compound.physicsBody!.categoryBitMask = PhysicsCategory.Block //设置为砖块物理体标识
    compound.isUserInteractionEnabled = true 
    compound.zPosition = 1
    return compound
  }

  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    interact()
  }


}
```
- step7 床节点的设置

```
class BedNode: SKSpriteNode, EventListenerNode {
  func didMoveToScene() {
    print("bed added to scene")

    let bedBodySize = CGSize(width: 40.0, height: 30.0)
    physicsBody = SKPhysicsBody(rectangleOf: bedBodySize)
    physicsBody!.isDynamic = false //不进入物理世界

    physicsBody!.categoryBitMask = PhysicsCategory.Bed
    physicsBody!.collisionBitMask = PhysicsCategory.None
  }
}
```

- step8 砖块节点的设置

```
class BlockNode: SKSpriteNode, EventListenerNode, InteractiveNode {

  func didMoveToScene() {
    isUserInteractionEnabled = true 
  }

  func interact() {
    isUserInteractionEnabled = false

    run(SKAction.sequence([
      SKAction.playSoundFileNamed("pop.mp3",
                                  waitForCompletion: false),
      SKAction.scale(to: 0.8, duration: 0.1),
      SKAction.removeFromParent()
      ]))
  }

  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    print("destroy block")
    interact()
  }
  
}

```

- step9 标签节点的设置

```
class MessageNode: SKLabelNode {

  convenience init(message: String) {
    self.init(fontNamed: "AvenirNext-Regular") //设置字体
    text = message
    fontSize = 256.0
    fontColor = SKColor.gray
    zPosition = 100
    let front = SKLabelNode(fontNamed: "AvenirNext-Regular")
    front.text = message
    front.fontSize = 256.0
    front.fontColor = SKColor.white
    front.position = CGPoint(x: -2, y: -2)
    addChild(front)

    physicsBody = SKPhysicsBody(circleOfRadius: 10) //设置物理体
    physicsBody!.collisionBitMask = PhysicsCategory.Edge
    physicsBody!.categoryBitMask = PhysicsCategory.Label
    physicsBody!.contactTestBitMask = PhysicsCategory.Edge
    physicsBody!.restitution = 0.7 //设置物理体弹性
  }

  //chapter 9, challenge 1
  private var bounceCount = 0

  func didBounce() {
    bounceCount += 1
    if bounceCount >= 4 { //弹起次数判断
      removeFromParent()
    }
  }
}
```

- step10 弹簧节点设置

```
class SpringNode: SKSpriteNode, EventListenerNode, InteractiveNode {
  func didMoveToScene() {
    isUserInteractionEnabled = true
  }

  func interact() {
    isUserInteractionEnabled = false
    physicsBody!.applyImpulse(CGVector(dx: 0, dy: 250), // 给物理体添加一个向上的脉冲力
                              at: CGPoint(x: size.width/2, y: size.height))
    run(SKAction.sequence([
      SKAction.wait(forDuration: 1),
      SKAction.removeFromParent()
      ]))
  }

  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    interact()
  }
}
```

- step11 钩子地基节点的设置

```
class HookBaseNode: SKSpriteNode, EventListenerNode {

  private var hookNode = SKSpriteNode(imageNamed: "hook") // 钩子节点
  private var ropeNode = SKSpriteNode(imageNamed: "rope") //绳子节点
  private var hookJoint: SKPhysicsJointFixed! //钩子的连接

  var isHooked: Bool { //是否被勾住
    return hookJoint != nil
  }

  func didMoveToScene() {
    guard let scene = scene else {
      return
    }

    let ceilingFix = SKPhysicsJointFixed.joint(withBodyA: //将场景和钩子地基物理体进行固定连接
      scene.physicsBody!, bodyB: physicsBody!, anchor: CGPoint.zero)
    scene.physicsWorld.add(ceilingFix) //将连接添加到物理世界中

    ropeNode.anchorPoint = CGPoint(x: 0, y: 0.5)
    ropeNode.zRotation = CGFloat(270).degreesToRadians()
    ropeNode.position = position
    scene.addChild(ropeNode)

    hookNode.position = CGPoint(
      x: position.x,
      y: position.y - ropeNode.size.width)
    hookNode.physicsBody =
      SKPhysicsBody(circleOfRadius: hookNode.size.width/2) //物理体标识的设置
    hookNode.physicsBody!.categoryBitMask = PhysicsCategory.Hook
    hookNode.physicsBody!.contactTestBitMask = PhysicsCategory.Cat
    hookNode.physicsBody!.collisionBitMask = PhysicsCategory.None
    scene.addChild(hookNode)

    let hookPosition = CGPoint(x: hookNode.position.x,
                               y: hookNode.position.y+hookNode.size.height/2)
    let ropeJoint = SKPhysicsJointSpring.joint( //钩子节点和钩子地基节点进行弹簧连接
      withBodyA: physicsBody!, bodyB: hookNode.physicsBody!,
      anchorA: position,
      anchorB: hookPosition)
    scene.physicsWorld.add(ropeJoint) 

    let range = SKRange(lowerLimit: 0.0, upperLimit: 0.0) //约束范围
    let orientConstraint = SKConstraint.orient(to: hookNode, offset: range) //添加方向约束
    ropeNode.constraints = [orientConstraint] //往绳子节点中添加约束

    hookNode.physicsBody!.applyImpulse(CGVector(dx: 50, dy: 0)) //给钩子节点添加一个横向的脉冲力

    NotificationCenter.default.addObserver( //添加通知
      self, selector: #selector(catTapped), name: Notification
        .Name(CatNode.kCatTappedNotification), object: nil)
  }

  func hookCat(catPhysicsBody: SKPhysicsBody) { //勾住猫
    catPhysicsBody.velocity = CGVector(dx: 0, dy: 0) //速度清零
    catPhysicsBody.angularVelocity = 0 //角速度清零

    let pinPoint = CGPoint( 
      x: hookNode.position.x,
      y: hookNode.position.y + hookNode.size.height/2)
    hookJoint = SKPhysicsJointFixed.joint( //钩子物理体和猫物理体进行固定连接
      withBodyA: hookNode.physicsBody!, bodyB: catPhysicsBody,
      anchor: pinPoint)
    scene!.physicsWorld.add(hookJoint)
    hookNode.physicsBody!.contactTestBitMask = PhysicsCategory.None
    
  }

  func releaseCat() { //释放猫
    hookNode.physicsBody!.categoryBitMask = PhysicsCategory.None
    hookNode.physicsBody!.contactTestBitMask = PhysicsCategory.None
    hookJoint.bodyA.node!.zRotation = 0
    hookJoint.bodyB.node!.zRotation = 0
    scene!.physicsWorld.remove(hookJoint) //物理世界中移除连接
    hookJoint = nil
  }

  func catTapped() { //点击猫
    if isHooked {
      releaseCat() 
    }
  }
}
```
- step12 相框节点的设置

```
class PictureNode: SKSpriteNode, EventListenerNode, InteractiveNode {

  func didMoveToScene() {
    isUserInteractionEnabled = true

    let pictureNode = SKSpriteNode(imageNamed: "picture") //相框节点
    let maskNode = SKSpriteNode(imageNamed: "picture-frame-mask") //蒙版节点

    let cropNode = SKCropNode() //类似于CALayer
    cropNode.addChild(pictureNode)
    cropNode.maskNode = maskNode //添加蒙版
    addChild(cropNode)
  }

  func interact() {
    isUserInteractionEnabled = false
    physicsBody!.isDynamic = true //进入物理世界
  }

  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    interact()
  }
}
```

- step13 迪斯科球节点的设置

```
class DiscoBallNode: SKSpriteNode, EventListenerNode, InteractiveNode {

  private var player: AVPlayer!
  private var video: SKVideoNode!

  func didMoveToScene() {
    isUserInteractionEnabled = true

    let fileUrl = Bundle.main.url(forResource: "discolights-loop", withExtension: "mov")! //读取视频文件
    player = AVPlayer(url: fileUrl)
    video = SKVideoNode(avPlayer:player)

    video.size = scene!.size
    video.position = CGPoint(
      x: scene!.frame.midX,
      y: scene!.frame.midY)
    video.zPosition = -1
    scene!.addChild(video)
    video.isHidden = true
    video.pause()

    NotificationCenter.default.addObserver(self,
                                           selector: #selector(didReachEndOfVideo),
                                           name: NSNotification.Name.AVPlayerItemDidPlayToEndTime,
                                           object: nil)

    video.alpha = 0.75
  }

  func interact() {
    if !isDiscoTime {
      isDiscoTime = true
    }
  }

  func didReachEndOfVideo() {
    print("rewind!")
    player.currentItem!.seek(to: kCMTimeZero)
    player.play()
  }

  private var isDiscoTime: Bool = false {
    didSet {
      video.isHidden = !isDiscoTime

      if isDiscoTime {
        video.play()
        run(spinAction)
      } else {
        video.pause()
        removeAllActions()
      }

      SKTAudio.sharedInstance().playBackgroundMusic(
        isDiscoTime ? "disco-sound.m4a" : "backgroundMusic.mp3")

      if isDiscoTime {
        video.run(SKAction.wait(forDuration: 5.0), completion: {
          self.isDiscoTime = false
        })
      }

      DiscoBallNode.isDiscoTime = isDiscoTime
    }
  }

  override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    interact()
  }

  private let spinAction = SKAction.repeatForever(
    SKAction.animate(with: [
      SKTexture(imageNamed: "discoball1"),
      SKTexture(imageNamed: "discoball2"),
      SKTexture(imageNamed: "discoball3")
      ], timePerFrame: 0.2))

  static private(set) var isDiscoTime = false


}
```
- step14 提示节点的设置

```
class HintNode: SKSpriteNode, EventListenerNode {

  func didMoveToScene() {
    color = SKColor.clear

    let shape = SKShapeNode(path: arrowPath)
    shape.strokeColor = SKColor.gray
    shape.lineWidth = 4
    shape.fillColor = SKColor.white
    addChild(shape)

    shape.fillTexture = SKTexture(imageNamed: "wood_tinted")
    shape.alpha = 0.8
    shape.fillColor = SKColor.green
    
    let move = SKAction.moveBy(x: -40, y: 0, duration: 1.0)
    let bounce = SKAction.sequence([
      move, move.reversed()
      ])
    let bounceAction = SKAction.repeat(bounce, count: 3)
    shape.run(bounceAction, completion: {
      self.removeFromParent()
    })
  }

  var arrowPath: CGPath = {
    let bezierPath = UIBezierPath() //绘制贝塞尔曲线
    bezierPath.move(to: CGPoint(x: 0.5, y: 65.69))
    bezierPath.addLine(to: CGPoint(x: 74.99, y: 1.5))
    bezierPath.addLine(to: CGPoint(x: 74.99, y: 38.66))
    bezierPath.addLine(to: CGPoint(x: 257.5, y: 38.66))
    bezierPath.addLine(to: CGPoint(x: 257.5, y: 92.72))
    bezierPath.addLine(to: CGPoint(x: 74.99, y: 92.72))
    bezierPath.addLine(to: CGPoint(x: 74.99, y: 126.5))
    bezierPath.addLine(to: CGPoint(x: 0.5, y: 65.69))
    bezierPath.close()
    return bezierPath.cgPath
  }()

}
```

- step15 进入场景的初始化设置

```
  override func didMove(to view: SKView) {
    // Calculate playable margin
    let maxAspectRatio: CGFloat = 16.0/9.0
    let maxAspectRatioHeight = size.width / maxAspectRatio
    let playableMargin: CGFloat = (size.height - maxAspectRatioHeight)/2
    let playableRect = CGRect(x: 0, y: playableMargin,
                              width: size.width, height: size.height-playableMargin*2)
    physicsBody = SKPhysicsBody(edgeLoopFrom: playableRect) //屏幕边缘物理体
    physicsWorld.contactDelegate = self //设置物理世界代理
    physicsBody!.categoryBitMask = PhysicsCategory.Edge //设置物理体标识

    enumerateChildNodes(withName: "//*", using: { node, _ in //遍历子节点, 调用方法
      if let eventListenerNode = node as? EventListenerNode {
        eventListenerNode.didMoveToScene()
      }
    })

    bedNode = childNode(withName: "bed") as! BedNode
    catNode = childNode(withName: "//cat_body") as! CatNode //节点name是有层级的

    SKTAudio.sharedInstance().playBackgroundMusic("backgroundMusic.mp3")

//    let rotationConstraint = SKConstraint.zRotation(
//      SKRange(lowerLimit: -π/4, upperLimit: π/4))
//    catNode.parent!.constraints = [rotationConstraint]

    hookBaseNode = childNode(withName: "hookBase") as? HookBaseNode

  }
```
- step16 加载完成物理世界时调用, 生命周期方法

```
  override func didSimulatePhysics() {
    if playable && hookBaseNode?.isHooked != true {
      if fabs(catNode.parent!.zRotation) >
        CGFloat(25).degreesToRadians() {
        lose() }
    }
  }
```
- step17 当物理体进行接触的时候调用 物理世界代理方法

```

  func didBegin(_ contact: SKPhysicsContact) {
    let collision = contact.bodyA.categoryBitMask | contact.bodyB.categoryBitMask // 碰撞标识

    if collision == PhysicsCategory.Label | PhysicsCategory.Edge { // 标签物理体与边界碰撞

      let labelNode = contact.bodyA.categoryBitMask == PhysicsCategory.Label ?
        contact.bodyA.node :
        contact.bodyB.node

      if let message = labelNode as? MessageNode {
        message.didBounce()
      }
    }

    if !playable {
      return
    }
    
    if collision == PhysicsCategory.Cat | PhysicsCategory.Bed { //猫物理体与床物理体接触
      print("SUCCESS")
      win()
    } else if collision == PhysicsCategory.Cat | PhysicsCategory.Edge { //猫物理体与边缘物理体碰撞
      print("FAIL")
      lose()
    }

    if collision == PhysicsCategory.Cat | PhysicsCategory.Hook //猫物理体与钩子物理体接触
      && hookBaseNode?.isHooked == false {
      hookBaseNode!.hookCat(catPhysicsBody:
        catNode.parent!.physicsBody!)
    }

    
  }
```

- step18 其他相关的游戏逻辑

```
 func inGameMessage(text: String) { //显示游戏标签
    let message = MessageNode(message: text)
    message.position = CGPoint(x: frame.midX, y: frame.midY)
    addChild(message)
  }

  func newGame() { //进入新场景
    view!.presentScene(GameScene.level(levelNum: currentLevel))
  }

  func lose() { //闯关失败
    if currentLevel > 1 {
      currentLevel -= 1
    }
    
    playable = false
    //1
    SKTAudio.sharedInstance().pauseBackgroundMusic()
    SKTAudio.sharedInstance().playSoundEffect("lose.mp3")
    //2
    inGameMessage(text: "Try again...")
    //3
    perform(#selector(newGame), with: nil, afterDelay: 5)

    catNode.wakeUp()
  }

  func win() { //闯关成功
    if currentLevel < 6 {
      currentLevel += 1
    }

    playable = false
    SKTAudio.sharedInstance().pauseBackgroundMusic()
    SKTAudio.sharedInstance().playSoundEffect("win.mp3")
    inGameMessage(text: "Nice job!")
    perform(#selector(GameScene.newGame), with: nil, afterDelay: 3)

    catNode.curlAt(scenePoint: bedNode.position)
  }

```
Notice: 一些可视化场景.sks文件仅是添加一些精灵节点的设置和Action等, 可以用代码实现

##### 演示效果:
![](http://upload-images.jianshu.io/upload_images/1229762-dc6bcc53249d7394.gif?imageMogr2/auto-orient/strip)

##### About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
