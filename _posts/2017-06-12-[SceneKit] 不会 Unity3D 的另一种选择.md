---
layout: post
title: SceneKit 不会 Unity3D 的另一种选择
date: 2017.06.12 15:14
tag: 移动开发
---

> 上周一, 相信很多人和我一样, 全程观看了WWDC2017的开发者大会, 其中虽然亮点平平但也能些许的看出苹果未来的战略, 虽然已经从先驱者变成跟随者, 但强者恒强的道理是亘古不变的真理, 而且在生态链的建设上也是无人能出其右, 虽然在消费者眼中最为关注的是HomePod和iPad Pro10.5, 而在开发者眼中为之眼前一亮的则是ARKit和Core ML.

![](http://upload-images.jianshu.io/upload_images/1229762-35db10c03779f7f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Core ML 刚发布的时候还以为是终于能用Swift进行模型的训练了, 终于不用学习缩进地狱的Python了, 然而这仅仅是一个类似适配器一样的东西, 把通过机器学习训练完成后的模型通过.py转换器转换成Core ML的格式并进行集成到Apple Devices中, 诶, 没意思...

ARKit 的出现能够看到苹果对扩展现实技术未来推动的决心, 的确如会上所说, ARKit凭借众多的移动设备一跃成为了最大的AR开发平台, 在会上的Demo也做的栩栩如生, 但对于ARKit, 并不是直接就能够学习的, 需要一些基础的知识, 比如跨平台的Unity, 可是并没有做过游戏开发的我, 新学一门语言虽然并非难事, 但要学个大概也并非易事啊!! 还好ARKit也支持SceneKit和SpriteKit, 诶... 亲儿子嘛, 所以为了学习之后的ARKit, 先学习下SceneKit打好基础吧~

#### SceneKit 基本概念

###### SCNVector3:

果然三维的向量, 苹果创建了一个有三个属性的结构体, 这也是意料之中的事情, 对于向量的理解就是方向加上速度, 还有毕达哥拉斯定理的应用也是非常重要的一部分.

![](http://upload-images.jianshu.io/upload_images/1229762-93e3bcf0d25f9b6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
public struct SCNVector3 {

    public var x: Float

    public var y: Float

    public var z: Float

    public init()

    public init(x: Float, y: Float, z: Float)
}

```

有了之前学习SpriteKit的经验, 现在对于游戏世界的概念也变的清晰了起来, SceneKit和SpriteKit的区别简单的来说就是二维和三维的区别, 现在我们就要对Z轴的概念需要有更深的理解了.

###### SCNCamera

摄像头的概念和之前的SKCamera还是有歇息不同的, 好歹也是个三维的摄像头, 更加真实的体现了拍电影时机位的特点, 360°无死角的拍摄, 比较能够符合这个概念吧.

![](http://upload-images.jianshu.io/upload_images/1229762-9dab5f5f62dd9b9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
  var cameraNode: SCNNode!

  func setupCamera() {

    cameraNode = SCNNode()

    cameraNode.camera = SCNCamera()

    cameraNode.position = SCNVector3(x: 0, y: 5, z: 10)

    scnScene.rootNode.addChildNode(cameraNode)
  }
```
Camera和SpriteKit中的概念相同但形式不同, 图中zNear和zFar就能了解到机位的概念, 代码中的position是指机位上调5个单位, 向后调10个单位.

###### SCNGeometry

三维几何图形, 这些几何图形都是系统框架内自带的, 当然后期可能会有其他方法进行自定义几何图形, 这些几何图形, 如果你了解CALayer的子类CAShapeLayer你就能够了解到, 只不过是二维和三维的区别了.

![](http://upload-images.jianshu.io/upload_images/1229762-b9517aacdd0ed061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
func spawnShape() {
    var geometry: SCNGeometry
    switch ShapeType.random() {
    case .box:
      geometry = SCNBox(width: 1.0, height: 1.0, length: 1.0, chamferRadius: 0.0)
    case .sphere:
      geometry = SCNSphere(radius: 0.5)
    case .pyramid:
      geometry = SCNPyramid(width: 1.0, height: 1.0, length: 1.0)
    case .torus:
      geometry = SCNTorus(ringRadius: 0.5, pipeRadius: 0.25)
    case .capsule:
      geometry = SCNCapsule(capRadius: 0.3, height: 2.5)
    case .cylinder:
      geometry = SCNCylinder(radius: 0.3, height: 2.5)
    case .cone:
      geometry = SCNCone(topRadius: 0.25, bottomRadius: 0.5, height: 1.0)
    case .tube:
      geometry = SCNTube(innerRadius: 0.25, outerRadius: 0.5, height: 1.0)
    }
    let color = UIColor.random()
    geometry.materials.first?.diffuse.contents = color
    let geometryNode = SCNNode(geometry: geometry)
    geometryNode.physicsBody = SCNPhysicsBody(type: .dynamic, shape: nil)
    
    let randomX = Float.random(min: -2, max: 2)
    let randomY = Float.random(min: 10, max: 18)
    let force = SCNVector3(x: randomX, y: randomY , z: 0)
    let position = SCNVector3(x: 0.05, y: 0.05, z: 0.05)
    geometryNode.physicsBody?.applyForce(force, at: position, asImpulse: true)
    
    let trailEmitter = createTrail(color: color, geometry: geometry)
    geometryNode.addParticleSystem(trailEmitter)
    
    if color == UIColor.black {
      geometryNode.name = "BAD"
      game.playSound(scnScene.rootNode, name: "SpawnBad")
    } else {
      geometryNode.name = "GOOD"
      game.playSound(scnScene.rootNode, name: "SpawnGood")
    }
    
    scnScene.rootNode.addChildNode(geometryNode)
  }

```
其中`SCNBox `, `SCNSphere `, `SCNPyramid `, `SCNTorus `, `SCNCapsule `, `SCNCylinder `, `SCNCone `, `SCNTube `对应这上图中的八个几何图形, 各自的初始化方法就是对于长宽高及弧度的属性设置. 
- `geometry.materials.first?.diffuse.contents = color` 表示这对几何图形的内容进行赋值
- `geometryNode.physicsBody = SCNPhysicsBody(type: .dynamic, shape: nil)` 和SpriteKit一样 创建三维的物理体
- `geometryNode.physicsBody?.applyForce(force, at: position, asImpulse: true)` 对物理体申请力及脉冲力
- `geometryNode.addParticleSystem(trailEmitter)` 添加粒子系统
#### SceneKit 渲染周期

所谓的渲染周期, 就是在每一帧系统会做哪些时期, 当然不是很理解也没关系, 我们可以用生命周期距离, UIKit中当压入栈的时候, 会出发生命周期, 从开辟到销毁会经历好几个方法, 当然渲染周期也是类似.

![](http://upload-images.jianshu.io/upload_images/1229762-af3aedce063a31cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

SceneKit在每一帧渲染的时候会经历图上九个过程:
1.更新：视图在其代理方法中进行渲染（_：updateAtTime :)。 一般写一些基本的更新逻辑.
2.执行操作和动画：SceneKit执行所有操作并执行所有连接的动画到场景图中的节点。
3.应用动画：视图调用其代理的渲染器（_：didApplyAnimationsAtTime :)。在这一点上，场景中的所有节点都基于应用的动作和动画完成了一帧的动画。
4.模拟物理：SceneKit将物理模拟的一个步骤应用于场景中的所有物理体。
5.完成模拟物理：视图在其委托上调用渲染器（_：didSimulatePhysicsAtTime :)。在这一点上，物理模拟步骤已经完成，您可以添加任何取决于上面应用的物理学的逻辑。
6.评估约束：SceneKit评估和应用约束，这是可以配置的规则，使SceneKit自动调整节点的转换。
7.将要渲染场景：视图在其委托上调用渲染器（_：willRenderScene：atTime :)。在这一点上，视图即将呈现场景，所以在这里应该执行最后一分钟的更改。
8.渲染场景视图：SceneKit渲染视图中的场景。
9.渲染场景完成：最后一步是让视图调用其代理渲染器（_：didRenderScene：atTime :)。这标志着渲染循环的一个循环的结束;您可以将任何游戏逻辑放在这里，需要在进程重新启动之前执行。

#### SceneKit 粒子系统

如果了解过CALayer的子类的话, 就可能会知道有一个粒子的Layer, 相信做过直播项目的同学们都有所了解, 其实技术做久了就会发现好多的东西概念都是相通的, 仅仅是属性和方法名不同罢了, 所以我认为做技术的朋友, 记住API本身是没有什么意义的, 毕竟现在谁开发能离开Google和Baidu, 对于技术来讲, 设计模式, 数据结构及算法, 才是技术进阶的根基, 所以现在我不再强调API了, 而是更加强调概念.

![Emitter](http://upload-images.jianshu.io/upload_images/1229762-af1f1b608f42fecb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 出生率：控制粒子的排放率。将其设置为25，指示粒子引擎以每秒25个粒子的速率产生新的粒子。
- 预热持续时间：在渲染粒子之前模拟运行的秒数。这可以用于在开始时显示充满颗粒的屏幕，而不是等待颗粒填充屏幕。将其设置为0，以便从一开始就可以观察到模拟。
- 位置：相对于形状，发射器产生其粒子的位置。将其设置为顶点，这意味着粒子将使用几何顶点作为产生位置。
- 排放空间：发射的颗粒将驻留的空间。将其设置为世界空间，以便将发射的粒子发射到世界空间，而不是对象节点本身的本地空间。
- 方向模式：控制如何产生的粒子行进;您可以将它们全部移动到恒定的方向，让它们从形状的表面径向向外移动，或者简单地将它们随机移动。将其设置为Constant，保持所有发射的粒子沿恒定方向移动。
- 方向：指定方向模式不变时使用的初始方向矢量。将此矢量设置为（x：0，y：0，z：0），将方向设置为无。
- 传播角度：随机产生的粒子的发射角度。将其设置为0°，从而在以前设置的方向上精确地发射颗粒。
- 初始角度：发射粒子的初始角度。将其设置为0°，因为这与零向矢量无关。
- 形状：发射粒子的形状。将形状设置为“球形”，从而使用球形作为几何体。
- 形状半径：此属性的存在取决于您使用的形状; 对于球形发射器，这决定了球体的大小。 将其设置为0.2，它定义了足够大的球体，满足您的需要。


![Simulation](http://upload-images.jianshu.io/upload_images/1229762-80f9077bc65e7758.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 使用寿命：指定粒子的寿命（以秒为单位）。 将其设置为1，因此单个粒子将只存在一秒钟。
- 线速度：指定发射粒子的线速度。 将其设置为0，以便粒子不会产生方向或速度。
- 角速度：指定发射的粒子的角速度。 将其设置为0，以便颗粒不会旋转。
- 加速度：指定施加到发射粒子的力矢量。 将它设置为（x：0，y：-5，z：0） - 它是一个向下的向量 - 一旦产生，就模拟颗粒上的软重力效应。
- 速度因子：设定粒子模拟速度的乘数。 将其设置为1以正常速度运行模拟。
- 拉伸因子：在其运动方向上延伸颗粒的乘数。 将其设置为0以不展开粒子图像。

![Image](http://upload-images.jianshu.io/upload_images/1229762-561b8955def86184.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 图像：指定要渲染每个粒子的图像。 选择CircleParticle.png图像，给出粒子的主要形状。
- 颜色：设置指定图像的色调。 将颜色设置为白色，给出粒子系统的基本颜色为白色。
- 动画颜色：使粒子在其使用寿命期间变色。 取消选中此项，因为粒子颜色根本不会改变。
- 颜色变化：为粒子颜色添加一点随机性。 您可以将其设置为（h：0，s：0，b：0，a：0），因为粒子颜色不会改变。
- 大小：指定粒子的大小。 将其设置为0.1，使发射的颗粒尺寸小。
- 初始帧：设置动画序列的第一个基于零的帧。 第零帧对应于网格中的左上角图像。 您正在使用单帧图像，因此将其设置为0。
- 帧率：以秒为单位控制动画的速率。 将其设置为0，因为这仅适用于使用包含多个帧的图像时。
- 动画：指定动画序列的行为。 重复循环动画，Clamp仅播放一次，Auto Reverse从开始到结束播放，然后再次播放。 你可以把它放在重复上，因为在使用单帧图像时并不重要。
- 维度：指定动画网格中的行数和列数。 由于您使用单帧图像，请将其设置为（行：1，列：1）。

![](http://upload-images.jianshu.io/upload_images/1229762-1f7dd8b3ae73f036.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 混合：指定在将粒子绘制到场景中时渲染器的混合模式。 将其设置为Alpha，将使用图像Alpha通道信息进行透明度。
- 方向：控制颗粒的旋转。 将其设置为Billboard屏幕对齐，这将始终保持平面微粒面向相机视图，因此您不会注意到颗粒确实是平面图像。
- 排序：设置粒子的渲染顺序。 此属性与混合模式配合使用，并影响如何应用混合。 将其设置为无，因此粒子系统将不会使用排序。
- 照明：控制SceneKit是否将照明应用于颗粒。 取消选中此项，以便粒子系统忽略场景中的任何指示灯。
- 受重力影响：使场景的重力影响颗粒。 取消选中此项，因为您不希望粒子系统参与物理模拟。
- 受物理场影响：造成场景内的物理场影响粒子。 取消选中此项，因为您不希望物理字段对粒子产生影响。
- 死于冲突：使您的场景中的物理体碰撞并破坏粒子。 取消选中此项，因为您不想在与场景中的节点对象冲突时删除粒子。
- 物理属性：在物理模拟过程中控制粒子物理行为的基本物理属性。 您可以将所有这些保留为默认值，因为粒子系统将不会使用它们。
- 发射持续时间：控制发射器发射新颗粒的时间长度。 将其设置为1，这将激活粒子发射器，总长度为1秒。
- 空闲持续时间：循环粒子系统在指定的发射持续时间内发射粒子，然后在指定的空闲持续时间内空转，之后循环重复。 将其设置为0，因此粒子系统将仅发射一次。
- 循环：指定粒子系统是否像爆炸一样发射粒子，或像火山一样持续发射粒子。 将其设置为循环，以使发射器在再次从场景中移除之前尽可能长的发射。

#### SceneKit 实战演练

我们今天所要实现的是一个类似水果忍者的游戏, 从底部发射出一些几何模块, 点击赚取分数, 当点到黑色的时候就会被扣除一条命. 根据我们刚刚所学的知识, 我们就能够实现出这样一个3D游戏, 就当入门吧!
![](http://upload-images.jianshu.io/upload_images/1229762-c87c9791d04f757f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### Step1 场景设置
```
  override func viewDidLoad() {
    super.viewDidLoad()
    setupView() //添加View
    setupScene() //添加场景
    setupCamera() //添加摄像头
    setupHUD() //添加文字
    setupSplash() //添加图片
    setupSounds() //添加声音
  }
  func setupView() {
    scnView = self.view as! SCNView
    //scnView.showsStatistics = true
    //scnView.allowsCameraControl = false
    scnView.autoenablesDefaultLighting = true
    scnView.delegate = self
    scnView.isPlaying = true
  }
  
  func setupScene() {
    scnScene = SCNScene()
    scnView.scene = scnScene
    scnScene.background.contents =
      "GeometryFighter.scnassets/Textures/Background_Diffuse.png"
  }
```
###### Step2 逐帧渲染
```
extension GameViewController: SCNSceneRendererDelegate {
  func renderer(_ renderer: SCNSceneRenderer, updateAtTime time:
    TimeInterval) {
      
      if game.state == .Playing { //当时可玩状态时
        if time > spawnTime { 进行生产几何模型速度的时间调节
          spawnShape()
          spawnTime = time + TimeInterval(Float.random(min: 0.2, max: 1.5))
        }
        cleanScene() //删除场景内节点
      }
      game.updateHUD() //更新文字表述
  }
}

  func cleanScene() {
    for node in scnScene.rootNode.childNodes {
      if node.presentation.position.y < -2 { 当几何图形到某个位置的时候 删除节点
        node.removeFromParentNode()
      }
    }
  }
```
###### Step3 用户交互
```
  override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    
    if game.state == .GameOver {
      return 
    }
    
    if game.state == .TapToPlay {
      game.reset()
      game.state = .Playing
      showSplash(splashName: "")
      return
    }
    
    let touch = touches.first
    let location = touch!.location(in: scnView)
    let hitResults = scnView.hitTest(location, options: nil) //获取触碰到的节点
    
    if let result = hitResults.first {
      
      if result.node.name == "HUD" || //根据节点名字判断执行业务逻辑
        result.node.name == "GAMEOVER" ||
        result.node.name == "TAPTOPLAY" {
          return
      } else if result.node.name == "GOOD" {
        handleGoodCollision() 
      } else if result.node.name == "BAD" {
        handleBadCollision()
      }
      
      createExplosion(geometry: result.node.geometry!, //爆炸效果
        position: result.node.presentation.position,
        rotation: result.node.presentation.rotation)
      
      result.node.removeFromParentNode() //删除子节点
    }
  }
 
```

###### Step4 交互逻辑

```
  func handleGoodCollision() {
    game.score += 1 //当时好的碰撞 加一分
    game.playSound(scnScene.rootNode, name: "ExplodeGood")
  }
  
  func handleBadCollision() {
    game.lives -= 1 当时坏的碰撞 减条命
    game.playSound(scnScene.rootNode, name: "ExplodeBad")
    game.shakeNode(cameraNode)
    
    if game.lives <= 0 { //当命数等于零时 游戏结束
      game.saveState()
      showSplash(splashName: "GameOver")
      game.playSound(scnScene.rootNode, name: "GameOver")
      game.state = .GameOver
      scnScene.rootNode.runAction(SCNAction.waitForDurationThenRunBlock(5) { (node:SCNNode!) -> Void in
        self.showSplash(splashName: "TapToPlay")
        self.game.state = .TapToPlay
        })
    }
  }
```

###### Step5 爆炸效果
```
  func createExplosion(geometry: SCNGeometry, position: SCNVector3,
    rotation: SCNVector4) {
      let explosion =
      SCNParticleSystem(named: "Explode.scnp", inDirectory:
        nil)!
      explosion.emitterShape = geometry
      explosion.birthLocation = .surface
      let rotationMatrix =
      SCNMatrix4MakeRotation(rotation.w, rotation.x, 
        rotation.y, rotation.z)
      let translationMatrix =
      SCNMatrix4MakeTranslation(position.x, position.y, position.z)
      let transformMatrix =
      SCNMatrix4Mult(rotationMatrix, translationMatrix)
      scnScene.addParticleSystem(explosion, transform: transformMatrix)
  }
```

##### 演示效果:

![](http://upload-images.jianshu.io/upload_images/1229762-82efcbd3df3541a4.gif?imageMogr2/auto-orient/strip)

##### About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [ 源码 请点这里 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
