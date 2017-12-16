---
layout: post
title: Hybird 搭建路由Router实现组件化
date: 2017.04.12 13:40
tag: 移动开发
---

> MVP是个非常好的设计模式, 能够进行代码解耦, 业务分层, 组件化必备良品, 但是还能不能再提升一步呢? 现在项目中的耦合是很少了, 但是控制器之前的切换还是有耦合存在, 有什么办法能够实现控制器零耦合呢?? 那就要祭出本系列的核心Router模式了.代码见: [github](https://github.com/coderZsq/coderZsq.target.swift)

![](http://upload-images.jianshu.io/upload_images/1229762-3866097366648136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 参考链接:

- [Hybird 搭建零耦合架构从MVC开始](http://www.jianshu.com/p/5a03995a6ce1)
- [Hybird 搭建后端Koa.js并过度到MVVM](http://www.jianshu.com/p/846b9f181cb7)
- [Hybird 搭建前端Vue.js并升级至MVP](http://www.jianshu.com/p/8d4a84e3ddaa)

以下内容在上述文章基础上进行, 请事先查阅.

零耦合一直都是神话般的存在, 但是身边的负能量总是不绝于耳, 不要为了用设计模式而设计模式, 诶.. 我们还是回到Router上来吧, 究竟如何才能构建Router架构呢? 不急, 我们先把前端的项目给搭建完毕吧.

首先我们将端口号改为3000, 对应端口 前端: 3000, 接口服务器: 3001, 图片服务器: 3002

```
├── config
  ├── dev.env.js
  ├── index.js
  └── prod.env.js
```
  
index.js 

```
  dev: {
    env: require('./dev.env'),
    port: 3000, //update
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    cssSourceMap: false
  }
}


```

我们先在package.json中添加网络请求axios, Vue2开始的官方推荐, 

package.json

```
  "dependencies": {
    "axios": "^0.15.3",
    "element-ui": "^1.1.2",
    "stylus": "^0.54.5",
    "stylus-loader": "^2.4.0",
    "vue": "^2.1.0",
    "vue-router": "^2.1.1"
  },
```

并在src下创建javascripts 和 stylesheets 两个目录:

```
.
├── App.vue
├── assets
│   └── logo.png
├── components
├── javascripts
├── main.js
├── router
│   └── index.js
└── stylesheets
```

并在javascripts中添加http.js来执行网路请求脚本:

```
├── javascripts
    ├── http.js

```

http.js

```

import axios from 'axios'

export function GET(url) {

  return new Promise((resolve, reject) => {
    axios.get(url).then((response) => {
      resolve(response.data.data);
    }).catch((error) => {
      reject(error);
    })
  });
}

export const URL = {
  getJ1List: 'http://localhost:3001/api/J1/getJ1List'
}


```

代码也很简单, 和之前Swift的Alamofire一样, 在外面封装一层, 使用ES6语法的Promise, 并定义接口URL.


接着我们将Hello.vue修改为J1.vue

```
├── src
  ├── components
    └── J1.vue
```   

J1.vue

```

<template lang="html">
  <div>
    <div class="cell" v-for="model in models">
        ![](model.imageUrl)
        <span>{{model.text}}</span>
        <span>{{model.detailText}}</span>
    </div>
  </div>
</template>

<script>

import { GET,URL } from '../javascripts/http'

export default {
  data () {
    return {
      title: '',
      models: []
    }
  },
  methods: {
    request: function() {
      GET(URL.getJ1List).then((data) => {
          this.models = data.models;
      }).catch((error) => {

      })
    }
  },
  mounted:function () {
    this.title = document.title = "J1";
    this.request();
    // alert(window.location.href);
  }
}
</script>

<style scoped>

.cell {
    height: 100px;
    position: relative;
    border-bottom: 1px solid lightgray;
}

.cell img {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    padding-left: 10px;
}

.cell span:first-of-type {
    position: absolute;
    top: 35%;
    transform: translateY(-35%);
    padding-left: 84px;
    font-size: 15px;
}

.cell span:last-of-type {
    position: absolute;
    top: 65%;
    transform: translateY(-65%);
    padding-left: 84px;
    font-size: 12px;
}

</style>

```
接下来我们修改路由:

```
├── src
  ├── router
    └── index.js
```

```
import Vue from 'vue'
import Router from 'vue-router'
import J1 from '../components/J1'

Vue.use(Router)

export default new Router({
  routes: [{
      path: '/',
      component: J1
    },
    {
      path: '/J1',
      component: J1
    },
    {
      path: '*',
      redirect: '/'
    }
  ]
})

```

执行一下, 怎么什么都没有, 这个就要说到ajax的跨域问题了, 我们这里不展开, 对跨域有疑问的同学请自行Google.

要解决跨域, 我们通过在后端加载跨域权限的方式:

在server/app.js 中添加:

```
const cors = require('koa-cors');

app.use(cors());

```
当然你需要 npm install一下了, 这样我们就能看见web端的显示了.

前端不封叙述的有点多了, 但这对于热修复还是必须的, 我们回到移动端的项目上来, 在加载webView的时候将端口号改为3000, 这样我们在app端就能够加载刚才创建相同版本的web页面了.

连载到现在, 我们发现了Koa中的接口是基于Router的, Vue中的页面跳转也是基于Router的, 那我们移动端能不能也有自己的Router跳转机制呢? 其实一些紧跟潮流的开发者都知道现在流行组件化, 而移动端组件化的代表就是Router, 也有一些第三方框架之类的, 孰优孰劣我没用过, 也不清楚, 在这里我只是基于对前端路由的理解来搭建一个移动端的路由, 首先我们添加一个Router.swift.

Router.swift

```
class Router {
    static let shareRouter = Router()
    var params: [String : Any]?
    fileprivate let map = ["J1" : "Controller"]
}


extension Router {
    
    func addParam(key: String, value: Any) {
        params?[key] = value
    }
    
    func clearParams() {
        params?.removeAll()
    }
    
    func push(_ path: String) {
        guard let nativeController = NSClassFromString("RouterPatterm.\(self.map[path]!)") as? UIViewController.Type else { return }
        currentController?.navigationController?.pushViewController(nativeController.init(), animated: true)
    }
}

```

首先Router必须是一个单例, Swift的单例比起Objective-C真是简单太多了, 来看我们的Router类, map是对应匹配跳转控制器,并可以添加参数, 删除参数, 和跳转的方法, 当然也可以加上pop等其他功能, 我这里就不演示, 有兴趣的同学自行研究. 

定义完成之后, 我们就可以进行使用了, 我们修改业务逻辑的Presenter.swift

Presenter.swift

```

extension Presenter: ViewOperation {
    
    func pushTo() {
        Router.shareRouter.params = [
            "text" : "app端 传入数据",
            "code" : 1001
        ]
        Router.shareRouter.push("J1") //update
    }
}

```

在操作这里直接更改为使用Router跳转, 这样更具map定义的匹配, 我们就能够进行零耦合的页面跳转了. 如果需要传参也可以通过字典的形式存储在路由中, 当使用完成后对参数做清空操作就可以了. 

现在我们对移动端的路由跳转已经到位了, 接下啦该说说重点的热更新的思想了, 我们通过一个从服务器读取的字段来匹配该页面是跳转到移动端还是前端, 简单来说就是当移动端代码逻辑发生错误时怎么将移动端的页面降级为前端的页面. 这里我们在后端定义下获取跳转的接口.

```
controllers
	└── J1.js
```

J1.js

```
exports.getRouters = async(ctx, next) => {

    ctx.body = {
        routers: {
            J1: 'web'
        }
    }
}

```
并挂载到路由上:

```
.
├── api
│   ├── J1_router.js
│   └── index.js
├── index.js
└── users.js
```

J1_router

```

var router = require('koa-router')();
var J1 = require('../../app/controllers/J1');

router.get('/getRouters', J1.getRouters); //update
router.get('/getJ1List', J1.getJ1List);

module.exports = router;

```

通过接口访问, 我们就能得到返回数据:

```
{
  "code": 0,
  "message": "success",
  "data": {
    "routers": {
      "J1": "web"
    }
  }
}

```

回到移动端, 我们就可以根据这个接口来判断路由的跳转了.

Router.swift

```
class Router {
    static let shareRouter = Router()
    var params: [String : Any]?
    var routers: [String : Any]? //update
    fileprivate let map = ["J1" : "Controller"]
    
    func guardRouters(finishedCallback : @escaping () -> ()) { //update
        
        Http.requestData(.get, URLString: "http://localhost:3001/api/J1/getRouters") { (response) in
            guard let result = response as? [String : Any] else { return }
            guard let data:[String : Any] = result["data"] as? [String : Any] else { return }
            guard let routers:[String : Any] = data["routers"] as? [String : Any] else { return }
            self.routers = routers
            finishedCallback()
        }
    }
}

extension Router {
    
    func addParam(key: String, value: Any) {
        params?[key] = value
    }
    
    func clearParams() {
        params?.removeAll()
    }
    
    func push(_ path: String) {
        
        guardRouters { //update
            guard let state = self.routers?[path] as? String else { return }
            
            if state == "app" {
                guard let nativeController = NSClassFromString("RouterPatterm.\(self.map[path]!)") as? UIViewController.Type else { return }
                currentController?.navigationController?.pushViewController(nativeController.init(), animated: true)
            }
            
            if state == "web" {
                let host = "http://localhost:3000/"
                let webViewController = WebViewController("\(host)")
                currentController?.navigationController?.pushViewController(webViewController, animated: true)
            }
        }
    }
}
```
现在我们在每次路由跳转的时候都请求到对应的页面状态就可以后台操作跳转Native还是H5了.

回顾之前的代码, 我们现在已经做到了Model, View, ViewModel, Presenter和Controller之间的零耦合, 想想是不是有点小激动, 下一节内容, 我们会将Swift, Vue, Koa三端进行统筹, 实现真正的热修复框架.

About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.target.swift)
