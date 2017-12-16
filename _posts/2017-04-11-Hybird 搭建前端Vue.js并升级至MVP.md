---
layout: post
title: Hybird 搭建前端Vue.js并升级至MVP
date: 2017.04.11 15:06
tag: 移动开发
---

> 上回我们说到MVVM, 说真的我真没觉得MVVM为什么会鼓吹的那么神, 用大篇幅的叙述我着实提不起劲. 回顾一下内容, 我们通过Node.js搭建了图片服务器, 并通过Kingfisher加载出来, 说真的前两篇主要还是叙述移动端入门后端吧.. 噗.. 笑.. 这一节我们来说说MVP模式, 也是我比较喜欢以及推荐大家使用的一种设计思想. 代码见:[github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-3866097366648136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 参考链接:

- [Hybird 搭建零耦合架构从MVC开始](http://www.jianshu.com/p/5a03995a6ce1)
- [Hybird 搭建后端Koa.js并过度到MVVM](http://www.jianshu.com/p/846b9f181cb7)

以下内容在上述文章基础上进行, 请事先查阅.

看了网上好多类型的架构设计, 用那些玄乎的理论, 诸如高内聚低耦合, DRY等专有名词之类的泛泛而谈, 一条条提纲挈领的诉说着编写代码的该有的规范, 我不否认这些思想很重要, 但不能运用到实际上也只能是夸夸其谈罢了, 也许是我的道行略低不能企及那样的高度, 但光有思维方式不给实操方式, 与宗教又有何异? 总结就是, 对于架构设计, 不上代码就是扯!

MVP 

MVP就是 Most Valuable Player, 最有价值球员.. 笑" 话说我大威少场均三双赛季封神啊, 言归正传, MVP在架构设计来说是指 Model, View, Presenter,  其中的Presenter指的是展示层, 操作页面逻辑的一层. MVP也是我最为推荐的一种设计模式, 做Android的同学应该有听说过适配器模式, 也就是Adapter, MVP中的Presenter就好像是双向的Adapter, 对代码进行完全的解耦实现组件化.

我们来看看如何切换到MVP模式:

```
.
├── Controller.swift
├── Model.swift
├── Presenter.swift
├── View.swift
└── ViewModel.swift
```

我们在目录结构中添加Presenter.swift

Presenter.swift

```

protocol ModelInterface {
    var text : String {get set}
    var detailText : String {get set}
    var imageUrl : String {get set}
}

protocol ViewInterface {
    var viewModel: ViewModelInterface? {get set}
}

protocol ViewModelInterface {
    var models: [ModelInterface] {get set}
    mutating func dynamicBinding(finishedCallback : @escaping () -> ())
}

class Presenter {
    var view: ViewInterface?
    var viewModel: ViewModelInterface?
}

extension Presenter {
    
    func adapter<VM: ViewModelInterface,V: ViewInterface>(viewModel: VM,  view: V) {
        
        self.view = view;
        self.viewModel = viewModel
        
        func dynamicBinding() {
            self.viewModel?.dynamicBinding {
                self.view?.viewModel = viewModel
            }
        }; dynamicBinding()
    }
}

```

MVP模式还是有必要详细的说明一下的, 我们对于Model, View, ViewModel, 也就是之前的MVVM对应的定义协议, 协议在Java中也叫接口, 就是制定一套大家都该遵守的规范. 将MVVM全部抽象化就是Presenter层的主要职责.

我们对应修改代码, 使其遵守Presenter定义的协议.

Model.swift

```
class Model: NSObject, ModelInterface { //update
    
    var text : String = ""
    var detailText : String = ""
    var imageUrl : String = ""
    
    init(dict : [String : Any]) {
        super.init()
        setValuesForKeys(dict)
    }
    
    override func setValue(_ value: Any?, forUndefinedKey key: String) {}
}

```

View.swift 

```
class View: UIView, ViewInterface { //update
    
    var value: ViewModelInterface? //update
    var viewModel: ViewModelInterface? { //update
        
        get { return value }
        set {
            value = newValue
            tableView.reloadData()
        }
    }
    
    fileprivate lazy var tableView: UITableView = { [weak self] in
        var tableView = UITableView(frame: self!.bounds, style: .plain)
        tableView.dataSource = self
        tableView.delegate = self
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
        let model: ModelInterface? = viewModel?.models[indexPath.row] //update
        let cell = tableView.dequeueReusableCell(withIdentifier: identifier) ?? UITableViewCell(style: .subtitle, reuseIdentifier: identifier)
        cell.textLabel?.text = model?.text
        cell.detailTextLabel?.text = model?.detailText
        cell.imageView?.loadUrl(imageUrl: model?.imageUrl)
        return cell
    }
}

extension View: UITableViewDelegate {
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 100
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
    }
}


```

ViewModel.swift

```
class ViewModel: ViewModelInterface { //update
    lazy var models: [ModelInterface] = [ModelInterface]() //update
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

Controller.swift 

```

class Controller: UIViewController {

    fileprivate lazy var viewModel: ViewModel = ViewModel()
    fileprivate lazy var presenter: Presenter = Presenter() //update
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
        presenter.adapter(viewModel: viewModel, view: baseView) //update
    }
}

```
最后我们在Controller中引入Presenter, 并对View和ViewModel进行适配.

运行一下, 和之前一样没有什么异常, 但代码之间已经完全的零耦合了, 只要遵守Presenter内所定义的协议, 替换掉任意一个部件, 也不会有什么问题. 这种代码设计就能够轻松的实现组件化了.

但Presenter仅仅是负责这点工作吗? 其实我们可以将View中的逻辑也通过协议的形式接入进来, 具体如何实现, 不急, 我们先构建一个前端的项目吧!

对于前端, 我使用的框架是尤雨溪大神的[Vue](https://vuejs.org/), 对于为什么使用Vue而不使用Google的Angluar, 或是Facebook的React, 哎.. 就是任性, 随便...

和之前的Koa一样, 我们也找一个生成器吧, 好在Vue三件套中就有生成器cli, 好吧专业名词脚手架 - - , 我们就通过它来快速构建前端工程吧!

1. $ npm install webpack -g
2. $ npm install vue-cli -g
3. cd 目录 $ vue init webpack RouterPattern

```
? Project name routerpattern
? Project description A Vue.js project
? Author zhushuangquan <zhushuangquan@j1.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? Yes
? Setup e2e tests with Nightwatch? Yes

   vue-cli · Generated "RouterPattern".

   To get started:
   
     cd RouterPattern
     npm install
     npm run dev
   
   Documentation can be found at https://vuejs-templates.github.io/webpack

// webpack的项目不能用大写字母, 所以我们都改为小写.

```

```
.
├── README.md
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── dev-client.js
│   ├── dev-server.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   └── prod.env.js
├── index.html
├── package.json
├── src
│   ├── App.vue
│   ├── assets
│   │   └── logo.png
│   ├── components
│   │   └── Hello.vue
│   ├── main.js
│   └── router
│       └── index.js
└── static

```

终端执行 $ npm install  $ npm run dev 后在localhost:8080 上就能看到欢迎页面了.

前端的部分我们就先到这里, 我们回到移动端来, 有了Web页面, 对应的我们就需要创建WebView了.

```
import WebKit

class WebViewController: UIViewController {
    
    fileprivate lazy var configuretion: WKWebViewConfiguration = { [weak self] in
        let configuretion = WKWebViewConfiguration()
        return configuretion
    }()
    
    fileprivate lazy var webView: WKWebView = { [weak self] in
        let webView = WKWebView(frame: self!.view.bounds)
        webView.navigationDelegate = self
        webView.uiDelegate = self
        return webView
    }()
    
    fileprivate lazy var progressView: UIProgressView = {
        let progressView = UIProgressView(frame: CGRect(x: 0, y: 65, width: self.view.bounds.size.width, height: 2))
        progressView.trackTintColor = .clear
        return progressView
    }()
    
    fileprivate var absoluteUrl = ""
    
    init (_ url: String) {
        super.init(nibName: nil, bundle: nil)
        absoluteUrl = url.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? ""
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(webView)
        view.addSubview(progressView)
        
        func loadRequest() {
            var request = URLRequest(url: URL(string: absoluteUrl)!)
            request.addValue("skey=skeyValue", forHTTPHeaderField: "Cookie")
            webView.addObserver(self, forKeyPath: "estimatedProgress", options: [.new, .old], context: nil)
            webView.load(request)
        }; loadRequest()
    }
    
    override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
        
        if keyPath == "estimatedProgress" {
            progressView.alpha = 1.0
            progressView.setProgress(Float(webView.estimatedProgress), animated: true)
            if webView.estimatedProgress >= 1.0 {
                UIView.animate(withDuration: 0.25, delay: 0.25, options: [.curveEaseOut], animations: {
                    self.progressView.alpha = 0.0
                    }, completion: { (finished) in
                        self.progressView.setProgress(0.0, animated: true)
                })
            }
        }
    }
    
    deinit {
        webView.removeObserver(self, forKeyPath: "estimatedProgress")
    }
}

extension WebViewController: WKNavigationDelegate {
    
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        title = webView.title
    }
}

