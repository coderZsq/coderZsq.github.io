---
layout: post
title: SpriteKit 瓦片地图什么的还挺好玩儿
date: 2017.05.24 11:49
tag: 移动开发
---

> 说实话这个2D游戏实战的入门看的我脑浆子都沸腾了, 好多新的概念涌入, 没做过游戏开发的我表示真的难以接受, 吸收效率与之前相比也下降好多, 不过越往后学, 就能够加深对之前知识的掌握, 这可能也是看书的好处吧, 今天我也把对瓦片地图的一些学习经验记录下来供大家探讨.代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-b5433f0a638b1a15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

说实话, 我很推荐Ray家的资源, 由浅入深手把手的教学, 内容前后呼应, 看几本书就能涵盖国内4个月培训班的课程体系. 遵循本系列一贯的风格, 我们还是从基础的API开始看起, 对API掌握熟练的话, 多敲两个Demo就能够基本的上手任何项目了. 
##### SKTileMapNode
```
@available(iOS 10.0, *)
open class SKTileMapNode : SKNode, NSCopying, NSCoding
     public init(tileSet: SKTileSet, columns: Int, rows: Int, tileSize: CGSize)
     open var numberOfColumns: Int
     open var numberOfRows: Int
     open var tileSize: CGSize
     open var mapSize: CGSize { get }
     open var tileSet: SKTileSet
     open var colorBlendFactor: CGFloat
     open func tileDefinition(atColumn column: Int, row: Int) -> SKTileDefinition?
     open func tileGroup(atColumn column: Int, row: Int) -> SKTileGroup?
     open func setTileGroup(_ tileGroup: SKTileGroup?, forColumn column: Int, row: Int)
     open func tileColumnIndex(fromPosition position: CGPoint) -> Int
     open func tileRowIndex(fromPosition position: CGPoint) -> Int
     open func centerOfTile(atColumn column: Int, row: Int) -> CGPoint
```
- `init(tileSet: SKTileSet, columns: Int, rows: Int, tileSize: CGSize)` 瓦片地图节点的初始化方法
- `numberOfColumns` 瓦片地图的列数
- `numberOfRows` 瓦片地图的行数
- `tileSize` 瓦片地图中每个瓦片的尺寸
- `mapSize` 瓦片地图的尺寸
- `tileSet` 瓦片地图的瓦片集
- `colorBlendFactor` 瓦片的渲染着色
- `tileDefinition(atColumn column: Int, row: Int) -> SKTileDefinition?` 根据列数和行数返回瓦片定义
- `tileGroup(atColumn column: Int, row: Int) -> SKTileGroup?` 根据列数和行数返回瓦片组
- `setTileGroup(_ tileGroup: SKTileGroup?, forColumn column: Int, row: Int)` 根据列数和行数设置瓦片组
- `tileColumnIndex(fromPosition position: CGPoint) -> Int` 根据瓦片位置返回瓦片在瓦片地图中列数下标
- ` tileRowIndex(fromPosition position: CGPoint) -> Int` 根据瓦片位置返回瓦片在瓦片地图中行数下标

##### SKTileSet
```
@available(iOS 10.0, *)
open class SKTileSet : NSObject, NSCopying, NSCoding 
     public init(tileGroups: [SKTileGroup])
     open var tileGroups: [SKTileGroup]
     open var name: String?
     open var defaultTileGroup: SKTileGroup?
     open var defaultTileSize: CGSize
     open var type: SKTileSetType

@available(iOS 10.0, *)
public enum SKTileSetType : UInt {
    case grid
    case isometric
    case hexagonalFlat
    case hexagonalPointy
}
```
- `init(tileGroups: [SKTileGroup])` 根据瓦片组初始化瓦片集
- `tileGroups` 瓦片组
- `name` 瓦片集的标识
- `defaultTileGroup` 瓦片集默认瓦片组
- `defaultTileSize` 瓦片集默认瓦片尺寸
- `type` 瓦片集类型 - 网格, 等值, 六边形

