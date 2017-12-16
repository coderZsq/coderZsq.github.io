---
layout: post
title: Hybird 搭建后端Koa.js并过度到MVVM
date: 2017.04.11 12:42
tag: 移动开发
---

> 回顾上期的内容, 我们通过Koa搭建服务器实现了移动端与后端的交互, 使得app不再只是个壳子,  通过CocoaPods集成了Alamofire成功调用后端的API, 实现了最精简也是最经典的MVC架构, 今天我们来说说之前大热的MVVM架构.代码详见:[github](coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-3866097366648136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 参考链接:

- [Hybird 搭建零耦合架构从MVC开始](http://www.jianshu.com/p/5a03995a6ce1)

以下内容在上述文章基础上进行, 请事先查阅.

其实现在iOS开发还是在Objective-C的大环境下, 让我在项目中使用MVVM这种架构, 其实我是拒绝的, 因为个人觉得OC的代码设计是不适合使用MVVM框架的, 具体原因就是 "丑", 一个字代表了所有的观点. 但作为技术研讨, MVVM的思想还是有向大家普及的必要的. 

MVVM

MVVM是指 Model, View, ViewModel, 这里的ViewModel代替了原先Controller的部分工作, MVVM的概念最初是从前端开始的, 以后面会说到的Vue举例, 每一个Vue文件就是一个ViewModel, 实现了动态绑定的功能, 通俗的说就是当数据发生改变, 视图立即改变, 相反当用户操作视图也同时操作了数据. ViewModel从名字上来看就是View和Model的中间层.

接下来我们来看一下移动端怎么实现MVVM架构:

```
.
├── Controller.swift
├── Model.swift
├── View.swift
└── ViewModel.swift

```

我们先在项目结构下添加ViewModel.swift, 将之前在Controller中的网络请求代码移植到VM中:

ViewModel.swift

```
class ViewModel {
    lazy var models: [Model] = [Model]()
}

extension ViewModel {
    
    func dynamicBinding(finishedCallback : @escaping () -> ()) {
        
        Http.requestData(.get, URLString: "http://localhost:3001/api/J1/getJ1List") { (response) in
            guard let result = response as? [String : Any] else { return }
            guard let data:[String : Any] = result["data"] as? [String : Any] else { return }
            guard let models:[[String : Any]] = data["models"] as? [[String : Any]] else { return }
            
            self.models.removeAll()
            for dict in models {
                self.models.append(Model(dict: dict))
            }
            
            finishedCallback()
        }
    }
}

```

小贴士:
Swift中的Class可以不用继承直接定义, 降低开销.

我们对应修改其他代码如下:

View.swift 

```

class View: UIView {
    
    var viewModel: ViewModel? { //update
        
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
        return viewModel?.models.count ?? 0 //update
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        let identifier = "identifier"
        let model: Model? = viewModel?.models[indexPath.row] //update
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

    fileprivate lazy var viewModel: ViewModel = ViewModel() //update
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
    

    fileprivate func adapterView() { //update
        viewModel.dynamicBinding {
            self.baseView.viewModel = self.viewModel
        }
    }
}


```

我们通过将网络请求封装到ViewModel中实现了代码分层, 设计也更为简洁. 但是我们现在的用户界面好Low啊, 是不是应该做点什么? 对了, 我们给界面里添加点图片吧!

当然图片我们也是通过后端获取, 我们在后端目录中添加image.js文件来实现图片服务器.

```
.
└── public
  └── images
	└── image.js

```

image.js 

```

const http = require('http');
const fs = require('fs');

http.createServer((req, res) => {
    console.log(req.url);

    let path = '..' + decodeURI(req.url);
    fs.readFile(path, 'binary', (err, file) => {
        if (err) {
            // console.log(err);
            return;
        } else {
            res.writeHead(200, {
                'Content-Type': 'image/png'
            });
            res.write(file, 'binary');
            res.end();
            return;
        }
    })
}).listen(3002);
console.log('port = 3002');

```
这里使用的是原生的Node.js搭建的图片服务器, 监听3002端口.

这里我们来说说服务器是如何实现想客户端传送数据的, 我们现在分为API服务器: 3001和图片服务器: 3002. API服务器传输的类型是Content-Type : application/json这种格式的, 而图片服务器返回的是二进制的格式Content-Type': 'image/png. 原理就是将文本或数据写在Body上.

我们cd 到目录中 通过node命令 $ node image.js 来执行js脚本文件. 看到port = 3002打印在终端上说明, 服务启动成功.

有图片服务器, 没有图片怎么行? 我们在images中添加图片文件. 添加完后在浏览器中输入: http://localhost:3002/images/J1/关于健一@2x.png, 就能够访问到服务器的图片了.

接下来我们将图片的URL地址通过API的形式返回给移动端调用.

```
.
└── app
  └── controllers
	└── J1.js

```

J1.js

```
exports.getJ1List = async(ctx, next) => {

    ctx.body = {
        models: [{
            text: '我的账户',
            detailText: "欢迎进入=>我的账户",
            imageUrl: "http://localhost:3002/images/J1/我的账户@2x.png"
        }, {
            text: '我的优惠券',
            detailText: "欢迎进入=>我的优惠券",
            imageUrl: "http://localhost:3002/images/J1/我的优惠券@2x.png"
        }, {
            text: '收货地址',
            detailText: "欢迎进入=>收货地址",
            imageUrl: "http://localhost:3002/images/J1/收货地址@2x.png"
        }, {
            text: '在线客服',
            detailText: "欢迎进入=>在线客服",
            imageUrl: "http://localhost:3002/images/J1/在线客服@2x.png"
        }, {
            text: '用药提醒',
            detailText: "欢迎进入=>用药提醒",
            imageUrl: "http://localhost:3002/images/J1/用药提醒@2x.png"
        }, {
            text: '药查查',
            detailText: "欢迎进入=>药查查",
            imageUrl: "http://localhost:3002/images/J1/药查查@2x.png"
        }, {
            text: '疾病百科',
            detailText: "欢迎进入=>疾病百科",
            imageUrl: "http://localhost:3002/images/J1/疾病百科@2x.png"
        }, {
            text: '药品百科',
            detailText: "欢迎进入=>药品百科",
            imageUrl: "http://localhost:3002/images/J1/药品百科@2x.png"
        }, {
            text: '健一咨询',
            detailText: "欢迎进入=>健一咨询",
            imageUrl: "http://localhost:3002/images/J1/健一咨询@2x.png"
        }, {
            text: '帮助中心',
            detailText: "欢迎进入=>帮助中心",
            imageUrl: "http://localhost:3002/images/J1/帮助中心@2x.png"
        }, {
            text: '点赞/吐槽',
            detailText: "欢迎进入=>点赞/吐槽",
            imageUrl: "http://localhost:3002/images/J1/点赞:吐槽@2x.png"
        }, {
            text: '关于健一',
            detailText: "欢迎进入=>关于健一",
            imageUrl: "http://localhost:3002/images/J1/关于健一@2x.png"
        }]
    }
}


```

在浏览器访问接口 http://localhost:3001/api/J1/getJ1List

```
{
  "code": 0,
  "message": "success",
  "data": {
    "models": [
      {
        "text": "我的账户",
        "detailText": "欢迎进入=>我的账户",
        "imageUrl": "http://localhost:3002/images/J1/我的账户@2x.png"
      },
      {
        "text": "我的优惠券",
        "detailText": "欢迎进入=>我的优惠券",
        "imageUrl": "http://localhost:3002/images/J1/我的优惠券@2x.png"
      },
      {
        "text": "收货地址",
        "detailText": "欢迎进入=>收货地址",
        "imageUrl": "http://localhost:3002/images/J1/收货地址@2x.png"
      },
      {
        "text": "在线客服",
        "detailText": "欢迎进入=>在线客服",
        "imageUrl": "http://localhost:3002/images/J1/在线客服@2x.png"
      },
      {
        "text": "用药提醒",
        "detailText": "欢迎进入=>用药提醒",
        "imageUrl": "http://localhost:3002/images/J1/用药提醒@2x.png"
      },
      {
        "text": "药查查",
        "detailText": "欢迎进入=>药查查",
        "imageUrl": "http://localhost:3002/images/J1/药查查@2x.png"
      },
      {
        "text": "疾病百科",
        "detailText": "欢迎进入=>疾病百科",
        "imageUrl": "http://localhost:3002/images/J1/疾病百科@2x.png"
      },
      {
        "text": "药品百科",
        "detailText": "欢迎进入=>药品百科",
        "imageUrl": "http://localhost:3002/images/J1/药品百科@2x.png"
      },
      {
        "text": "健一咨询",
        "detailText": "欢迎进入=>健一咨询",
        "imageUrl": "http://localhost:3002/images/J1/健一咨询@2x.png"
      },
      {
        "text": "帮助中心",
        "detailText": "欢迎进入=>帮助中心",
        "imageUrl": "http://localhost:3002/images/J1/帮助中心@2x.png"
      },
      {
        "text": "点赞/吐槽",
        "detailText": "欢迎进入=>点赞/吐槽",
        "imageUrl": "http://localhost:3002/images/J1/点赞:吐槽@2x.png"
      },
      {
        "text": "关于健一",
        "detailText": "欢迎进入=>关于健一",
        "imageUrl": "http://localhost:3002/images/J1/关于健一@2x.png"
      }
    ]
  }
}

```

数据返回没有问题了, 接下来我们来看看如何从移动端访问服务器图片, 这里使用喵神的Kingfisher来请求图片, 之前已经通过Pods导入到项目中了.

使用第三方框架的时候我们都需要在外面再封装一层, 我们创建Image.swift

Image.swift

```
import Kingfisher

extension UIImageView {
    
    func loadUrl(imageUrl: String?, placeholder: String = "placeholder") {
        self.kf.setImage(with: URL(string: imageUrl?.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? ""), placeholder: UIImage(named: placeholder), options: nil, progressBlock: nil, completionHandler: nil)
    }
}

```

接下来在Model和View中添加imageUrl字段并修改如下:

Model.swift

```
class Model: NSObject {

    var text : String = ""
    var detailText : String = ""
    var imageUrl : String = "" //update

    init(dict : [String : Any]) {
        super.init()
        setValuesForKeys(dict)
    }
    
    override func setValue(_ value: Any?, forUndefinedKey key: String) {}
}

```

View.swift

```
class View: UIView {
    
    var viewModel: ViewModel? {
        
        didSet {
            tableView.reloadData()
        }
    }
    
    fileprivate lazy var tableView: UITableView = { [weak self] in
        var tableView = UITableView(frame: self!.bounds, style: .plain)
        tableView.dataSource = self
        tableView.delegate = self //update
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
        return viewModel?.models.count ?? 0
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        let identifier = "identifier"
        let model: Model? = viewModel?.models[indexPath.row]
        let cell = tableView.dequeueReusableCell(withIdentifier: identifier) ?? UITableViewCell(style: .subtitle, reuseIdentifier: identifier)
        cell.textLabel?.text = model?.text
        cell.detailTextLabel?.text = model?.detailText
        cell.imageView?.loadUrl(imageUrl: model?.imageUrl) //update
        return cell
    }
}

extension View: UITableViewDelegate { //update
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 100
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
    }
}

```
以上就是个人理解的MVVM, 也说出你对于MVVM的想法, 我们一起探讨!

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
