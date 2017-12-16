---
layout: post
title: Web 使用Vue代替陈旧的jQuery
date: 2017.07.14 11:34
tag: 前端开发
---

> 这周的身体状态不好, 所以更新也比较有限了, 反正也不急这一时半会, 少则得多则惑不是吗, 我在现在在想一个问题, 简历制作完成进行投递的时候, 尼玛人事还要装个Node环境在在终端进行npm操作, 诶... 没有IP的痛啊~ 不多想了, 直接给个Png图片算了, 毕竟也不在乎这些...

![](http://upload-images.jianshu.io/upload_images/1229762-4daa1dfb99967aa8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 参考链接:

- [Web 是时候用前端写个简历了!](http://www.jianshu.com/p/d1497da0f9ab)
- [Web 前端项目要从基本布局开始](http://www.jianshu.com/p/5c4788c0389d)
- [Web 简历一定要设计的美美的](http://www.jianshu.com/p/b3389f66f539)

以下内容在上述文章基础上进行, 请事先查阅.

状态不好, 我就不多逼逼了, 直接进行我们的项目, 上周我们完成了home和project两个模块的开发, 今天我们来进行接下来的工作, 首先, 我们先来看一下今天的技术点:

![](http://upload-images.jianshu.io/upload_images/1229762-4a037769697f6645.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1229762-6750478a776e1537.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择Vue的理由是去年公司没有HTML5开发, 让我去主导前端开发, 技术选型的时候选择了Vue这个框架, 至今我也没有用过react和angluar, 但这都不重要, 我中专的时候可是科班出身, 像前端这种容易入门的语言还是很好学的. 说实话, 看官方文档是学习新技术的最佳方法了, 但也不排除有些官方文档写的很烂什么的.

简单的东西也不过多赘述了, 直接进行组件的封装:
###### 父组件 github.vue
```
<template>
<div>
  <div class="subtitle">
    <h1>GitHub</h1>
  </div>
  <div class="github">
    <div class="github-block">
      <target name="coderZsq.project.oc" description="Use the SQExtension & SQTemplate to quickly build a project architecture, quickly use wheel components, and quickly integrate animated effects." href="https://github.com/coderZsq/coderZsq.project.oc" color="#438eff"
        language="Objective-C" star="113"></target>
      <target name="coderZsq.target.swift" description="Learn the advanced path of Swift Including Hybrid, RxSwift, Animations, SpriteKit, SceneKit, CoreData, LLDB, iOS, watchOS, tvOS etc." href="https://github.com/coderZsq/coderZsq.target.swift" color="#ffac45"
        language="Swift" star="4"></target>
      <target name="coderZsq.display.js" description="Through Vue.js to build a personal resume, hope that most companies to discuss cooperation." href="https://github.com/coderZsq/coderZsq.display.js" color="#2C3E50" language="Vue" star="0"></target>
      <target name="coderZsq.github.io" description="Through Hexo to build a personal blog, hoping to leave a good memory moments with you." href="https://github.com/coderZsq/coderZsq.github.io" color="#563D7C" language="CSS" star="1"></target>
    </div>
  </div>
</div>
</template>
```
###### 子组件 target.vue
```
<template>
<div class="target">
  <div class="target-block">
    <h2>{{name}}</h2>
    <p>{{description}}</p>
    <div class="download">
      <a :href="href">Clone or download</a>
    </div>
    <div class="comment">
      <div class="dot" ref="dot"></div>
      <div class="language">
        {{language}}
      </div>
      <div class="star">
        ★ {{star}}
      </div>
    </div>
  </div>
</div>
</template>
```
这里的大多数概念在之前的文章中都有说明, 这次主要想讲的就是ref这个关键字, 看了上面官方文档的说明就能了解ref是javascript用来访问dom用的, 但是一般人都会想到用jquery吧, 昨天我问了做前端的同事, 现在还有人用jquery吗?  他说官网那些jsp的都用吧, 用着一脸嘲讽的语气, 
因为我个人是比较讨厌使用第三方的东西的, 所以既然vue有这个关键字用就好啦~~ 其实这里不适宜用jquery的真正原因是, Vue是一个单页的网页框架, 所以使用jquery无论是获取class还是id, 在进行循环的时候, 总会是重复的, 而ref和scope就避免了这个问题. 
###### 子组件 target.vue
```
<script>
export default {
  data() {
    return {

    }
  },
  props: {
    href: String,
    name: String,
    description: String,
    color: String,
    language: String,
    star: String
  },
  methods: {
    setDotColor: function() {
      this.$refs.dot.style.backgroundColor = this.color;
    }
  },
  mounted: function() {
    this.setDotColor();
  }
}
</script>
```
这里也有知识点, methods是用来写函数的, 而mounted是Vue组件的生命周期和viewDidLoad类似, 通过`this.$refs.dot.style.backgroundColor = this.color;`可以设置dom的css.

![实现效果](http://upload-images.jianshu.io/upload_images/1229762-1dbea85e6bd3b013.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完成了github模块, 我们就来进行articles模块的实现, 这里我们需要了解一个知识点:

![](http://upload-images.jianshu.io/upload_images/1229762-9eb4b8779970e140.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

了解了v-for的基本用法, 我们来看下组件的通信:
###### articles.vue
```
<template>
<div>
  <div class="subtitle">
    <h1>Articles</h1>
  </div>
  <div class="articles">
    <div class="articles-block">
      <div v-for="item in items">
        <column :name="item.name" :articles="item.articles"></column>
      </div>
    </div>
  </div>
</div>
</template>
```
```
<script>
import column from './column'
export default {
  data() {
    return {
      items: [{
        name: "SQTemplate Column",
        articles: [{
          name: "Hybird 搭建零耦合架构从MVC开始",
          href: "http://www.jianshu.com/p/5a03995a6ce1"
        }, {
          name: "Hybird 搭建后端Koa.js并过度到MVVM",
          href: "http://www.jianshu.com/p/846b9f181cb7"
        }, {
          name: "Hybird 搭建前端Vue.js并升级至MVP",
          href: "http://www.jianshu.com/p/8d4a84e3ddaa"
        }, {
          name: "Hybird 搭建路由Router实现组件化",
          href: "http://www.jianshu.com/p/36314d0c0032"
        }, {
          name: "Hybird 搭建客户端实时降级架构",
          href: "http://www.jianshu.com/p/7054a694cfeb"
        }, {
          name: "iOS 执行.py脚本生成解耦架构",
          href: "http://www.jianshu.com/p/47d565bf200e"
        }, {
          name: "iOS 执行.py脚本生成UI层结构",
          href: "http://www.jianshu.com/p/d15379908582"
        }]
      }
      ...
  },
  components: {
    column
  }
}
```
这个就是v-for的基本使用了, 通过在data中的默认参数进行数据绑定, 比起iOS的模型映射还是简单很多, 这里要注意`<column :name="item.name" :articles="item.articles">`, 传入参数带有`:`的是传入引用类型, 而没有`:`的是传入字符串, 这点请注意.

###### column.vue

```
<template>
<div class="column">
  <div class="column-block">
    <div class="column-list">
      <div class="row" v-for="article in articles">
        <a :href="article.href">{{article.name}}</a>
      </div>
    </div>
    <div class="column-title">
      {{name}}
    </div>
  </div>
</div>
</template>
```
```
<script>
export default {
  data() {
    return {

    }
  },
  props: {
    articles: Object,
    name: String
  }
}
</script>
```
这里有一个双重v-for的使用, 注意`articles: Object,`传入的为对象类型而不是String类型.
```
...
.column-list {
  height: 460px;
  overflow: scroll;
}
...
```

其他的css都不用看了, 之前都有讲到过, 这里注意`overflow: scroll`这样设置就能将普通div变成scrollView.


![](http://upload-images.jianshu.io/upload_images/1229762-70b8e5680807d8e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实现的效果也没有太差, 而且文章多了就会支持上下滑动, 颜色也是根据简书的颜色来的. 还是蛮好看的, 好了, 我么来看下整体效果就结束这一种的内容吧.


![整体效果](http://upload-images.jianshu.io/upload_images/1229762-b4c525b8272d8398.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好了, 今天就到这里了, 我们下周再见.

##### About: 

![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.webpack.js)