##### SKTileGroup
```
@available(iOS 10.0, *)
open class SKTileGroup : NSObject, NSCopying, NSCoding
     open class func empty() -> Self
     public init(tileDefinition: SKTileDefinition)
     public init(rules: [SKTileGroupRule])
     open var name: String?
```
- `empty()` 返回一个空的瓦片组
- `init(tileDefinition: SKTileDefinition)` 根据瓦片定义初始化瓦片组
- `init(rules: [SKTileGroupRule])` 根据瓦片组规则初始化瓦片组
- `name` 瓦片组的标识

##### SKTileGroupRule
```
@available(iOS 10.0, *)
open class SKTileGroupRule : NSObject, NSCopying, NSCoding
     public init(adjacency: SKTileAdjacencyMask, tileDefinitions: [SKTileDefinition])
     open var adjacency: SKTileAdjacencyMask
     open var tileDefinitions: [SKTileDefinition]
     open var name: String?
```
- `init(adjacency: SKTileAdjacencyMask, tileDefinitions: [SKTileDefinition])` 根据瓦片链接和瓦片定义初始化瓦片组规则
- `adjacency` 瓦片链接
- `tileDefinitions` 瓦片规则
- `name` 瓦片组规则标识

##### SKTileDefinition
```
@available(iOS 10.0, *)
open class SKTileDefinition : NSObject, NSCopying, NSCoding
     public init(texture: SKTexture)
     public init(textures: [SKTexture], normalTextures: [SKTexture], size: CGSize, timePerFrame: CGFloat)
     open var userData: NSMutableDictionary?
     open var name: String?
     open var size: CGSize
     open var timePerFrame: CGFloat
     open var rotation: SKTileDefinitionRotation
     open var flipVertically: Bool
     open var flipHorizontally: Bool
```

- `init(texture: SKTexture)` 根据纹理初始化瓦片定义
- ` init(textures: [SKTexture], normalTextures: [SKTexture], size: CGSize, timePerFrame: CGFloat)` 根据纹理集合, 尺寸, 和帧率初始化瓦片定义
- `userData` 瓦片定义的用户数据
- `name` 瓦片定义的标识
- `timePerFrame` 瓦片定义的帧率
- `rotation` 瓦片定义的旋转规则
- `flipVertically` 是否垂直翻转
- `flipHorizontally` 是否水平翻转

#### 实战:

API, 了解一些基本的就够了, 如果要深究可以打开头文件逐个尝试, 我们现在就来实现一个小游戏, 这个游戏中包含了3个场景, 控制人物在规定时间内消灭所有的害虫, 我们着手进行游戏的开发吧!


