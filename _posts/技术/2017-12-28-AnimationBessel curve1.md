---
layout: blog
technology: true 
background: purple
background-image: http://p4wibncwo.bkt.clouddn.com/casey-horner-423559-unsplash.jpg
title:  "贝塞尔曲线绘制1"
date:   2017-12-28
category: 技术
tags:
- iOS
- Core Graphics
- 贝塞尔动画
- Animation
---


我们日常生活中用的贝塞尔曲线的地方还是比较多的，我见过的例如,自定义侧边栏的动画效果,表格的下拉效果,视图的波浪循环的效果,数据分析时的折线图,自定义TabBar的时候也会用到等等，很多自定义的想要炫酷的效果都得用到。我也一直想做一些自己喜欢的效果，所以，就腾出时间，潜心研究研究。

首先我们要扫扫盲,比较清楚的去了解它:
http://blog.csdn.net/cdnight/article/details/48468653

贝塞尔曲线包括了
>一阶曲线
给定点P0、P1，线性贝塞尔曲线只是一条两点之间的直线。这条线由下式给出：
![线性公式.png](http://upload-images.jianshu.io/upload_images/1120896-25e74e3128f92dad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>二阶曲线
二次方贝塞尔曲线的路径由给定点P0、P1、P2的函数B（t）追踪：
![二次方公式.png](http://upload-images.jianshu.io/upload_images/1120896-de0a3fbaa242e694.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>三阶曲线
P0、P1、P2、P3 四个点在平面或在三维空间中定义了三次方贝塞尔曲线。曲线起始于 P0 走向 P1 ，并从 P2 的方向来到 P3 。一般不会经过 P1 或 P2 ；这两个点只是在那里提供方向资讯。 P0 和 P1 之间的间距，决定了曲线在转而趋进 P2 之前，走向 P1 方向的“长度有多长”。
曲线的参数形式为：
![三次方公式.png](http://upload-images.jianshu.io/upload_images/1120896-7cf3b17525b3bff8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>高阶曲线
n阶贝塞尔曲线可如下判断，给定点P0、p1、...、Pn，其贝塞尔曲线即
![n阶曲线.jpg](http://upload-images.jianshu.io/upload_images/1120896-b458c3d42d48ef4e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

WTF? 公式太抽象? 那就来看看动画吧。借用上面那篇文章的动图
 
![曲线动态图.gif](http://upload-images.jianshu.io/upload_images/1120896-2ddbbb4500cbd7ea.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到为二阶曲线,包含了P0,P1,P2三个点. 

这里有个网址比较容易的让我们能够直观方便的绘制出曲线:
https://aaaaaaaty.github.io/bezierMaker.js/playground/playground.html

####为了让程序有趣点,我们来画个眉毛?（直线）
![眉毛.png](http://upload-images.jianshu.io/upload_images/1120896-d6288aec964559f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

当然咯,我们从简单的一步一步来。
首先是如何定义贝塞尔曲线
```
//先创建实例
UIBezierPath *bPath = [UIBezierPath bezierPath];

//然后我们可以设置它的颜色:
UIColor *color = hexStrColor(@"#FF8C69");
[color set];

//然后就是它的线条宽度:
bPath.lineWidth = 2.0;

基础的属性设置先用到这些。
接下来我们就要开始绘画简单的线:
线就是两个点组成的，所以我们首先设定一个起始点:
//设定屏幕中间点,这个就起始点就自己根据需求设置了。
CGPoint center = (CGPoint){kSCREENWIDTH/2,kSCREENHEIGHT/2};
将起始点移动该坐标点
[bPath moveToPoint:(CGPoint){center.x-100,center.y-50}];

//然后设定末尾的点:
[bPath addLineToPoint:(CGPoint){center.x-50,center.y-50}];

//画完记得关闭绘画的路径:
[bPath closePath];
//开始绘制之前组合出来的线
[bPath stroke];
```


####然后我们再来画眼睛?(画实心圆)
![眼睛.png](http://upload-images.jianshu.io/upload_images/1120896-274a4822fc8cc678.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)
```
//移动到画的中心点
[bPath moveToPoint:(CGPoint){center.x-50,center.y-40}];

//画圆
/**
 *  以某个中心点画弧线
 *  @param center     指定了圆弧所在正圆的圆心点坐标
 *  @param radius     指定了圆弧所在正圆的半径
 *  @param startAngle 指定了起始弧度位置  注意: 起始与结束这里是弧度 
 *  @param endAngle   指定了结束弧度位置  
 *  @param clockwise  指定了绘制方向，以时钟方向为判断基准   看下图
 */
[bPath addArcWithCenter:(CGPoint){center.x-75,center.y-40} 
radius:10
startAngle:0
endAngle: DEGREES_TO_RADIANS(360)
clockwise:YES];

[bPath closePath];
//file 填充路径
[bPath fill];
```
如果设定pi的值不那么直观,我们可以用下面的宏直接填写角度
` #define   DEGREES_TO_RADIANS(degrees)  ((M_PI * degrees)/ 180)`

angle的值可以参照该图
![圆 .png](http://upload-images.jianshu.io/upload_images/1120896-5c6d3cc3f09cf913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####嗯,我们再画个嘴(弧线)
我们画个由三个点组成的曲线。
![嘴.png](http://upload-images.jianshu.io/upload_images/1120896-4c71c17f6fdd0fbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

不太明白的可以用
https://aaaaaaaty.github.io/bezierMaker.js/playground/playground.html 
自己画一个看看:
![bezier.png](http://upload-images.jianshu.io/upload_images/1120896-e5e0c1fd22566e8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/440)

可以看到,我们利用三个点绘制出我们想要的弧线。
```
UIBezierPath *bPathB = [UIBezierPath bezierPath];
bPathB.lineWidth = 2.0;
[bPathB moveToPoint:(CGPoint){center.x-60,center.y}];
[bPathB addQuadCurveToPoint:(CGPoint){center.x+60,center.y} controlPoint:(CGPoint){center.x,center.y+50}];
[bPathB stroke];
```
然后你通过上述方法会发现三个点竟然不是顺序的。是的。moveToPoint是p1,CurveToPoint是p3,而controlPoint是p2。

然后我们再利用其它的方式画个空心圆当做它的大肥脸。
![脸.png](http://upload-images.jianshu.io/upload_images/1120896-97c9c6ab6e560a00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

```
//创建CAShapeLayer对象
CAShapeLayer *shapeLayer = [CAShapeLayer layer];
shapeLayer.frame = CGRectMake(center.x-150, center.y-150, 300,300);//设置shapeLayer的尺寸和位置
shapeLayer.fillColor = [UIColor clearColor].CGColor;//填充颜色为ClearColor
//设置线条的宽度和颜色
shapeLayer.lineWidth = 2.0f;
shapeLayer.strokeColor = hexStrColor(@"#FF8C69").CGColor;
//创建一个圆形贝塞尔曲线
UIBezierPath *aPath = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0,0,300,300)];
//将贝塞尔曲线设置为CAShapeLayer的path
shapeLayer.path = aPath.CGPath;
//将shapeLayer添加到视图的layer上
[self.layer addSublayer:shapeLayer];
```
这个实现方式就比较简单了，就是你绘制了一个CAShapeLayer图层，然后利用它的路径进行绘制。
需要注意的是,shapeLayer的frame设置的x,y不是圆心，而是外圆开始的点。 那个宽高设不设无所谓。宽高设定由UUIBezierPath决定。

其他的绘制我们就简单的绘制一下了,毕竟我们的重点是动画。
####矩形
![矩形.png](http://upload-images.jianshu.io/upload_images/1120896-cf6d87db7f92c1da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
绘制矩形只需要调用bezierPathWithRectF方法,然后填入参数就可以了。
UIBezierPath *rectPath = [UIBezierPath bezierPathWithRect:(CGRect){10,kSCREENHEIGHT-154,90,60}];
[rectPath setLineWidth:2.0];
[rectPath stroke];//实心的需要使用fill方法
```
####圆角
![圆角.png](http://upload-images.jianshu.io/upload_images/1120896-60e563708f28c7fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

生成圆角的方法:
```
UIBezierPath *roundedRectPath = [UIBezierPath bezierPathWithRoundedRect:(CGRect){110,kSCREENHEIGHT-154,90,60}
                                                          byRoundingCorners:UIRectCornerTopLeft
                                                                cornerRadii:(CGSize){20,20}];
[roundedRectPath setLineWidth:2.0];
[roundedRectPath fill];
```
```
cornerRadii: 表示圆角的大小咯，范围越大，圆角越圆。
奇怪的是指定了宽度就可以了，高度怎么写都不会影响。毕竟宽度可以表示半径，这高度有啥用呢

byRoundingCorners: 参数的值就是指定生成哪个方位的圆角

typedef NS_OPTIONS(NSUInteger, UIRectCorner) {
    UIRectCornerTopLeft     = 1 << 0, 左上角
    UIRectCornerTopRight    = 1 << 1, 右上角
    UIRectCornerBottomLeft  = 1 << 2, 左下角
    UIRectCornerBottomRight = 1 << 3, 右下角
    UIRectCornerAllCorners  = ~0UL 全部
};
```
#### 3阶曲线

![三阶曲线.png](http://upload-images.jianshu.io/upload_images/1120896-6539e508966da07c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3阶曲线设置如下图:
![三阶曲线.jpeg](http://upload-images.jianshu.io/upload_images/1120896-bf5e8033447d9a3f.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以通过下面的动态图看一下:
![三阶曲线.gif](http://upload-images.jianshu.io/upload_images/1120896-6899d551cff5b137.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分为起点,终点，还有两个控制点。
可以用下面的方法进行实现:
```
    UIBezierPath *curveRectPath = [UIBezierPath bezierPath];
    [curveRectPath moveToPoint:(CGPoint){210,kSCREENHEIGHT-124}];
    [curveRectPath addCurveToPoint:(CGPoint){290,kSCREENHEIGHT-124}
                     controlPoint1:(CGPoint){250,kSCREENHEIGHT-154}
                     controlPoint2:(CGPoint){250,kSCREENHEIGHT-94}];
    [curveRectPath setLineWidth:2.0];
    [curveRectPath stroke];
```

DEMO:
https://github.com/yanggenwei/GWAnimation/tree/master