extension WebViewController: WKUIDelegate {
    
    func webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String, initiatedByFrame frame: WKFrameInfo, completionHandler: @escaping () -> Void) {
        
        let alertViewController = UIAlertController(title: nil, message: message, preferredStyle: .alert)
        alertViewController.addAction(UIAlertAction(title: "确定", style: .default, handler: { (action) in
            completionHandler()
        }))
        
        present(alertViewController, animated: false, completion: nil)
    }
}

```

之前的UIWebView的简单交互已经有讲解, 这次就使用最新的WKWebView吧. 代码没什么说的, 只不过WK的内核相同, 与JS交互更加方便了, 这个我们下次再谈.

我们点击cell让它push到刚才vue的展示页, 在View中拿到当前控制器不方便, 我们创建一个全局变量来保存当前控制器.

ViewController.swift

```
var currentController: ViewController?

class ViewController: UIViewController {
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        currentController = self
    }
}

```

再让Controller和WebViewController继承与ViewController, 这样在哪里都能拿到当前控制器了.

View.swift

```

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        currentController?.navigationController?.pushViewController(WebViewController("http://localhost:8080/"), animated: true)
    }

```

这样我们就能够跳转到刚才Vue生成的页面了, 但是这种跳转逻辑写在View中, 不是很好维护, 这时Presenter, 就能够实行它的第二个用途了 -- 统筹业务逻辑. 

我们在View中定义操作协议: ViewOperation

View.swift 

```

