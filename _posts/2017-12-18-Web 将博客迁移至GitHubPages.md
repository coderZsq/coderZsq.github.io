---
layout: post
title: Web 将博客迁移至GitHubPages
date: 2017.12.18 22:23
tag: 前端开发
---

> 前段时间简书[饱醉豚](https://www.jianshu.com/u/rHypHw)诋毁程序员的事件导致大量我关注的大佬纷纷同一时间离开简书, 为了表示了对在简书CEO[简叔](http://www.jianshu.com/u/y3Dbcz)的一文[饱醉豚对简书的意义](http://www.jianshu.com/p/eba255074765)的无声的抗议吧.

![](http://upload-images.jianshu.io/upload_images/1229762-8eec1ee0477e4a9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于这个事件有些大佬很愤慨, 比如[饱且撑着](http://www.jianshu.com/c/d0e03deec61e)中收录的长文, 也有些默默的迁移到其他平台, 留下一段链接, 不带走一片云彩.

当然简书官方眼看事情闹大, 可能是想尽快平息事件, 连续发布了两篇公告 `-->` [关于简书签约作者饱醉豚违反简书社区原则的公示说明](https://www.jianshu.com/p/56e59acbf2e3), [关于「作者饱醉豚违反简书社区规则」事件的后续处理公告](https://www.jianshu.com/p/8a736f64f509).

当然先在这里吐个槽:

`牢记简书 “全民创作” 的初心和使命。如果哪天简书开始封杀饱醉豚，那将是偏离初心的开端。如果仅仅因为饱醉豚的存在而导致简书无法增长，那一定是我们产品出现了问题。` 选自: [简叔 饱醉豚对简书的意义](http://www.jianshu.com/p/eba255074765)

`签约作者饱醉豚违反简书社区原则的行为，简书认定属实，正式解除饱醉豚的签约作者身份，并给予警告。` 选自: [关于简书签约作者饱醉豚违反简书社区原则的公示说明](http://www.jianshu.com/p/56e59acbf2e3)

打脸什么的我就不说了, `封杀`和`正式解除`? 这是使用`语言的模糊性`来欺瞒广大的读者朋友么? `仅仅因为饱醉豚的存在而导致简书无法增长`?, 这个`仅仅`两字的使用真是恰到好处, 妙哉, 妙哉!

当然对于我使用信息论的观点进行`带节奏`的行为, 我也不会谴责自己的, 嘿嘿...

对此, 我也来说说我的看法(其实我对于这种诋毁是`没有什么观点`的, 但为了`蹭下热度`还是说说吧, 谁叫一些大佬随便写一篇`离开简书`都能够上`首页投稿`).

好了, 我来认真谈谈这个问题(让我好好蹭蹭热度, 别凉了), 我是`2016`年入驻简书的, 那时候还是一个刚刚成为`程序员的新兵`(为什么不用菜鸟来描述? 因为我现在依然是个`菜鸟`啊!), 不过在简书写文章的岁月中, 对自己的成长还是有一定程度的帮助的(当然也不是说简书有多牛什么的, 其实简书`并没有什么`实质性的帮助), 其实自己写作的时候会对于自己学习到的知识`进行梳理`, 文章发布后的别人的评论也是对自己最好的`及时反馈`, 在简书也认识了很多大佬, 也有机会到`京东`及`饿了么`这种一线互联网公司进行面试(当然因为`水平菜`, 就被拒啦), 当然有机会尝试一下也是极好的, `了解了实力上的差距, 也为今后的学习指引了方向`.

说完`自我介绍`, 接下来也该进行事件的讲解了, 为了尽可能的穷尽事件的来龙去脉(事实证明 真的是闲的蛋疼!), 我快速翻阅了[饱醉豚](https://www.jianshu.com/u/rHypHw)的文章, `敏感内容`已经被`简书查封`, 看看如下简书想让你看到的:

本次事件中相关文章的处理如下：

如下文章经审核判定违反社区规则，依规则予以删除

   · 饱醉豚：《我不骂程序员低素质，我只是去证明他们是`低素质群体`》

   · 饱醉豚：《为什么有这么蠢的程序员？难道程序员真是`低智商福利职业`？》

   · 饱醉豚：《为什么简书的程序员用户里有那么多`mother fucker和恋尸癖`？》

   · 饱醉豚：《为什么中国的程序员`巨婴`这么多》

2.以下文章经审核判定不违反社区规则，但由于在简书社区造成大范围负面影响，予以删除

   · 饱醉豚：《为什么程序员是`出轨率最高的群体`》

   · 饱醉豚：《`笨到学不懂中学物理`怎么办？`逻辑极差`怎么办？当程序员去！》

3.以下文章经审核判定不违反社区规则，不删除

   · 简叔：《饱醉豚对简书的意义》

4.其他涉嫌违规的文章仍然在紧张排查中，简书内容审核人员将逐步进行删除，在此不予以一一公告

上述摘自: [关于「作者饱醉豚违反简书社区规则」事件的后续处理公告](http://www.jianshu.com/p/8a736f64f509)

关于上述内容, 已经被简书官方删除了, 当然在删除之前我也点进去看了一下(谁都有一颗八卦的心不是么), 内容其实感觉和鸡肋一般食之无味, 既没有`形式主义`的文笔, 也没有`走心走肾`的内容, 真怀疑是怎么当上`签约作者`的, 还有简叔的力挺? 真是怪哉, 怪哉...

可能是这两天有什么事情打击到他了吧, 不然怎么会有勇气去攻击一个团体, 要不就是`和简书合作`为了迅速提高自己的热度? 细想我X,果然跑流量啊!! (诶.. 我这种阴谋论和厚黑学...) 

来看看[饱醉豚](https://www.jianshu.com/u/rHypHw)最热的一篇文章好了 `-->` [毁掉你的不是高房价，而是你自己](http://www.jianshu.com/p/faa6359b4ef3)

`170k`人阅读, `2k`人评论, `4k`人点赞?? 还以为是一篇什么文章, 观其大略简直连鸡汤也算不上啊, 全文`单边逻辑`, 没有`数据支持`, 没有`科学认证`, 就连`操纵舆论`的`说服能力`也是`极其低下`的, 这样的文章代表了简书的`实力展现`, 相信也能想象[饱醉豚](https://www.jianshu.com/u/rHypHw)的关注人群了, 果然认知水平需要提升啊.

`高度契合简书 “真实、新鲜、多元 ” 的内容主张`, 我是看出了简书的内容主张了, `所谓真实, 就是逻辑基点基于猜测和想象之上并以为是真的`, `所谓新鲜, 就是对于内容质量毫无追求, 仅仅凭量来达到新鲜`, `所谓多元, 就是接纳各种黑子喷子的节奏, 不经自己的价值过滤而内化自身的行为`, 果然只能说是人各有志吧.

其实, 我感觉大量的大佬出走, 并不是因为程序员这个群体`一碰就炸`, 可能是如我上述而言和简书的`价值观不符`吧, 大佬们都是一些`正直善良`的人, 像我这种道德底线低下的程序员, 可能最应该被[饱醉豚](https://www.jianshu.com/u/rHypHw)抨击吧, 来啊! 对`A`啊!, 就用你最擅长的文字`羞辱`你!

诶... 好了好了, 不蹭热度了, 真是有够闲的, 读到这里, 你以为是一篇水文当然就错了, 因为`大反转`才是世间运行的要义啊!

#### GitHub Pages

今天要分享给大家的是[GitHub Pages](https://pages.github.com/)的使用.

新学习一个技术首先就要看[官方文档](https://pages.github.com/).

![](http://upload-images.jianshu.io/upload_images/1229762-e6f3e774eb4bf905.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一步: 创建一个GitHub仓库名字叫`coderZsq`.github.io, 其中着重符为GitHub用户名, 后面的是一定要这样写(不要问我为什么, 问了我也不知道.)

![](http://upload-images.jianshu.io/upload_images/1229762-63c7005163f02347.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二步: 通过终端克隆仓库到本地.

![](http://upload-images.jianshu.io/upload_images/1229762-8e1e17e585d5c5f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三步: 创建一个index.html页面.

![](http://upload-images.jianshu.io/upload_images/1229762-7580064e9561f14c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第四步: 将代码推送到远程仓库.

![](http://upload-images.jianshu.io/upload_images/1229762-2b4f526411de9cc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后: 通过浏览器访问[https://coderZsq.github.io](https://coderZsq.github.io)就能够看到我们的网页啦~

是不是觉得这次分享的东西有点简单, 是的, 就是那么简单, 但要做一个博客的话, 还是要花点心思, 好在有像[Jekyll](https://jekyllrb.com/)和[Hexo](https://hexo.io/)这类建站工具可以帮我们进行快速的博客搭建.

#### Jekyll

去年的时候用过`Hexo`做过博客, 但是感觉不是太好, 还有一套`hexo`自己的命令, 什么`hexo g`, `hexo d`什么的感觉很不好.

这次我们就用`Jekyll`来进行博客的搭建吧!

和之前一样, 新学习一个技术还是要看[官方文档](https://jekyllrb.com/)

![](http://upload-images.jianshu.io/upload_images/1229762-a5fb758fd82e00e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进行这几步就能够生成一个博客? 诶.. 怎么我好想不行啊, 这年头官方文档也坑人啊... 

原来我们没有`ruby`环境... 如果你安装过`cocoapods`应该就没有这个问题了.

#### Ruby

以下截取网络`ruby环境搭建的文章`:

步骤0 － 安装系统需要的包

```
# For Mac 
# 先安装 [Xcode](http://developer.apple.com/xcode/) 开发工具，它将帮你安装好 Unix 环境需要的开发包
```

步骤1 － 安装 RVM
 
RVM 是干什么的这里就不解释了，后面你将会慢慢搞明白。

```
$ curl -L https://get.rvm.io | bash -s stable
```

期间可能会问你sudo管理员密码，以及自动通过homebrew安装依赖包，等待一段时间后就可以成功安装好 RVM。

然后，载入 RVM 环境（新开 Termal 就不用这么做了，会自动重新载入的）

```
$ source ~/.rvm/scripts/rvm
检查一下是否安装正确

$ rvm -v
rvm 1.22.17 (stable) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
```

步骤2 － 用 RVM 安装 Ruby 环境
  

列出已知的ruby版本

```
$ rvm list known
```


可以选择现有的rvm版本来进行安装（下面以rvm 2.0.0版本的安装为例）

```
$ rvm install 2.0.0
```

同样继续等待漫长的下载，编译过程，完成以后，Ruby, Ruby Gems 就安装好了。

另附：

查询已经安装的ruby

```
$ rvm list
```

卸载一个已安装版本 

```
$ rvm remove 1.9.2
```

步骤3 － 设置 Ruby 版本

RVM 装好以后，需要执行下面的命令将指定版本的 Ruby 设置为系统默认版本

```
$ rvm 2.0.0 --default
```

同样，也可以用其他版本号，前提是你有用 rvm install 安装过那个版本

这个时候你可以测试是否正确

```
$ ruby -v
ruby 2.0.0p247 (2013-06-27 revision 41674) [x86_64-darwin13.0.0]

$ gem -v
2.1.6

```

这有可能是因为Ruby的默认源使用的是cocoapods.org，国内访问这个网址有时候会有问题，网上的一种解决方案是将远替换成淘宝的，替换方式如下：

```
$gem source -r https://rubygems.org/
$ gem source -a https://ruby.taobao.org
```

要想验证是否替换成功了，可以执行：

```
$ gem sources -l  
```

正常的输出结果：

```
CURRENT SOURCES　　　　　　　　　　　　

http://ruby.taobao.org/　　　　　　　　　　　　
```

到这里就已经把Ruby环境成功的安装到了Mac OS X上，接下来就可以进行相应的开发使用了。

有了`Ruby`环境, 我们就能够跑刚才的`Jekyll`文档上的命令了. 但是跑出来的的网站好丑, 怎么办, 我们需要一套皮肤!!!

#### Vno-Jeklly

这里我们就选用`喵神`开发的皮肤`Vno-Jeklly`进行主题设置吧. 首先我们来看看`喵神`的[文档](http://vno.onevcat.com/2016/02/hello-world-vno/)

```
Usage
$ git clone https://github.com/onevcat/vno-jekyll.git your_site
$ cd your_site
$ bundler install
$ bundler exec jekyll serve
Your site with Vno Jekyll enabled should be accessible in http://127.0.0.1:4000.

Configuration
All configuration could be done in _config.yml. Remember you need to restart to serve the page when after changing the config file. Everything in the config file should be self-explanatory.

Background image and avatar
You could replace the background and avatar image in assets/images folder to change them.
```

```
To add new posts, simply add a file in the _posts directory that follows the convention YYYY-MM-DD-name-of-post.ext and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.
```

照做后访问`http://127.0.0.1:4000`就能够看到欢迎页面.

![](http://upload-images.jianshu.io/upload_images/1229762-28ba4fd4a5218446.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们将其更改后`git push`到刚才`GitHub Pages`设置的仓库后, 我们就能够看到我们的博客站点啦!!

#### gh-pages

对了, 关注我的朋友们肯定知道我之前有一个`H5简历`的一个系列, 我们现在就把它也搭建到`github`上.

那怎么做呢?? `coderZsq.github.io`这个`GitHub Pages`只有一个啊... 好在还有`gh-pages`能够帮我们完成这项任务.

什么是`gh-pages`? 我们来看看[文档](https://www.npmjs.com/package/gh-pages)

原来是创建一个子分支来进行`GitHub Pages`啊. `SOGA`

以下截取网络`gh-pages分支搭建的文章`:

在github上建立gh-pages分支

为什么要建立gh-pages分支呢，因为github项目的静态页面解析需要这个名字的分支

进入到你想要上传的文件夹下：

```
cd text
```

git初始化

```
git init
```

创建gh-pages分支

```
git checkout --orphan gh-pages
```

添加文件到暂存区

```
git add .
```

添加信息

```
git commit -m "This is add message"
```

或者不写上面的`git add .`直接写` git commit -a -m \"First pages commit\"`这个`-a`参数我查了之后说是对`git add .`的替代，但我不建议大家使用。

添加仓库

```
git remote add origin git@github.com:username/project.git
```

部署你的项目到`github`

```
git push origin gh-pages
```

我们通过[Web Vue项目速转.htm静态网页
](https://coderzsq.github.io/2017/07/Web-Vue%E9%A1%B9%E7%9B%AE%E9%80%9F%E8%BD%AC.htm%E9%9D%99%E6%80%81%E7%BD%91%E9%A1%B5/)中的方法, 将我们的`Vue`项目压成一个静态页面, 并推送到远程的`gh-pages`分支, 就搞定啦!

![](http://upload-images.jianshu.io/upload_images/1229762-e9c3edaac26e6e1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样我们的简历也通过`GitHub Pages`发布到线上啦!! 可以通过[https://coderzsq.github.io/coderZsq.webpack.js/#/](https://coderzsq.github.io/coderZsq.webpack.js/#/)进行访问!! 看这个路径就知道和之前创建方法的区别了.

#### 最后

到这里, 我感觉已经很详细的讲述了`GitHub Pages`如何搭建了, 其实并没有什么技术含量, 只是因为`饱醉豚`事件, 不想继续将文章挂靠在简书这个第三方平台了, 还是挂在`GitHub Pages`上更为可靠.

当然简书这里我也会继续更新, 当然首发会在`GitHub Pages`上, 简书和掘金只做同步处理. 我才不会因为`饱醉豚`这种`无脑喷`的影响就让我少了一个推广渠道, 正所谓`语言是思想的牢笼`, 才不会和这种只能驾驭低等文字的人一般见识呢. 两般见识怎么样? `三板斧`砍死你额!!!

诶... 砍砍杀杀这种见血的多不好, 诛心才是我们行走江湖的第一要诀不是么? ... 诛什么心, 还是修心吧, 最近不是流行什么`佛系`云云... `善哉`...

![](http://upload-images.jianshu.io/upload_images/1229762-f3593bc6698ef19e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后声明: 以后文章首发地址更改为[https://github.com/coderZsq/coderZsq.github.io](https://github.com/coderZsq/coderZsq.github.io) 我这里精编了目录记录我的学习和成长, 虽然很`肤浅`但是很开心~~


![点击下方链接跳转!!](http://upload-images.jianshu.io/upload_images/1229762-3f8925783027b3fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 [🌟 简历源码 请点这里🌟 >>> 喜欢的朋友请点喜欢 >>> 下载源码的同学请送下小星星 >>> 有闲钱的壕们可以进行打赏 >>> 小弟会尽快推出更好的文章和大家分享 >>> 你的激励就是我的动力!! ](https://github.com/coderZsq/coderZsq.webpack.js)