![](http://upload-images.jianshu.io/upload_images/1229762-f837c5a6c01be613.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- step1 设置游戏场景的属性

```
class GameScene: SKScene {
  var background: SKTileMapNode! //背景瓦片地图节点
  var obstaclesTileMap: SKTileMapNode? //障碍物瓦片地图节点
  var bugsprayTileMap: SKTileMapNode? //杀虫喷剂瓦片地图节点

  var bugsNode = SKNode() //害虫的节点
  
  var player = Player() //玩家的节点
  var hud = HUD() //文字说明
  
  var firebugCount: Int = 0 //高级害虫的节点数

  var timeLimit: Int = 10 //时间限制
  var elapsedTime: Int = 0 //经过时间
  var startTime: Int? //开始时间

  var currentLevel: Int = 1 //当前关卡等级

  var gameState: GameState = .initial { //游戏状态默认为初始状态
    didSet { 
      hud.updateGameState(from: oldValue, to: gameState) //更新游戏状态
    }
  }
  ...
}
```
- step2 加载游戏场景的初始化设置

```
  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    background =
      childNode(withName: "background") as! SKTileMapNode //通过节点名读取背景瓦片地图节点
    obstaclesTileMap = childNode(withName: "obstacles") 
      as? SKTileMapNode //通过节点名读取障碍物瓦片地图节点

    if let timeLimit =
      userData?.object(forKey: "timeLimit") as? Int {
      self.timeLimit = timeLimit //通过节点的用户数据设置每个场景的时间限制
    }
    // 1
    let savedGameState = aDecoder.decodeInteger(forKey: "Scene.gameState") //解档保存游戏状态
    if let gameState = GameState(rawValue: savedGameState), gameState == .pause { //当解档保存游戏状态为暂停时
      self.gameState = gameState //赋值游戏状态
      firebugCount = aDecoder.decodeInteger(
        forKey: "Scene.firebugCount") //解档高级害虫数
      elapsedTime = aDecoder.decodeInteger(
        forKey: "Scene.elapsedTime") //解档经过时间
      currentLevel = aDecoder.decodeInteger(
        forKey: "Scene.currentLevel") //解档当前关卡等级
      // 2
      player = childNode(withName: "Player") as! Player //根据节点名读取玩家节点
      hud = camera!.childNode(withName: "HUD") as! HUD //根据节点名读取文字说明
      bugsNode = childNode(withName: "Bugs")! //根据节点名读取害虫节点
      bugsprayTileMap = childNode(withName: "Bugspray") 
        as? SKTileMapNode //通过节点名读取杀虫喷雾瓦片地图节点
    }

    addObservers() //添加观察者
  }

  deinit {
    NotificationCenter.default.removeObserver(self) //移除观察者
  }
```
- step3 当场景移动到屏幕时的设置

```
  override func didMove(to view: SKView) {
    if gameState == .initial { //当游戏状态为初始状态时
      addChild(player) //添加玩家到场景
      setupWorldPhysics() //添加物理世界
      createBugs() //添加害虫
      setupObstaclePhysics() //添加障碍物
      if firebugCount > 0 { //如果有高级害虫
        createBugspray(quantity: firebugCount + 10) //添加杀虫喷雾
      }
      setupHUD() //添加文字说明
      gameState = .start //设置游戏状态为开始状态
    }
    setupCamera() //添加摄像头
  }
```
- step4 物理世界的设置

```
  func setupWorldPhysics() {
    background.physicsBody =
      SKPhysicsBody(edgeLoopFrom: background.frame) //设置边缘物理体
    background.physicsBody?.categoryBitMask = PhysicsCategory.Edge //设置物理体标识为边缘
    
    physicsWorld.contactDelegate = self //物理世界代理

  }
```
- step5 创建害虫的设置

```
  func createBugs() {
    guard let bugsMap = childNode(withName: "bugs")
      as? SKTileMapNode else { return } //校验害虫瓦片地图节点
    // 1
    for row in 0..<bugsMap.numberOfRows { //逐行遍历害虫瓦片地图
      for column in 0..<bugsMap.numberOfColumns { //逐列遍历害虫瓦片地图
        // 2
        guard let tile = tile(in: bugsMap,
                              at: (column, row)) 
          else { continue } //校验瓦片地图中的每个瓦片
        // 3
        let bug: Bug 
        if tile.userData?.object(forKey: "firebug") != nil { //从用户数据中判断是否为高级害虫
          bug = Firebug() //将害虫设置为高级害虫
          firebugCount += 1 //高级害虫书自增
        } else {
          bug = Bug() //将害虫设置为普通害虫
        }
        bug.position = bugsMap.centerOfTile(atColumn: column,
                                            row: row) //从害虫瓦片地图中读取位置并赋值
        bugsNode.addChild(bug) //添加节点
        bug.moveBug() //移动害虫
      }
    }
    // 4
    bugsNode.name = "Bugs" //设置害虫节点标识
    addChild(bugsNode) //添加父节点到场景
    // 5
    bugsMap.removeFromParent() //删除害虫瓦片地图地图节点
  }
```
- step6 添加障碍物的设置

```
  func setupObstaclePhysics() {
    guard let obstaclesTileMap = obstaclesTileMap else { return } //校验障碍物瓦片地图节点
    // 1
    for row in 0..<obstaclesTileMap.numberOfRows {
      for column in 0..<obstaclesTileMap.numberOfColumns {
        // 2
        guard let tile = tile(in: obstaclesTileMap,
                              at: (column, row))
          else { continue }
        guard tile.userData?.object(forKey: "obstacle") != nil
          else { continue }
        // 3
        let node = SKNode() //创建节点
        node.physicsBody = SKPhysicsBody(rectangleOf: tile.size) 根据瓦片尺寸创建物理体
        node.physicsBody?.isDynamic = false //不进入物理世界
        node.physicsBody?.friction = 0 //摩擦系数为0
        node.physicsBody?.categoryBitMask =
          PhysicsCategory.Breakable //设置物理体标识
        
        node.position = obstaclesTileMap.centerOfTile(
          atColumn: column, row: row)
        obstaclesTileMap.addChild(node)
      }
    }
  }
```

- step7 添加杀虫喷雾的设置

```
  func createBugspray(quantity: Int) {
    // 1
    let tile = SKTileDefinition(texture:
      SKTexture(pixelImageNamed: "bugspray")) //创建瓦片定义
    // 2
    let tilerule = SKTileGroupRule(adjacency:
      SKTileAdjacencyMask.adjacencyAll, tileDefinitions: [tile]) //创建瓦片组规则
    // 3 
    let tilegroup = SKTileGroup(rules: [tilerule]) //创建瓦片组
    // 4
    let tileSet = SKTileSet(tileGroups: [tilegroup]) //创建瓦片集
    
    // 5
    let columns = background.numberOfColumns //读取背景瓦片地图节点的列数
    let rows = background.numberOfRows //读取背景瓦片地图节点的行数
    bugsprayTileMap = SKTileMapNode(tileSet: tileSet, 
                                    columns: columns,
                                    rows: rows,
                                    tileSize: tile.size) //创建新的瓦片地图节点
    // 6
    for _ in 1...quantity { 
      let column = Int.random(min: 0, max: columns-1) //随机列数
      let row = Int.random(min: 0, max: rows-1) //随机行数
      bugsprayTileMap?.setTileGroup(tilegroup,
                                    forColumn: column, row: row) //在新额的瓦片地图节点上随机生成瓦片组
    }
    // 7
    bugsprayTileMap?.name = "Bugspray" //设置瓦片地图节点的标识
    addChild(bugsprayTileMap!) //将瓦片地图添加到场景

  }
```
- step8 添加摄像头设置

```
  func setupCamera() {
    guard let camera = camera, let view = view else { return }
    
    let zeroDistance = SKRange(constantValue: 0)
    let playerConstraint = SKConstraint.distance(zeroDistance,
                                                 to: player) //对玩家进行约束
    // 1
    let xInset = min(view.bounds.width/2 * camera.xScale,
                     background.frame.width/2)
    let yInset = min(view.bounds.height/2 * camera.yScale,
                     background.frame.height/2)
    
    // 2
    let constraintRect = background.frame.insetBy(dx: xInset,
                                                  dy: yInset)
    // 3
    let xRange = SKRange(lowerLimit: constraintRect.minX,
                         upperLimit: constraintRect.maxX)
    let yRange = SKRange(lowerLimit: constraintRect.minY,
                         upperLimit: constraintRect.maxY)
    
    let edgeConstraint = SKConstraint.positionX(xRange, y: yRange)
    edgeConstraint.referenceNode = background
    // 4
    camera.constraints = [playerConstraint, edgeConstraint]
  }

```
- step9 获取瓦片的一些帮助方法

```
  func tile(in tileMap: SKTileMapNode,
            at coordinates: TileCoordinates)
    -> SKTileDefinition? {
      return tileMap.tileDefinition(atColumn: coordinates.column,
                                    row: coordinates.row)
  }

  func tileCoordinates(in tileMap: SKTileMapNode,
                       at position: CGPoint) -> TileCoordinates {
    let column = tileMap.tileColumnIndex(fromPosition: position)
    let row = tileMap.tileRowIndex(fromPosition: position)
    return (column, row)
  }

  func tileGroupForName(tileSet: SKTileSet, name: String)
    -> SKTileGroup? {
      let tileGroup = tileSet.tileGroups
        .filter { $0.name == name }.first
      return tileGroup
  }
```
- step10 点击场景的设置

```
  override func touchesBegan(_ touches: Set<UITouch>,
                             with event: UIEvent?) {
    guard let touch = touches.first else { return }
    switch gameState {
    // 1
    case .start: //开始状态
      gameState = .play //切换成游戏状态
      isPaused = false //开始
      startTime = nil
      elapsedTime = 0
    // 2
    case .play: //游戏状态
      player.move(target: touch.location(in: self)) //移动玩家
    case .win: //获胜状态
      transitionToScene(level: currentLevel + 1) //切换场景
    case .lose: //落败状态
      transitionToScene(level: 1) //切换场景
    case .reload: //唤醒状态
      // 1
      if let touchedNode =
        atPoint(touch.location(in: self)) as? SKLabelNode {
        // 2
        if touchedNode.name == HUDMessages.yes { //如果点击的节点是YES
          isPaused = false
          startTime = nil
          gameState = .play
          // 3
        } else if touchedNode.name == HUDMessages.no { //如果点击的节点是NO
          transitionToScene(level: 1)
        }
      }
    default:
      break
    }
  }

```
- step11 切换场景的设置

```
  func transitionToScene(level: Int) {
    // 1
    guard let newScene = SKScene(fileNamed: "Level\(level)")
      as? GameScene else {
        fatalError("Level: \(level) not found")
    }
    // 2
    newScene.currentLevel = level
    view!.presentScene(newScene,
                       transition: SKTransition.flipVertical(withDuration: 0.5))
  }
```

- step12 刷帧

```
  override func update(_ currentTime: TimeInterval) {
    if gameState != .play  {
      isPaused = true //如果不是游戏状态就暂停刷帧
      return
    }

    if !player.hasBugspray {
      updateBugspray() //如果玩家没有杀虫喷雾, 就进行更新
    }
    advanceBreakableTile(locatedAt: player.position) //更新障碍物的物理体状态
    updateHUD(currentTime: currentTime) //更新文字说明
    checkEndGame() //检查是否达到胜负条件
  }

```
- step13 更新杀虫喷雾

```
  func updateBugspray() {
    guard let bugsprayTileMap = bugsprayTileMap else { return }
    let (column, row) = tileCoordinates(in: bugsprayTileMap,
                                        at: player.position)
    if tile(in: bugsprayTileMap, at: (column, row)) != nil {
      bugsprayTileMap.setTileGroup(nil, forColumn: column,
                                   row: row)
      player.hasBugspray = true
    }
  }
```
- step14 更新障碍物的物理体状态

```
  func advanceBreakableTile(locatedAt nodePosition: CGPoint) {
    guard let obstaclesTileMap = obstaclesTileMap else { return }
    // 1
    let (column, row) = tileCoordinates(in: obstaclesTileMap,
                                        at: nodePosition)
    // 2
    let obstacle = tile(in: obstaclesTileMap,
                        at: (column, row))
    //3
    guard let nextTileGroupName =
      obstacle?.userData?.object(forKey: "breakable") as? String
      else { return }
    // 4
    if let nextTileGroup =
      tileGroupForName(tileSet: obstaclesTileMap.tileSet,
                       name: nextTileGroupName) {
      obstaclesTileMap.setTileGroup(nextTileGroup, 
                                    forColumn: column, row: row) //设置新的瓦片组到瓦片地图中
    }
  }
```
- step15 更新文字说明

```
  func updateHUD(currentTime: TimeInterval) {
    // 1
    if let startTime = startTime {
      // 2
      elapsedTime = Int(currentTime) - startTime
    } else {
      // 3
      startTime = Int(currentTime) - elapsedTime
    }
    // 4
    hud.updateTimer(time: timeLimit - elapsedTime) //对文字说明进行更新
  }
```

- step16 检查是否达到胜负条件

```
  func checkEndGame() {
    if bugsNode.children.count == 0 { //是否消灭全部害虫
      player.physicsBody?.linearDamping = 1
      gameState = .win
    } else if timeLimit - elapsedTime <= 0 { //是否时间用完
      player.physicsBody?.linearDamping = 1
      gameState = .lose
    }
  }
```
- step17 物理世界代理的设置

```
extension GameScene : SKPhysicsContactDelegate {
  func remove(bug: Bug) { //消灭害虫
    bug.removeFromParent()
    background.addChild(bug)
    bug.die()
    hud.updateBugCount(with: bugsNode.children.count)
  }

  func didBegin(_ contact: SKPhysicsContact) {
    let other = contact.bodyA.categoryBitMask
      == PhysicsCategory.Player ?
        contact.bodyB : contact.bodyA
    
    switch other.categoryBitMask {
    case PhysicsCategory.Bug:
      if let bug = other.node as? Bug {
        remove(bug: bug) //当玩家接触到普通害虫, 消灭普通害虫
      }
    case PhysicsCategory.Firebug:
      if player.hasBugspray {
        if let firebug = other.node as? Firebug {
          remove(bug: firebug)
          player.hasBugspray = false //当玩家手持杀虫喷雾接触高级害虫才能消灭高级害虫
        }
      }
    case PhysicsCategory.Breakable:
      if let obstacleNode = other.node {
        // 1
        advanceBreakableTile(locatedAt: obstacleNode.position) //更新障碍物
        // 2
        obstacleNode.removeFromParent() //删除原障碍物
      }

    default:
      break
    }
    if let physicsBody = player.physicsBody {
      if physicsBody.velocity.length() > 0 {
        player.checkDirection() //进行玩家方向的设置
      }
    }
  }
}

```

- step18 观察者的设置

```
extension GameScene {
  func applicationDidBecomeActive() {
    print("* applicationDidBecomeActive")
    if gameState == .pause {
      gameState = .reload //重新进入, 进行游戏重载
    }
  }
  
  func applicationWillResignActive() {
    print("* applicationWillResignActive")
    isPaused = true
    if gameState != .lose {
      gameState = .pause  //暂停游戏进程
    }
  }
  
  func applicationDidEnterBackground() {
    print("* applicationDidEnterBackground")
    if gameState != .lose {
      saveGame() //进入后台保存游戏进度
    }
  }

  func addObservers() {
    NotificationCenter.default.addObserver(self,
                                           selector: #selector(applicationDidBecomeActive),
                                           name: .UIApplicationDidBecomeActive, object: nil)
    NotificationCenter.default.addObserver(self,
                                           selector: #selector(applicationWillResignActive),
                                           name: .UIApplicationWillResignActive, object: nil)
    NotificationCenter.default.addObserver(self,
                                           selector: #selector(applicationDidEnterBackground),
                                           name: .UIApplicationDidEnterBackground, object: nil)
  }

}
```

- step19 游戏的存储设置

```
extension GameScene {
  func saveGame() {
    // 1
    let fileManager = FileManager.default
    guard let directory =
      fileManager.urls(for: .libraryDirectory,
                       in: .userDomainMask).first
      else { return }
    // 2
    let saveURL = directory.appendingPathComponent("SavedGames")
    // 3
    do {
      try fileManager.createDirectory(atPath: saveURL.path,
                                      withIntermediateDirectories: true,
                                      attributes: nil)
    } catch let error as NSError {
      fatalError(
        "Failed to create directory: \(error.debugDescription)")
    }
    // 4
    let fileURL = saveURL.appendingPathComponent("saved-game")
    print("* Saving: \(fileURL.path)")
    // 5
    NSKeyedArchiver.archiveRootObject(self, toFile: fileURL.path) //文件处理器新建路径并归档
  }
  
  override func encode(with aCoder: NSCoder) { /对关键属性的归档
    aCoder.encode(firebugCount,
                  forKey: "Scene.firebugCount")
    aCoder.encode(elapsedTime,
                  forKey: "Scene.elapsedTime")
    aCoder.encode(gameState.rawValue,
                  forKey: "Scene.gameState")
    aCoder.encode(currentLevel,
                  forKey: "Scene.currentLevel")
    super.encode(with: aCoder)
  }

  class func loadGame() -> SKScene? { //重新加载存储游戏进程
    print("* loading game")
    var scene: SKScene?
    // 1
    let fileManager = FileManager.default
    guard let directory =
      fileManager.urls(for: .libraryDirectory,
                       in: .userDomainMask).first
      else { return nil }
    // 2
    let url = directory.appendingPathComponent(
      "SavedGames/saved-game")
    // 3
    if FileManager.default.fileExists(atPath: url.path) {
      scene = NSKeyedUnarchiver.unarchiveObject(  //根据路径进行解档游戏进程
        withFile: url.path) as? GameScene
      _ = try? fileManager.removeItem(at: url)
    }
    return scene
  }

}

```

Notice: 忽略了一些节点的设置, 但不影响瓦片地图的理解.

##### 演示效果:

![](http://upload-images.jianshu.io/upload_images/1229762-be56289d668241e4.gif?imageMogr2/auto-orient/strip)

##### About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
