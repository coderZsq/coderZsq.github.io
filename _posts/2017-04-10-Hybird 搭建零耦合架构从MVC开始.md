---
layout: post
title: Hybird 搭建零耦合架构从MVC开始
date: 2017.04.10 17:17
tag: 移动开发
---

> 写这个target的想法是源于接下来可能要接的新项目, 开发小组的成员们都要表达出一些对架构设计的想法, 自然平时懒散的我也不能例外, 所以这个系列会基于我浅显的知识来表达出我对于架构设计的一些思考, 和大家分享, 代码详见: [github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-3866097366648136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


说到热修复, 最近JSPatch跪下来叫爸爸的经历还历历在目, 那我们真的有线上Bug就只能发紧急版本了吗? 虽然说审核比以前快了但时间也不短啊, 我们看如何通过架构设计来实现热修复吧!

首先我们先创建一个目录结构:
.
├── app
├── server
└── web

目录结构包括移动端, 前端和后端, 对应用到的技术栈: Swift3, Vue2, Koa2.

作为一个iOS开发者, 我们还是从app入手, 我们创建一个名为RouterPatterm的工程, 语言选择为Swift, 对于app的架构设计, 耳熟能详的 MVC, MVVM, MVP 之类的, 我们也不免俗的引用前人的思想:

MVC

MVC 也就是 Model, Controller, View之前进行交互的设计模式, 是一种比较经典的架构设计吧, 苹果的Cocoa框架也是基于MVC设计模式的, 也是平时开发最常用到的设计了. 我们与之对应先创建一个目录:
.
├── Controller.swift
├── Model.swift
└── View.swift

在AppDelegate中将Controller设为根控制器

```

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        window?.rootViewController = UINavigationController(rootViewController: Controller())
        return true
    }
```

Model.swift

```
class Model: NSObject {

    var text : String = ""
    var detailText : String = ""
}
```

View.swift

```

let kScreenW = UIScreen.main.bounds.width
let kScreenH = UIScreen.main.bounds.height

class View: UIView {
    
    var models: [Model]? {
        didSet {
            tableView.reloadData()
        }
    }
    
    fileprivate lazy var tableView: UITableView = { [weak self] in
        var tableView = UITableView(frame: self!.bounds, style: .plain)
        tableView.dataSource = self
        return tableView
        }()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        addSubview(tableView)
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

extension View: UITableViewDataSource {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return models?.count ?? 0
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        let identifier = "identifier"
        let model: Model? = models?[indexPath.row]
        let cell = tableView.dequeueReusableCell(withIdentifier: identifier) ?? UITableViewCell(style: .subtitle, reuseIdentifier: identifier)
        cell.textLabel?.text = model?.text
        cell.detailTextLabel?.text = model?.detailText
        return cell
    }
}


```


Controller.swift

```

class Controller: UIViewController {

    fileprivate lazy var models: [Model] = {
        var models = [Model]()
        
        for i in 0...20 {
            var model = Model()
            model.text = "text -- \(i)"
            model.detailText = "detailText -- \(i)"
            models.append(model)
        }
        
        return models
    }()
    
    fileprivate lazy var baseView: View = { [weak self] in
        return View(frame: self!.view.bounds)
    }()
    
    override func loadView() {
        super.loadView()
        title = "J1"
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupView()
        adapterView()
    }
}

extension Controller {
    
    fileprivate func setupView() {
        view.addSubview(baseView)
    }
    
    fileprivate func adapterView() {
        baseView.models = models
    }
}

```
这可能是最朴实无华的MVC了吧, Model 负责数据, View负责视图, Controller负责视图和数据间的通信.

但这些都只是模拟出来的假数据, app开发工程师只会做个壳子肯定不是我们的追求, 所以, 我们需要自己搭建服务器来进行交互, 这里我就使用Koa进行简单的演示.

[Koa](http://koajs.com/) 对于有些iOS开发可能会觉得有些陌生, Koa是基于Node.js的一个开发框架, 熟悉Node的同学想必一定知道Express这个框架了, Koa是Express原班人马打造的先驱的服务端框架, 使用最新ES6/ES7的语法, 通过中间件的形式极大的解决了回调地狱的问题.

介绍就那么多, 我们的关注点还是在如何搭建服务器, 首先我们需要Node环境, 没有的同学[点击这里](https://nodejs.org/en/), 下载最新的稳定版即可, 已有的也请更新到最新版本, 不然可能会造成新语法在旧环境不兼容的问题.

有了Node环境, 我们就可以进行服务端的开发了, 首先我们先找一个Koa的生成器, [官方Wiki](https://github.com/koajs/koa/wiki)上有很多推荐, 练手的话随便选一个就好, 我选择的是koa-generator.

1. 安装 $ npm install -g koa-generator
2. 生成 $ koa2 RouterPatterm
3. 加载 $ cd RouterPatterm $ npm install
4. 启动 $ npm start

打开localhost:3000 就能看见服务器已经搭建完毕了.

可以看到基本的项目结构如下:
.
└── RouterPatterm
    ├── app.js
    ├── bin
    │   ├── run
    │   └── www
    ├── node_modules    
    ├── package.json
    ├── public
    │   ├── images
    │   ├── javascripts
    │   └── stylesheets
    │       └── style.css
    ├── routes
    │   ├── index.js
    │   └── users.js
    └── views
        ├── error.jade
        ├── index.jade
        └── layout.jade

彩蛋:
到这有些同学可能会问, 项目结构目录是怎么打印出来的:

1. $ brew install tree
2. $ tree -L n (n 代表你的遍历层级)


为了能够根据不同的运行环境加载不同的配置内容，我们需要添加一些配置文件。
首先在项目根目录下添加config目录，在config目录下添加index.js、test.js、development.js三个文件，内容如下。

development.js

```
/**
 * 开发环境的配置内容
 */

module.exports = {
    env: 'development', //环境名称
    port: 3001,         //服务端口号
    mongodb_url: '',    //数据库地址
    redis_url:'',       //redis地址
    redis_port: ''      //redis端口号
}
```
test.js

```
/**
 * 测试环境的配置内容
 */

module.exports = {
    env: 'test',        //环境名称
    port: 3801,         //服务端口号
    mongodb_url: '',    //数据库地址
    redis_url:'',       //redis地址
    redis_port: ''      //redis端口号
}
```
index.js

```
var development_env = require('./development');
var test_env = require('./test');

//根据不同的NODE_ENV，输出不同的配置对象，默认输出development的配置对象
module.exports = {
    development: development_env,
    test: test_env
}[process.env.NODE_ENV || 'development']
```

代码应该都没什么可解释的，然后我们再来编辑bin/www文件。

bin/www添加如下代码

```
//引入配置文件
var config = require('../config');

// 将端口号设置为配置文件的端口号，默认值为3000
var port = normalizePort(config.port || '3000');
// 打印输出端口号
console.log('port = ' + config.port);
```

测试效果，在终端输入npm start，可以看到

process.env.NODE_ENV=development
port = 3001
到浏览器中访问http://localhost:3001，可以看到原来的输入内容，说明配置文件已经生效。

配置好环境, 我们就要提供API来给客户端调用了, 在他们的世界称为RESTFUL. 
首先我们创建一个controller目录:

app
└── controllers
    └── J1.js

来对应客户端的J1控制器

J1.js

```
exports.getJ1List = async(ctx, next) => {

    ctx.body = {
        models: [{
            text: '我的账户',
            detailText: "欢迎进入=>我的账户",
        }, {
            text: '我的优惠券',
            detailText: "欢迎进入=>我的优惠券",
        }, {
            text: '收货地址',
            detailText: "欢迎进入=>收货地址",
        }, {
            text: '在线客服',
            detailText: "欢迎进入=>在线客服",
        }, {
            text: '用药提醒',
            detailText: "欢迎进入=>用药提醒",
        }, {
            text: '药查查',
            detailText: "欢迎进入=>药查查",
        }, {
            text: '疾病百科',
            detailText: "欢迎进入=>疾病百科",
        }, {
            text: '药品百科',
            detailText: "欢迎进入=>药品百科",
        }, {
            text: '健一咨询',
            detailText: "欢迎进入=>健一咨询",
        }, {
            text: '帮助中心',
            detailText: "欢迎进入=>帮助中心",
        }, {
            text: '点赞/吐槽',
            detailText: "欢迎进入=>点赞/吐槽",
        }, {
            text: '关于健一',
            detailText: "欢迎进入=>关于健一",
        }]
    }
}


```

接下来我们为API添加路由

域名 + 端口号 /api/功能类型/具体端口

.
├── api
│   ├── J1_router.js
│   └── index.js

index.js

```

var router = require('koa-router')();
var J1_router = require('./J1_router');

router.use('/J1', J1_router.routes(), J1_router.allowedMethods())

module.exports = router;

```

J1_router.js

```

var router = require('koa-router')();
var J1 = require('../../app/controllers/J1');

router.get('/getJ1List', J1.getJ1List);

module.exports = router;

```

app.js文件中添加如下代码：

```
const api = require('./routes/api');

router.use('/api', api.routes(), api.allowedMethods());
```

这时我们在浏览器中输入http://localhost:3001/api/J1/getJ1List后就能看见数据返回了.

```

{
  "models": [
    {
      "text": "我的账户",
      "detailText": "欢迎进入=>我的账户"
    },
    {
      "text": "我的优惠券",
      "detailText": "欢迎进入=>我的优惠券"
    },
    {
      "text": "收货地址",
      "detailText": "欢迎进入=>收货地址"
    },
    {
      "text": "在线客服",
      "detailText": "欢迎进入=>在线客服"
    },
    {
      "text": "用药提醒",
      "detailText": "欢迎进入=>用药提醒"
    },
    {
      "text": "药查查",
      "detailText": "欢迎进入=>药查查"
    },
    {
      "text": "疾病百科",
      "detailText": "欢迎进入=>疾病百科"
    },
    {
      "text": "药品百科",
      "detailText": "欢迎进入=>药品百科"
    },
    {
      "text": "健一咨询",
      "detailText": "欢迎进入=>健一咨询"
    },
    {
      "text": "帮助中心",
      "detailText": "欢迎进入=>帮助中心"
    },
    {
      "text": "点赞/吐槽",
      "detailText": "欢迎进入=>点赞/吐槽"
    },
    {
      "text": "关于健一",
      "detailText": "欢迎进入=>关于健一"
    }
  ]
}
```

接下来我们要做的是格式化输出, 毕竟我们现在返回的格式不是特别友好.

error
├── ApiError.js
└── ApiErrorNames.js

middlewares
└── response_formatter.js

response_formatter.js

```
var ApiError = require('../app/error/ApiError');

var response_formatter = (ctx) => {

    if (ctx.body) {
        ctx.body = {
            code: 0,
            message: 'success',
            data: ctx.body
        }
    } else {
        ctx.body = {
            code: 0,
            message: 'success'
        }
    }
}

var url_filter = (pattern) => {
    return async(ctx, next) => {
        var reg = new RegExp(pattern);
        try {
            await next();
        } catch (error) {
            if (error instanceof ApiError && reg.test(ctx.originalUrl)) {
                ctx.status = 200;
                ctx.body = {
                    code: error.code,
                    message: error.message
                }
            }
            throw error;
        }

        if (reg.test(ctx.originalUrl)) {
            response_formatter(ctx);
        }
    }
}
module.exports = url_filter;

```
最后将中间件挂载到koa上

```
const response_formatter = require('./middlewares/response_formatter');

app.use(response_formatter('^/api')); //在路由之前调用
``` 

好了, 我们再看下返回的格式:

```

{
  "code": 0,
  "message": "success",
  "data": {
    "models": [
      {
        "text": "我的账户",
        "detailText": "欢迎进入=>我的账户"
      },
      {
        "text": "我的优惠券",
        "detailText": "欢迎进入=>我的优惠券"
      },
      {
        "text": "收货地址",
        "detailText": "欢迎进入=>收货地址"
      },
      {
        "text": "在线客服",
        "detailText": "欢迎进入=>在线客服"
      },
      {
        "text": "用药提醒",
        "detailText": "欢迎进入=>用药提醒"
      },
      {
        "text": "药查查",
        "detailText": "欢迎进入=>药查查"
      },
      {
        "text": "疾病百科",
        "detailText": "欢迎进入=>疾病百科"
      },
      {
        "text": "药品百科",
        "detailText": "欢迎进入=>药品百科"
      },
      {
        "text": "健一咨询",
        "detailText": "欢迎进入=>健一咨询"
      },
      {
        "text": "帮助中心",
        "detailText": "欢迎进入=>帮助中心"
      },
      {
        "text": "点赞/吐槽",
        "detailText": "欢迎进入=>点赞/吐槽"
      },
      {
        "text": "关于健一",
        "detailText": "欢迎进入=>关于健一"
      }
    ]
  }
}

```

服务端终于告一段落, 我们再回到客户端, 我们使用Alamofire来进行网络请求.

对于iOS的包管理工具, 我们还是使用cocoapods, 虽然我并不是那么喜欢.

1. $ gem install cocoapods

2. $ pod init 

3. edit profile

4. $ pod install

5. $ pod update

第三步的编辑如下:

```

# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'RouterPatterm' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for RouterPatterm
     pod 'Alamofire'
     pod 'Kingfisher'
end


```
Kingfisher是喵神写的图片框架, 先导入以后备用.

我们在Alamofire的基础上再封装一层, 取名为Http.swift

Http.swift

```

import Alamofire

enum MethodType {
    case get
    case post
}

class Http {
    class func requestData(_ type : MethodType, URLString : String, parameters : [String : Any]? = nil, finishedCallback :  @escaping (_ result : Any) -> ()) {
        let method = type == .get ? HTTPMethod.get : HTTPMethod.post
        Alamofire.request(URLString, method: method, parameters: parameters).responseJSON { (response) in
            
            guard let result = response.result.value else {
                print(response.result.error)
                return
            }
            
            finishedCallback(result)
        }
    }
}

```

我们在Controller使用网络请求, URL为之前的API http://localhost:3001/api/J1/getJ1List

```
class Controller: UIViewController {

    fileprivate lazy var models: [Model] = [Model]()
    fileprivate lazy var baseView: View = { [weak self] in
        return View(frame: self!.view.bounds)
    }()
    
    override func loadView() {
        super.loadView()
        title = "J1"
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupView()
        requestData()
    }
}

extension Controller {
    
    fileprivate func setupView() {
        view.addSubview(baseView)
    }
    
    fileprivate func requestData() {
        Http.requestData(.get, URLString: "http://localhost:3001/api/J1/getJ1List") { (response) in
            guard let result = response as? [String : Any] else { return }
            guard let data:[String : Any] = result["data"] as? [String : Any] else { return }
            guard let models:[[String : Any]] = data["models"] as? [[String : Any]] else { return }
            
            self.models.removeAll()
            for dict in models {
                self.models.append(Model(dict: dict))
            }
            self.adapterView()
        }
    }
    
    fileprivate func adapterView() {
        baseView.models = models
    }
}


```

运行发现什么都没有, 因为本地的是Http协议, 我们需要强制越权.

```
info.plist
App Transport Security Setting
Allow Arbitrary Loads = YES
```

再次运行就能够读取到服务器返回的数据了, 但是终端的打印乱码强迫症不能忍啊!

```
Product -> Scheme -> Edit Scheme
Environment Variable 中输入:
OS_ACTIVITY_MODE , disable

```
这样乱码问题就解决了!!

这就是我理解的MVC模式, 不知道和大家理解的有没有什么区别.. 

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
