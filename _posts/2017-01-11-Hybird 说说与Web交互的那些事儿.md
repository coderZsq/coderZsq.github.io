---
layout: post
title: Hybird 说说与Web交互的那些事儿
date: 2017.01.11 23:33
tag: 移动开发
---

> 喵神100Tips的解读连载已经到了第十篇了, 是要好好敲一个Demo来巩固一下自己Swift的学习情况, 说来也巧, 前两天领导来问我能不能帮他做一个需求, 要求web页面调用app原生相册, 选择照片后将图片加载到web页面上. (同学们先想想怎样才能够实现这个需求), 这正是一个锻炼Swift3的机会, 绝对不能错过, 虽然一口答应, 但实际操作中还是有一些技术难点存在的, 在此与你分享. 


![](http://upload-images.jianshu.io/upload_images/1229762-3866097366648136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


要做这个功能, 虽然对我来说还是Objective-C比较熟悉, 各项技术黑魔法都手到擒来, 但是Swift毕竟是未来的趋势, 到Swift3后API也趋于稳定, 因此我就选择了Swift作为app端开发语言. web端当仁不让的就选择Javascript了, 领导一开始要求我使用HTML5的Canvas来实现, 可杀鸡焉用牛刀, 我就简单的使用img标签代替了, 其实Canvas与Quartz2D的API极为相似, 有经验的同学很好上手!

我们先来分析一下这个需求, 更好的拆解需求能够设计出理想的代码, 首先, 我们想到了需要一个WebView, 然后实现代理进行交互, 但是一般来说我们与H5交互都是传字符串, 如何传输图片数据呢? (在此声明, 此需求不涉及请求及服务端) 

带着这个问题, 我们先把工程创建起来, 毕竟良好的开端是成功的一半!
```
fileprivate lazy var webView: UIWebView = {[weak self] in
     let webView = UIWebView()
     webView.delegate = self       
     webView.frame = CGRect(x: 0,
                            y: 0,
                            width:  kScreenW,
                            height: kScreenH)
     return webView
}()
```
我们先通过懒加载创建webView, 与Objective-C重写get方法不同的是, Swift中自带lazy关键字, 可以轻松实现懒加载, 上面是通过闭包形式的懒加载方式, 所以在闭包中引用self的话会造成循环引用, 所以我们需要加上[weak self] in, fileprivate关键字是Swift3更新的, 意思是本文件中似私有.

```
extension ViewController {
    
    fileprivate func loadWebView() {
        guard let file = Bundle.main.path(forResource: "index", ofType: "html") else { return }
        guard let htmlString = try? String(contentsOfFile: file, encoding: .utf8) else { return }
        let baseURL = URL(fileURLWithPath: Bundle.main.bundlePath)
        webView.loadHTMLString(htmlString, baseURL: baseURL)
    }
}
```
创建完webView我们就需要加载本地的HTML文件了, 加载方式还是和Objective-C的一样, 但需要注意的是我把这个scope放在了一个extension中, extension是Swift中本人最为喜欢的功能了, 对比Objective-C的Catagroy来说, 它的装饰模式的灵活性真是强大太多了.(不理解装饰模式的同学可以看看设计模式方面的书籍) guard也是Swift中我比较喜欢的功能, 可选绑定校验能够使代码逻辑更为清晰, 对我这种对代码美感有强迫症的人来说, 真是太合适不过了. 第三个注意点事try?, 在Objective-C的时代我们对异常的处理其实非常的少, 但在Swift中却极为常见, 可能是对安全方面的考量吧, 有兴趣的同学可以看看我之前分享的喵神的书中就有这一段分析.

```

override func viewDidLoad() {
      super.viewDidLoad()
      setupWebView()
      loadWebView()    
}

extension ViewController {
    
    fileprivate func setupWebView() {
        view.addSubview(webView)
    }
}
```
我们把webView添加到控制器的view上并进行读取HTML文件.

```

<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <title>PhotoGallery</title>

<style>
    
p {
    margin: 0;
    padding: 0;
    text-align: center;
    font-size: 18px;
}

</style>
</head>

<body>
    
![](placeholder)
<p> # click here to select a photo # </p>

<script type="text/javascript">

</script>
</body>

</html>
```
接下来我们先创建一个非常简单的HTML文件, 里面只包含了一张图片和一段文字, 这里我图个方便将CSS和JS都集成在HTML文件中了, 一般做项目建议分开引用. script标签按照规范放置在body标签的最后, 这是web项目最基础的优化.

```
function prepareParagraphEvent() {
    if (!document.getElementsByTagName) return false;
    var ps = document.getElementsByTagName("p");
    for (var i = 0; i < ps.length; i++) {
        ps[i].onclick = function() {
            window.location.href = "ios://openPhotoGallery";
        }
    }
}
```
我们通过操作DOM的方式来给p标签绑定一个点击事件, 当然你也可以用jQuery这种JS库, 我这里就用原生的API进行实现了, 第一个判断的的意义是要让网页做到平稳退化, 当然现在大部分浏览器都支持的很好, 但我们也要照顾到那些不支持JS的浏览器不是吗? 这里有一个技术点: **ios://openPhotoGallery** 这个字符串其实就是跳转的关键, 如果在工作中就需要app端和web端制定规则, 因为app端需要根据这段字符串来进行解析.

```

function addLoadEvent(func) {
    var oldonload = window.onload;
    if (typeof window.onload != 'function') {
        window.onload = func;
    } else {
        window.onload = function() {
            oldonload();
            func();
        }
    }
}

addLoadEvent(prepareParagraphEvent);
```
我们默认加载此prepareParagraphEvent函数, window.onload可以简单的理解为viewDidLoad.

```
extension ViewController: UIWebViewDelegate {
    
    func webView(_ webView: UIWebView, shouldStartLoadWith request: URLRequest, navigationType: UIWebViewNavigationType) -> Bool {
        guard let href = request.url?.absoluteString else { return false }
        if href.hasPrefix("ios") {
            guard let method = href.components(separatedBy: "://").last else { return false }
            let selector = Selector.init(method)
            if self.responds(to: selector) {
                self.perform(selector)
            }
            return false
        }
        return true
    }
}
```
当window.location.href触发的时候, 会调用UIWebViewDelegate的上述代理方法. 我们根据刚才所制定的规则进行解析, 我这里是使用类似协议头的方式, 这样看起来更加的正式, 当然你也可以制定你想制定的任何一种方式. 我们根据协议头后的方法名字符串通过桥接Objective-C的runtime来动态执行方法.

```

extension ViewController {
    
    @objc fileprivate func openPhotoGallery() {
        if UIImagePickerController.isSourceTypeAvailable(.photoLibrary){
            let picker = UIImagePickerController()
            picker.delegate = self
            picker.sourceType = .photoLibrary
            picker.allowsEditing = true
            self.present(picker, animated: true)
        } else {
            print("access photo gallery error")
        }
    }
}
```
当然我们也要准备相应的方法, @objc 关键字 就是进行与Objective-C的桥接, 方法内部的功能是,打开系统图片库, 这代码我就不分析了, 因为实在是没有什么可说的.. 到这一步, 我们就已经实现了一半了, 通过点击web中的标签打开了app的系统的照片库.当然就像伟大航线的新世界一样, 好戏在后头.

```
extension ViewController: UIImagePickerControllerDelegate,UINavigationControllerDelegate {
    
    func imagePickerController(_ picker: UIImagePickerController,didFinishPickingMediaWithInfo info: [String : Any]) {
        let image = info[UIImagePickerControllerOriginalImage] as? UIImage
        guard let Document = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true).last
            else { return }
        let path = "\(Document)/\("image\(arc4random()%100).png")"
        if FileManager.default.fileExists(atPath: path) {
            try! FileManager.default.removeItem(atPath: path)
        }
        try! UIImagePNGRepresentation(image!)?.write(to: URL(fileURLWithPath: path))
        picker.dismiss(animated: true) {
            self.webView.stringByEvaluatingJavaScript(from: "loadImageWithPath('\(path)');")
        }
    }
}
```
这段代码可谓是这个Demo中最核心的一段了, 这个是UIImagePickerController的代理方法回调, 简单来说就是拿到选择的照片, 我实现的思路是将图片写入沙盒并将沙盒路径传递给Javascript. (在做这个的过程中了解到了一个之前忽视掉的细节, 就是在iOS8开始, 每次打开沙盒的路径都是不同的) 传递的核心代码就是 **stringByEvaluatingJavaScript(from: "loadImageWithPath('\(path)');**, 就是调用JS的函数. 

```

function loadImageWithPath(src) {
    if (!document.getElementsByTagName) return false;
    var imgs = document.getElementsByTagName("img");
    for (var i = 0; i < imgs.length; i++) {
        imgs[i].src = src;
    }
}
```
通过调用函数, 更改图片路径, 我们的功能也终于大功告成啦!!! 来看一下实现的效果.


![](http://upload-images.jianshu.io/upload_images/1229762-9a4eea0d27b2a9ae.gif?imageMogr2/auto-orient/strip)

其实还有一个bug并没有很好的解决, **let path = "\(Document)/\("image\(arc4random()%100).png")"**, 我使用了随机数生成的图片名来抑制bug的产生, 但这并不是合适的做法, 如果不生成随机数的话, 切换第一张图片后, 第二张图片将不会切换, 但沙盒中的图片数据已经发生更改, 为了解决这个bug, 还请大神们伸出援助之手吧!

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
