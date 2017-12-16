---
layout: post
title: CoreData SQL写烦了? 试试亲儿子!
date: 2017.06.08 15:52
tag: 移动开发
---

> 目前来说 iOS的数据库一般都使用FMDB, 之前看了下挺火的Rleam, 以我目前的水平还分不清楚各种移动数据库孰优孰劣, 以一般覆盖80%的页面缓存, 个人愚见使用写.plist的方式最为简单有效, 但作为iOS开发者, 怎么能不试试亲儿子Core Data呢?

![](http://upload-images.jianshu.io/upload_images/1229762-f0dbc8384ac301fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从来没有看过Core Data的我, 只是道听途说底层也是SQLite, 性能不好, 学习曲线陡峭, 诶... 听起来就很丧, 但作为亲儿子, 如此不成气候, 莫非是地主家的傻儿子? 从iOS3 ~ iOS10 Apple老爹还一直在推, ObjC.io 和 Raywenderlich的亲戚朋友们还一直帮忙出书? 素以逻辑严谨的我表示百思不得其解, 带着这份疑惑, 我也就毅然决然的踏上了Core Data踩坑之旅.

#### Core Data 架构

一个基本的 Core Data 栈由四个主要部分组成：托管对象 (managed objects) (NSManagedObject)，托管对象上下文 (managed object context) (NSManagedObjectContext)，持久化存储协调器 (persistent store coordinator) (NSPersistentStoreCoordinator)，以及持久化存储 (persistent store) (NSPersistentStore), 我们分别看下对应的API.

###### NSManagedObject:
NSManagedObject以我理解, 就如同WWDC2017中开放的Core ML的MLModel一样应该是一个模型对象, 用以缓存数据的模型, Core Data中能缓存的数据只有继承自NSManagedObject的才可以, 和Rleam一样需要对模型对象进行标注, 这也是我对Rleam无感的原因 -- 对第三方的耦合性太强了!!
```
@available(iOS 3.0, *)
open class NSManagedObject : NSObject 
@available(iOS 10.0, *)
    public init(entity: NSEntityDescription, insertInto context: NSManagedObjectContext?) //根据实体描述及托管对象上下文创建托管对象
@available(iOS 10.0, *)
    open class func fetchRequest() -> NSFetchRequest<NSFetchRequestResult> //返回当前查询对象
    public init(entity: NSEntityDescription, insertInto context: NSManagedObjectContext?) //通过实体及上下文进行初始化
@available(iOS 10.0, *)
    public convenience init(context moc: NSManagedObjectContext) //通过上下文进行初始化
    unowned(unsafe) open var managedObjectContext: NSManagedObjectContext? { get } //当前对象的上下文
    open var objectID: NSManagedObjectID { get } //当前对象ID
@available(iOS 5.0, *)
    open var hasChanges: Bool { get } //当前对象中是否有属性改变
    open func value(forKey key: String) -> Any? //KVC取值
    open func setValue(_ value: Any?, forKey key: String) //KVC赋值
```
上述具体分析`NSManagedObject`中的关键属性及方法, 可以看到的是, iOS10新增的API的数量还是很多的, 这也侧边证明了老爹对儿子的关心与疼爱.

###### NSManagedObjectContext:
```
@available(iOS 3.0, *)
open class NSManagedObjectContext : NSObject, NSCoding, NSLocking 
@available(iOS 5.0, *)
    open var parent: NSManagedObjectContext? //父级上下文
    open var hasChanges: Bool { get } //上下文是否有更改
    open func save() throws //保存上下文
    open func insert(_ object: NSManagedObject) //将托管对象插入上下文
    open func delete(_ object: NSManagedObject) //将托管对象从上下文中删除
@available(iOS 8.0, *)
    open func execute(_ request: NSPersistentStoreRequest) throws -> NSPersistentStoreResult //执行获取请求
    public func fetch<T : NSFetchRequestResult>(_ request: NSFetchRequest<T>) throws -> [T] //获取数据
    @available(iOS 3.0, *)
```
###### NSPersistentStoreCoordinator:
```
@available(iOS 3.0, *)
open class NSPersistentStoreCoordinator : NSObject, NSLocking 
   open var persistentStores: [NSPersistentStore] { get } //持续化存储集
```
###### NSPersistentStore:
```
@available(iOS 3.0, *)
open class NSPersistentStore : NSObject
```

#### Core Data 基本概念

我们创建一个Core Data文件 后缀名为.xcdatamodeld, 图形化界面配置存储数据, 由于是基本使用, 就简单的创建一个Person实体, Person实体中有一个name的字符串, 如图:
![Core Data 文件 Person实体中添加一个name属性 简单设置](http://upload-images.jianshu.io/upload_images/1229762-e5dd9602ce3726f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### NSPersistentContainer:
```
@available(iOS 10.0, *)
open class NSPersistentContainer : NSObject
open var viewContext: NSManagedObjectContext { get } //托管对象上下文
public convenience init(name: String) //通过Bundle中的名称来创建持久化容器
open func loadPersistentStores(completionHandler block: @escaping (NSPersistentStoreDescription, Error?) -> Swift.Void) //加载持久化存储, 完成回调
open func performBackgroundTask(_ block: @escaping (NSManagedObjectContext) -> Swift.Void) //后台线程执行
```
在此, 我们需要事先了解下iOS10中Core Data的新特性`NSPersistentContainer`

###### NSEntityDescription:
```
@available(iOS 3.0, *)
open class NSEntityDescription : NSObject, NSCoding, NSCopying, NSFastEnumeration
open class func entity(forEntityName entityName: String, in context: NSManagedObjectContext) -> NSEntityDescription? //通过实例名及上下文获取对应实例描述
```

###### NSFetchRequest:
```
@available(iOS 3.0, *)
open class NSFetchRequest<ResultType : NSFetchRequestResult> : NSPersistentStoreRequest, NSCoding
@available(iOS 4.0, *)
    public convenience init(entityName: String) //通过实例名创建抓取请求
    open var predicate: NSPredicate? //谓词, 用于查询过滤
    open var sortDescriptors: [NSSortDescriptor]? //进行排序
    open var propertiesToFetch: [Any]? //属性抓取
@available(iOS 3.0, *)
    open var resultType: NSFetchRequestResultType //返回的结果类型
```

###### 简易 Core Data Stack
```
  //懒加载一个持久化容器属性 
  lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "HitList") //根据名称取出对应的持久化容器
    container.loadPersistentStores(completionHandler: { (storeDescription, error) in //加载持久化存储
      if let error = error as NSError? {
        fatalError("Unresolved error \(error), \(error.userInfo)")
      }
    })
    return container
  }()
```
###### 实现 Core Data Stack
```
class CoreDataStack { //创建CoreDataStack类以备复用

  private let modelName: String //CoreData模型名称, 对应.xcdatamodeld文件名

  init(modelName: String) {
    self.modelName = modelName //初始化方法并赋值
  }

  lazy var managedContext: NSManagedObjectContext = { //懒加载托管对象上下文
    return self.storeContainer.viewContext
  }()

  private lazy var storeContainer: NSPersistentContainer = { //懒加载持续化容器

    let container = NSPersistentContainer(name: self.modelName)
    container.loadPersistentStores { (storeDescription, error) in
      if let error = error as NSError? {
        print("Unresolved error \(error), \(error.userInfo)")
      }
    }
    return container
  }()

  func saveContext () { //保存托管对象上下文
    guard managedContext.hasChanges else { return } //校验是否上下文有所改变

    do {
      try managedContext.save() //进行保存
    } catch let error as NSError {
      print("Unresolved error \(error), \(error.userInfo)")
    }
  }
}
```

###### 简单演示如何存储: 
```
  var people: [NSManagedObject] = [] //创建一个people数组, 数组内部都是托管对象Person实例
```
```   
  //读取数据
  let fetchRequest = NSFetchRequest<NSManagedObject>(entityName: "Person") //通过实例名字抓取数据
  do {
    people = try managedContext.fetch(fetchRequest) //通过托管对象上下文获取数据并赋值给people数组
  } catch let error as NSError {
    print("Could not fetch. \(error), \(error.userInfo)")
  }
```
```
  //存储数据
  let entity = NSEntityDescription.entity(forEntityName: "Person", in: managedContext)! //通过实例名及托管对象上下文获取对应实例描述
  let person = NSManagedObject(entity: entity, insertInto: managedContext) //通过实例及托管对象上下文创建托管对象
  person.setValue(name, forKeyPath: "name") //通过KVC将新值赋值给name属性
  do {
    try managedContext.save() //保存托管对象上下文
    people.append(person) // 将实例添加至实例数组
  } catch let error as NSError { //如果存储失败抛出异常
    print("Could not save. \(error), \(error.userInfo)")
  }
```
这样我们就通过读取和存储, 了解了Core Data的基本概念了, 比起写SQL还是更有爱一点~

#### Core Data 模型制作
上一个例子中, 我们通过KVC的方式取值和赋值, 但我们的托管对象不可能永远只有一个属性, 所以通过字符串Key来取值的方法往往不是最佳方案, 我们看看如何更好地面向对象, 我们先使用图形化界面生成Core Data模型.


![Core Data 文件, 添加了很多属性, 其中Type为Transformable 是指可转变为其他类型](http://upload-images.jianshu.io/upload_images/1229762-994472315fecb1fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![通过Editor 可以进行创建对应的swift文件映射, 和IB很类似](http://upload-images.jianshu.io/upload_images/1229762-465ea4433fe427c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![只需要class 类名匹配即可](http://upload-images.jianshu.io/upload_images/1229762-12eb5d0d557eeb01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以如图自动创建文件, 但ObjC.io的建议是自行输入为上, 因为并不是很神奇的东西, 只需要class 类名匹配即可. 生成的文件可以自行输入, Bowtie是继承自NSManagedObject的托管对象.

```
extension Bowtie {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<Bowtie> {
        return NSFetchRequest<Bowtie>(entityName: "Bowtie");
    }

    @NSManaged public var isFavorite: Bool
    @NSManaged public var lastWorn: NSDate?
    @NSManaged public var name: String?
    @NSManaged public var photoData: NSData?
    @NSManaged public var searchKey: String?
    @NSManaged public var timesWorn: Int32
    @NSManaged public var tintColor: NSObject?
    @NSManaged public var rating: Double
}
```
我们可以看到自动生成了图形化工具中的全部属性, 并创建了抓取函数, 方便使用. 注意之前的Type为Transformable, 生成出来就变成了NSObject?, 可以分析得知, 可转变顾名思义, 所有类型都是继承自NSObject, 当然是对于Objetcive-C来说, 不过对于Foundation框架中的类, Swift也是如此.

#### Core Data 关系映射
学会了制作Core Data模型后, 我们就要来谈谈各个实例之间的关系了, 各个实例化之间的关系也可以通过图形化界面来实现:

![Dog 实例中 与Walk实例的关系 Type 设置为 To Many 作为集合](http://upload-images.jianshu.io/upload_images/1229762-bf718c7920145bae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Walk 实例中 与Dog实例的关系 Type 设置为 To One 作为属性](http://upload-images.jianshu.io/upload_images/1229762-e011fd70b386aafe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![设置Inverse, 隐射两个实体之间的对应关系, 图示双向箭头表示集合嵌套](http://upload-images.jianshu.io/upload_images/1229762-c511b90e350d2970.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### RelationShips Core Data 模型
```
extension Dog {

  @nonobjc public class func fetchRequest() -> NSFetchRequest<Dog> {
    return NSFetchRequest<Dog>(entityName: "Dog");
  }

  @NSManaged public var name: String?
  @NSManaged public var walks: NSOrderedSet? //这里不是数组类型, 而是特殊的集合类型
}

// MARK: Generated accessors for walks
extension Dog { //由于RelationShips关系映射会自动生成集合增删改查逻辑

  @objc(insertObject:inWalksAtIndex:)
  @NSManaged public func insertIntoWalks(_ value: Walk, at idx: Int)

  @objc(removeObjectFromWalksAtIndex:)
  @NSManaged public func removeFromWalks(at idx: Int)

  @objc(insertWalks:atIndexes:)
  @NSManaged public func insertIntoWalks(_ values: [Walk], at indexes: NSIndexSet)

  @objc(removeWalksAtIndexes:)
  @NSManaged public func removeFromWalks(at indexes: NSIndexSet)

  @objc(replaceObjectInWalksAtIndex:withObject:)
  @NSManaged public func replaceWalks(at idx: Int, with value: Walk)

  @objc(replaceWalksAtIndexes:withWalks:)
  @NSManaged public func replaceWalks(at indexes: NSIndexSet, with values: [Walk])

  @objc(addWalksObject:)
  @NSManaged public func addToWalks(_ value: Walk)

  @objc(removeWalksObject:)
  @NSManaged public func removeFromWalks(_ value: Walk)

  @objc(addWalks:)
  @NSManaged public func addToWalks(_ values: NSOrderedSet)

  @objc(removeWalks:)
  @NSManaged public func removeFromWalks(_ values: NSOrderedSet)
}
```
```
extension Walk {

  @nonobjc public class func fetchRequest() -> NSFetchRequest<Walk> {
    return NSFetchRequest<Walk>(entityName: "Walk");
  }

  @NSManaged public var date: NSDate?
  @NSManaged public var dog: Dog?
}
```
看到生成后的两个实体之间的关系也很好理解, 只是未显示NSOrderedSet中包含的实体类型, 这点在Swift4中已经有改善

#### Core Data 获取操作

之前使用数据库的时候, 我们都会使用SQL查询语句 , select 字段 from 表 where 筛选条件, 进行数据的过滤, 而我们使用Core Data的时候用什么方法呢? 首先我们可以先了解以下类:

###### NSPredicate 谓词用于筛选过滤
```
@available(iOS 3.0, *)
open class NSPredicate : NSObject, NSSecureCoding, NSCopying 
    public /*not inherited*/ init(format predicateFormat: String, argumentArray arguments: [Any]?) //谓词格式, 用以过滤信息
```

###### NSSortDescriptor 用于数据的排序
```
open class NSSortDescriptor : NSObject, NSSecureCoding, NSCopying 
    public init(key: String?, ascending: Bool) //通过key值, 进行升序或降序的排序
```

###### NSExpressionDescription 对没有属性名的数据操作
```
  func populateDealsCountLabel() {

    let fetchRequest = NSFetchRequest<NSDictionary>(entityName: "Venue")
    fetchRequest.resultType = .dictionaryResultType //将返回结果设置为字典类型

    let sumExpressionDesc = NSExpressionDescription() //由于是字典类型并没有再图形化界面中设置, 故需要使用NSExpressionDescription
    sumExpressionDesc.name = "sumDeals"

    let specialCountExp = NSExpression(forKeyPath: #keyPath(Venue.specialCount)) //根据KVC获取属性值
    sumExpressionDesc.expression = NSExpression(forFunction: "sum:", arguments: [specialCountExp])
    sumExpressionDesc.expressionResultType = .integer32AttributeType

    fetchRequest.propertiesToFetch = [sumExpressionDesc] //进行获取

    do {
      let results = try coreDataStack.managedContext.fetch(fetchRequest)
      let resultDict = results.first!
      let numDeals = resultDict["sumDeals"]!
      numDealsLabel.text = "\(numDeals) total deals"

    } catch let error as NSError {
      print("Count not fetch \(error), \(error.userInfo)")
    }
  }
```
###### NSBatchUpdateRequest & NSAsynchronousFetchRequest 批量更新和异步更新
```
  override func viewDidLoad() {
    super.viewDidLoad()

    let batchUpdate = NSBatchUpdateRequest(entityName: "Venue") //初始化批量更新
    batchUpdate.propertiesToUpdate = [#keyPath(Venue.favorite) : true] //更新的属性
    batchUpdate.affectedStores = coreDataStack.managedContext.persistentStoreCoordinator?.persistentStores //持久化存储
    batchUpdate.resultType = .updatedObjectsCountResultType //设置类型

    do {
      let batchResult = try coreDataStack.managedContext.execute(batchUpdate) as! NSBatchUpdateResult
      print("Records updated \(batchResult.result!)")
    } catch let error as NSError {
      print("Could not update \(error), \(error.userInfo)")
    }
    
    fetchRequest = Venue.fetchRequest()

    asyncFetchRequest = NSAsynchronousFetchRequest<Venue>(fetchRequest: fetchRequest) { [unowned self] (result: NSAsynchronousFetchResult) in //异步获取

      guard let venues = result.finalResult else {
        return
      }

      self.venues = venues
      self.tableView.reloadData()
    }

    do {
      try coreDataStack.managedContext.execute(asyncFetchRequest)
    } catch let error as NSError {
      print("Could not fetch \(error), \(error.userInfo)")
    }
  }
```

#### Core Data 与 TableView 协作
Core Data中有一个类是和列表视图度身订造的, 那就是NSFetchedResultsController, 可以将完美的与TableView进行结合, 我们先来看下常用的方法:

###### NSFetchedResultsController
```
@available(iOS 3.0, *)
open class NSFetchedResultsController<ResultType : NSFetchRequestResult> : NSObject 
  public init(fetchRequest: NSFetchRequest<ResultType>, managedObjectContext context: NSManagedObjectContext, sectionNameKeyPath: String?, cacheName name: String?) //初始化
  open func performFetch() throws //获取
  unowned(unsafe) open var delegate: NSFetchedResultsControllerDelegate? //代理
  open func object(at indexPath: IndexPath) -> ResultType //获取特定对象

public protocol NSFetchedResultsSectionInfo 
  public var numberOfObjects: Int { get } //获取对象数量
```
其中sectionNameKeyPath参数是传入根据什么字段进行分组, 而cacheName, 则是进行内存缓存.

###### 简单演示如何操作:
```
  override func viewDidLoad() {
    super.viewDidLoad()

    let fetchRequest: NSFetchRequest<Team> = Team.fetchRequest()

    let zoneSort = NSSortDescriptor(key: #keyPath(Team.qualifyingZone), ascending: true)
    let scoreSort = NSSortDescriptor(key: #keyPath(Team.wins), ascending: false)
    let nameSort = NSSortDescriptor(key: #keyPath(Team.teamName), ascending: true)

    fetchRequest.sortDescriptors = [zoneSort, scoreSort, nameSort] //进行三层排序

    fetchedResultsController = NSFetchedResultsController(fetchRequest: fetchRequest,
                                                          managedObjectContext: coreDataStack.managedContext,
                                                          sectionNameKeyPath: #keyPath(Team.qualifyingZone),
                                                          cacheName: "worldCup")

    fetchedResultsController.delegate = self //设置代理

    do {
      try fetchedResultsController.performFetch() //进行获取数据
    } catch let error as NSError {
      print("Fetching error: \(error), \(error.userInfo)")
    }
  }
```
```
// MARK: - UITableViewDataSource
extension ViewController: UITableViewDataSource {

  func numberOfSections(in tableView: UITableView) -> Int {
    guard let sections = fetchedResultsController.sections else { //section的数量
      return 0
    }

    return sections.count
  }

  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {

    guard let sectionInfo = fetchedResultsController.sections?[section] else { //row的数量
      return 0
    }

    return sectionInfo.numberOfObjects
  }

  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

    let cell = tableView.dequeueReusableCell(withIdentifier: teamCellIdentifier, for: indexPath)
    guard let cell = cell as? TeamCell else {
      return
    }

    let team = fetchedResultsController.object(at: indexPath) //拿到对应cell的数据
    cell.flagImageView.image = UIImage(named: team.imageName!)
    cell.teamLabel.text = team.teamName
    cell.scoreLabel.text = "Wins: \(team.wins)"
    return cell
  }

  func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
    let sectionInfo = fetchedResultsController.sections?[section]
    return sectionInfo?.name //拿到对象的名字
  }
}

// MARK: - UITableViewDelegate
extension ViewController: UITableViewDelegate {

  func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {

    let team = fetchedResultsController.object(at: indexPath)
    team.wins = team.wins + 1 //修改数据
    coreDataStack.saveContext() //存入Core Data
  }
}

```
```
extension ViewController: NSFetchedResultsControllerDelegate {

  //当上下文更改开始前调用
  func controllerWillChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
    tableView.beginUpdates() //reloadData自定义方法开始
  }

  //当托管对象进行改变时调用
  func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>, didChange anObject: Any, at indexPath: IndexPath?, for type: NSFetchedResultsChangeType, newIndexPath: IndexPath?) {

    switch type {
    case .insert:
      tableView.insertRows(at: [newIndexPath!], with: .automatic)
    case .delete:
      tableView.deleteRows(at: [indexPath!], with: .automatic)
    case .update:
      let cell = tableView.cellForRow(at: indexPath!) as! TeamCell
      configure(cell: cell, for: indexPath!)
    case .move:
      tableView.deleteRows(at: [indexPath!], with: .automatic)
      tableView.insertRows(at: [newIndexPath!], with: .automatic)
    }
  }
  
  //当上下文更改结束后调用
  func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
    tableView.endUpdates()  //reloadData自定义方法结束
  }

  //当组信息进行改变的时候调用
  func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>, didChange sectionInfo: NSFetchedResultsSectionInfo, atSectionIndex sectionIndex: Int, for type: NSFetchedResultsChangeType) {

    let indexSet = IndexSet(integer: sectionIndex)

    switch type {
    case .insert:
      tableView.insertSections(indexSet, with: .automatic)
    case .delete:
      tableView.deleteSections(indexSet, with: .automatic)
    default: break
    }
  }
}
```
以上就是Core Data的学习概要, 当然还有更重要的Core Data的迁移及父子上下文和线程并发问题等没有讲到, 当然这些深层的东西还需要慢慢的研究, 还是之前的一句话 -- 一般覆盖80%的页面缓存, 个人愚见使用写.plist的方式最为简单有效.

 [🌟 想要关注作者 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