protocol ViewOperation { //update
    mutating func pushTo() -> Void
}

class View: UIView, ViewInterface {
    
    var operation: ViewOperation? //update
    var value: ViewModelInterface?
    var viewModel: ViewModelInterface? {
        
        get { return value }
        set {
            value = newValue
            tableView.reloadData()
        }
    }
    
    fileprivate lazy var tableView: UITableView = { [weak self] in
        var tableView = UITableView(frame: self!.bounds, style: .plain)
        tableView.dataSource = self
        tableView.delegate = self
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
        let model: ModelInterface? = viewModel?.models[indexPath.row]
        let cell = tableView.dequeueReusableCell(withIdentifier: identifier) ?? UITableViewCell(style: .subtitle, reuseIdentifier: identifier)
        cell.textLabel?.text = model?.text
        cell.detailTextLabel?.text = model?.detailText
        cell.imageView?.loadUrl(imageUrl: model?.imageUrl)
        return cell
    }
}

extension View: UITableViewDelegate {
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 100
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        operation?.pushTo() //update
    }
}


```

在Presenter中进行实现

Presenter.swift

```

extension Presenter: ViewOperation {
    
    func pushTo() {
        currentController?.navigationController?.pushViewController(WebViewController("http://localhost:8080/"), animated: true)
    }
}

```

这样我们把业务逻辑都抽象出来了, Presenter中既能拿到数据又能拿到视图还能进行业务操作, 简直太棒了, 这就是我向你推荐的MVP模式!

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
