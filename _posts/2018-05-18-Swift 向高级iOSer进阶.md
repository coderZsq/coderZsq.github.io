---
layout: post
title: Swift 向高级iOSer进阶
date: 2018.05.18 22:46
tag: 移动开发
---

>GitHub Repo：[coderZsq.target.swift](https://github.com/coderZsq/coderZsq.target.swift)
<br/>Follow: [coderZsq · GitHub](https://github.com/coderZsq)
<br/>Resume: [https://coderzsq.github.io/coderZsq.webpack.js/#/](https://coderzsq.github.io/coderZsq.webpack.js/#/)

#### 日常扯淡

前段日子写了篇面经, 得到了`掘金`的`征文活动`的`三等奖`还是非常开心, 但是被寒神说`面试不过就泄题`, 影响不好, 我想想也是, 马上就把大厂的名字给`抹掉`了, 但被转载的就`无能为力`了, 看到下面好多喷的, 真是背后一凉, 一首凉凉送给自己, 造成的伤害无法挽回, 再此郑重道歉, 再也不写面经了. 希望以后还能获得`大厂面试`的机会!

被`大厂`刷掉后, 我心情难以平复, 因为其实我是做了`充足的准备`来的, 但是还是`实力上有差`距, 诶... 还是想想改如何`提升`自己的水平吧!

其实对于自己, 我其实并不知道`iOS`该如何进行学习, 也不知道`技术这条路`我半道转的是不是正确, 更不知道在当今环境下我这种水平的`iOS`开发者是否还有存在的必要.

说起转行做`iOS`到现在, 从`Objective-C`基础语法学起, 认为`OC`是最好的语言, 学会做几个简单的`UITableView`页面, 能播放`音频视频`, 觉得自己真是`转了个高大上的职业`, 现在想想, 真是`肤浅`的让人忍不住发笑. 

我把做`iOS`这段学习经历分为五个阶段:

第一个阶段: 是刚刚转行进入互联网企业, 那时候觉得学好`CALayer`掌握了一些酷炫的动画(`贝塞尔曲线`), 感觉就已经比大多数人都强了, 那时候还把常用的工具类封装起来, 就觉得, 嗯, 自己还不错.

第二个阶段: 就是瞎学些有的没的, 如`Swift`, `JavaScript`, `Java`什么的, 觉得自己全栈了, 什么都会了, 很牛逼, 你看还自己能够写一个前端简历, 还做过公司的前端项目, 服务器开发也会了, 自信心爆棚啊, 觉得自己真是无所不能.

第三个阶段: 写了一个`架构生成器`, 觉得自己`所向披靡`了, 公司项目都在用我写的架构, `用着我制定的规则`, 尼玛不就是一个简单的`字符串替换`, 嘚瑟个啥... 而且对于架构的理解`肤浅至极`... 

第四个阶段: 察觉到了自己的薄弱, 开始学习`iOS`的底层原理, 学习了`C\C++`的语法, 学习了`Linux`基础, 学习了`8086, ARM64`汇编, 了解了一些`自认为`还是比较深的知识点. 觉得自己前途还是有希望的.

第五个阶段: 也就是被刷掉之后的现在, 其实我现在也很迷茫, 也不知道现在学的东西到底有没有用, 也不知道大厂到底要什么样的人才(只知道牛逼就行...), 但我也通过把知识点进行分类进行进阶吧. 把最近的学习总结分享出来和大家一起讨论, 像我这种水平的玩家到底该怎么玩耍.

#### 学习计划

![](https://upload-images.jianshu.io/upload_images/11636671-fb9392e05bca48e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我把最近学习的方面全部都整理在`StudyNote`这个里面了, 可以看得出, 这个阶段, 我明显的就是像要学习加强`算法与数据结构`方面的, 可能是因为被大厂刷的有阴影了吧.

![](https://upload-images.jianshu.io/upload_images/11636671-ba1c10cca3368998.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除了`数据结构与算法`, 最近刚看完的`CS193p`的教程, 白胡子老头的这个视频的`质量很高`, 发现了很多我平时忽略的东西, 而且觉得很多我以为的写法从`本质上`都是有问题的.

![](https://upload-images.jianshu.io/upload_images/11636671-0a475c3c731309cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有就是`objc.io`的这几本书了, 质量挺高的, `函数式Swift`的学习对我很有帮助. 感觉就像是打开了新世界.

![](https://upload-images.jianshu.io/upload_images/11636671-22d60025d6ff44bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


还有就是排在后面的学习计划: `python`, `数据分析`, `机器学习`, `深度学习`.可以看得出, 其实这些都学完, 其实也不知道能够干什么, 无所事事诶... 但至少这些都是我能够找到的比较`高质量`的资料了, 如果有其他高质量的资料, 欢迎进行资料共享hhhh~

#### 算法学习

果然, 算法是挡在(`复制黏贴`)程序员和(`正常`)程序员之前的一条很难跨越的鸿沟, 我这里不是说, 你会写`快速排序`,`二分查找`, `深度广度优先`这类大家都能够背出来的东西就叫做掌握算法了, 我在`Coursera`上看了`加州大学`的算法课, 才知道该如何设计算法, 但由于是英语的, 而且没有代码, 纯数学的看不太懂, 所以转向了`北京大学`的算法课, 以下就是我最新学到的算法和你分享.

![](https://upload-images.jianshu.io/upload_images/11636671-972f7996faf07e9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 这个只是`算法基础`的第二节课, 你能想象这是算法基础么? 光是这个题目, 我就看了老半天, 大意是点击一个灯, 上下左右的灯会`自动点亮(熄灭)`, 当随机给出点亮熄灭数的时候, 需要算出点击哪几个灯可以将所有的灯点亮或熄灭.
 
 这种算法题, 真是`闻所未闻见所未见吧`, 而且这是算法基础的开头的课... 我在想难道那些大厂的人做这种题`跟玩的一样`么, 想起了`面试官的微笑`, 那可真是有力量的微笑呢.
 
```c
#include <stdio.h>

int puzzle[6][8], press[6][8];
/*
推测验证过程：
根据第一行猜测
*/
bool guess() {
    int c, r;
    //根据press第1行和puzzle数组，计算press其他行的值
    for(r=1; r<5; r++) {
        for(c=1; c<7; c++) {
            press[r+1][c]=(puzzle[r][c]+press[r][c]+press[r-1][c]+press[r][c-1]+press[r][c+1])%2;
        }
    }
    //判断所计算的press数组能否熄灭第5行的所有灯
    for(c=1; c<7; c++) {
        if ((press[5][c-1]+press[5][c]+press[5][c+1]+press[4][c])%2 != puzzle[5][c]) {
            return false;
        }
    }
    return true;
}

/*
枚举过程：
对press第1行的元素press[1][1]~press[1][6]的各种取值进行枚举
*/
void enumerate() {
    int c;
    bool success; //这个变量时当时定义了没排上用场吧，NodYoung注
    for(c=1; c<7; c++) {
        press[1][c]=0;
    }
    while(guess()==false) {
        press[1][1]++;
        c=1;
        while(press[1][c]>1) {  //累加进位
            press[1][c]=0;
            c++;
            press[1][c]++;
        }
    }
    return ;
}

int main() {
    int cases, i, r, c;
    scanf("%d", &cases);
    for(r=0; r<6; r++) {
        press[r][0]=press[r][7]=0;
    }
    for(c=0; c<7; c++) {
        press[0][c]=0;
    }
    for(i=0; i<cases; i++) {
        for(r=1; r<6; r++) {
            for(c=1; c<7; c++) {
                scanf("%d", &puzzle[r][c]); //读入输入数据
            }
        }
        enumerate();
        printf("PUZZLE#%d\n", i+1);
        for (r=1; r<6; r++) {
            for (c=1; c<7; c++) {
                printf("%d ", press[r][c]);
            }
            printf("\n");
        }
    }
    return 0;
}
```
这是`北大老师`视频里给出的算法的答案, 讲的很好, 但是说真的听的是一知半解, 原因在于不知道为什么, 这些网课都不是`在线编译`的, 而是直接对着`代码分析`, 上面的代码摘抄自-> 可以搜索`熄灯问题`.
 
```swift
var puzzle = [[Int]](repeating: [Int](repeating: 0, count: 8), count: 6)
var press = [[Int]](repeating: [Int](repeating: 0, count: 8), count: 6)
```

```swift
func guess() -> Bool {
    for r in 1..<5 {
        for c in 1..<7 {
            press[r + 1][c] = (puzzle[r][c] + press[r][c] + press[r - 1][c] + press[r][c - 1] + press[r][c + 1]) % 2
        }
    }
    for c in 1..<7 {
        if (press[5][c - 1] + press[5][c] + press[5][c + 1] + press[4][c]) % 2 != puzzle[5][c] {
            return false
        }
    }
    return true
}
```
```swift
func enumerate() {
    var c = 1
    for _ in 1..<7 {
        press[1][c] = 0
        while (guess() == false) {
            press[1][1] += 1
            c = 1
            while press[1][c] > 1 {
                press[1][c] = 0
                c += 1
                press[1][c] += 1
            }
        }
        c += 1
    }
}
```
```swift
class Enumerate {
    
    static func main() {
        let cases = 1
        for r in 0..<6 {
            press[r][0] = 0
            press[r][7] = 0
        }
        for c in 1..<7 {
            press[0][c] = 0
        }
        for i in 0..<cases {
            for r in 1..<6 {
                for c in 1..<7 {
                    puzzle[r][c] = 2.arc4random
                }
            }
            enumerate()
            print("PUZZLE #\(i + 1)")
            for r in 1..<6 {
                for c in 1..<7 {
                    print(puzzle[r][c], terminator: "")
                }
                print()
            }
            print("== press ==")
            for r in 1..<6 {
                for c in 1..<7 {
                    print(press[r][c], terminator: "")
                }
                print()
            }
            print()
        }
    }
}
``` 

以上是我学习的时候转换成`swift`表达的, 不为什么, 只是用来熟悉`Swift`语法罢了, 毕竟`OC`也不知道还能活个几年了.

```swift
PUZZLE #1
011010
001110
010011
000101
100000
== press ==
001001
000101
001010
001101
011110
```
 
但是光看代码, 很难看懂这个结果到底是正确还是不正确的... 因为跑出来是这样的一个东西, 但是有些地方还是可以讲一下, 比如是`外面包了一圈0`来避免冗余逻辑判断, 用`2进制`进位的方法进行运算, 还是有学到一些皮毛的.

#### 看得见的算法

当然, 这种文字上的描述, 很难有深刻的印象的, 所以, 我就在想是否可以把这个熄灯游戏给做出来, 再自己测试一下呢? 想到就干吧!!

通过`CS193p`的学习, 对于`画UI`方面有了全新的认识, 该如何添加约束, `MVC`到底怎么写, 以至于我以前理解的感觉完全就是错的, 正好趁这个机会来练练手.

![](https://upload-images.jianshu.io/upload_images/11636671-f776b3d1909ee702.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们通过`StoryBoard`先把`View`画好, 不得不说`UIStackView`真是好用到爆!!

```swift
import Foundation

struct Matrix {
    var rows: Int
    var columns: Int
}

struct LightSwitch {
    
    private var puzzle: [[Int]]
    private var matrix: Matrix
    var lights = [Int]()
    
    mutating func lightUp(index: Array<Any>.Index?) {
        guard let index = index else { return }
        var m = Matrix(rows: 0, columns: 0)
        if index <= matrix.rows {
            m.columns = index + 1
        } else {
            m.columns += index % matrix.columns + 1
        }
        for i in 0...index {
            if i % matrix.columns == 0 {
                m.rows += 1
            }
        }
        puzzle[m.rows][m.columns] = puzzle[m.rows][m.columns] == 0 ? 1 : 0
        puzzle[m.rows + 1][m.columns] = puzzle[m.rows + 1][m.columns] == 0 ? 1 : 0
        puzzle[m.rows][m.columns + 1] = puzzle[m.rows][m.columns + 1] == 0 ? 1 : 0
        puzzle[m.rows - 1][m.columns] = puzzle[m.rows - 1][m.columns] == 0 ? 1 : 0
        puzzle[m.rows][m.columns - 1] = puzzle[m.rows][m.columns - 1] == 0 ? 1 : 0
        lights.removeAll()
        for r in 1..<matrix.rows + 1 {
            for c in 1..<matrix.columns + 1 {
                lights.append(puzzle[r][c])
            }
        }
    }
    
    init(matrix: Matrix) {
        self.matrix = matrix
        puzzle = [[Int]](repeating: [Int](repeating: 0, count: matrix.columns + 2), count: matrix.rows + 2)
        for r in 1..<matrix.rows + 1 {
            for c in 1..<matrix.columns + 1 {
                puzzle[r][c] = 2.arc4random
                lights.append(puzzle[r][c])
            }
        }
        print("========")
        for r in 0..<matrix.rows + 2 {
            for c in 0..<matrix.columns + 2 {
                print(puzzle[r][c], terminator: "")
            }
            print()
        }
        print("========")
    }
}
```
`Model`代码, 原来`MVC`的`M`需要这样写的, 以前都只是认为是简单的`数据结构来的`真是肤浅, 这种直接业务逻辑写在`M`里面的的写法真是好用到爆啊!

```swift
import UIKit

extension UIColor {
    var toImage: UIImage {
        let bounds = CGRect(origin: .zero, size: CGSize(width: 1.0, height: 1.0))
        let renderer = UIGraphicsImageRenderer(bounds: bounds)
        return renderer.image { context in
            self.setFill()
            context.fill(CGRect(x: 0.0, y: 0.0, width: 1.0, height: 1.0))
        }
    }
}

extension Int {
    var arc4random: Int {
        if self > 0 {
            return Int(arc4random_uniform(UInt32(self)))
        } else if self < 0 {
            return -Int(arc4random_uniform(UInt32(self)))
        } else {
            return 0
        }
    }
}

class ViewController: UIViewController {
    
    @IBOutlet var lights: [UIButton]! {
        didSet {
            for (index, light) in lights.enumerated() {
                light.setBackgroundImage(UIColor.yellow.toImage, for: .normal)
                light.setBackgroundImage(UIColor.darkGray.toImage, for: .selected)
                light.isSelected = switchs.lights[index] == 1 ? true : false
            }
        }
    }
    
    @IBAction func lightUp(_ sender: UIButton) {
        switchs.lightUp(index: lights.index(of: sender))
        for (index, light) in lights.enumerated() {
            light.isSelected = switchs.lights[index] == 1 ? true : false
        }
        if Set(switchs.lights).count == 1 {
            let alert = UIAlertController(title: "Congratulation", message: "You made all light up successfully", preferredStyle: .alert)
            alert.addAction(UIAlertAction(
                title: "again",
                style: .default,
                handler: { [weak self] _ in
                    self?.restart()
                }
            ))
            present(alert, animated: true)
        }
    }
    
    @IBAction func restart(_ sender: UIButton? = nil) {
        switchs = LightSwitch(matrix: Matrix(rows: 5, columns: 6))
        for (index, light) in (self.lights.enumerated()) {
            light.isSelected = self.switchs.lights[index] == 1 ? true : false
        }
    }
    
    var switchs: LightSwitch = LightSwitch(matrix: Matrix(rows: 5, columns: 6))
}
```

`Controller`的代码, 这才能够真正理解什么叫做控制器用来协调`View`和`Model`的交互, `Model`和`View`毫无关联, 这才是`iOS`的正确写法啊. 

<img src="https://upload-images.jianshu.io/upload_images/11636671-0f1a14bb4740313f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="375" style="margin: clear: both; display: block; margin:auto; 0;"/>

运行了一下, 果然白胡子大叔没有骗我, 跑的`6到飞起`~

![](https://upload-images.jianshu.io/upload_images/11636671-45d83dfa5320297d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到的是, 终端打印的矩阵和界面上显示的是一一对应的, 从`北大老师`学到的外面包一圈的方法也是特别好用的.

#### 算法测试

```swift
    init(matrix: Matrix) {
        self.matrix = matrix
        puzzle = [[Int]](repeating: [Int](repeating: 0, count: matrix.columns + 2), count: matrix.rows + 2)
        for r in 1..<matrix.rows + 1 {
            for c in 1..<matrix.columns + 1 {
//                puzzle[r][c] = 2.arc4random
                puzzle[1][1] = 1
                lights.append(puzzle[r][c])
            }
        }
        print("========")
        for r in 0..<matrix.rows + 2 {
            for c in 0..<matrix.columns + 2 {
                print(puzzle[r][c], terminator: "")
            }
            print()
        }
        print("========")
    }
```

<img src="https://upload-images.jianshu.io/upload_images/11636671-b60492f97b2c7916.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="375" style="margin: clear: both; display: block; margin:auto; 0;"/>

我们将初始状态从随机数改成只暗一个灯, 位置是[1][1]

```swift
class Enumerate {
    
    static func main() {
        let cases = 1
        for r in 0..<6 {
            press[r][0] = 0
            press[r][7] = 0
        }
        for c in 1..<7 {
            press[0][c] = 0
        }
        for i in 0..<cases {
            for r in 1..<6 {
                for c in 1..<7 {
//                    puzzle[r][c] = 2.arc4random
                    puzzle[1][1] = 1
                }
            }
            enumerate()
            print("PUZZLE #\(i + 1)")
            for r in 1..<6 {
                for c in 1..<7 {
                    print(puzzle[r][c], terminator: "")
                }
                print()
            }
            print("== press ==")
            for r in 1..<6 {
                for c in 1..<7 {
                    print(press[r][c], terminator: "")
                }
                print()
            }
            print()
        }
    }
}

```

我们把`北大算法`也改成对应的[1][1]

```swift
PUZZLE #1
100000
000000
000000
000000
000000
== press ==
000111
101010
101100
001000
110000
```
可以看懂只要按下全部位置对应为`1`的按钮就可以将所有灯都打开了, 我们来试一下.

<img src="https://upload-images.jianshu.io/upload_images/11636671-da37f8fa3f13b5bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="375" style="margin: clear: both; display: block; margin:auto; 0;"/>

经过好几次测试, 可以看见, `算法是正确`的, 我也学到了这个算法`背后的思维`, 更通过了写了一个`Demo`来证明了算法的正确性. 这个`Demo`的难点在于`向量`和`矩阵`之间的互相转换, 这里为什么不说`一维数组`和`二维数组`呢? , 原因在于`吴恩达`的机器学习课程中也教会了我一些比较厉害的算法, 比如`梯度下降`之类的.

好了, 现在写文章没有之前频繁了, 原因在于之前那些文章都`太水`, `太肤浅`, 写了对自己也没有太大的意义, 被人看到也只会觉得是垃圾而已... 所以在我`第五阶段的学习`后, 希望能够有机会进入一家大厂继续深造吧!


最后 本文中所有的源码都可以在github上找到:
>GitHub Repo：[coderZsq.target.swift](https://github.com/coderZsq/coderZsq.target.swift)
<br/>Follow: [coderZsq · GitHub](https://github.com/coderZsq)
<br/>Resume: [https://coderzsq.github.io/coderZsq.webpack.js/#/](https://coderzsq.github.io/coderZsq.webpack.js/#/)





