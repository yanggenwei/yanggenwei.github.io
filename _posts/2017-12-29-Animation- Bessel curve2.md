---
layout: post
title: "利用贝塞尔曲线制作动画"
date: 2017-12-29
excerpt: "Animation"
tags: [iOS][Animation]
post: true
comments: true
---
<small>
在贝塞尔曲线(1)中，我们介绍了贝塞尔曲线的绘制,但是那是固定的,运用场景很少,运用更多的是一些动画效果。
而前端主要的功能就是负责貌美如花,所以掌握一定的动画技巧还是有必要的。
所以呢，我就开始从简单的慢慢研究吧。

##下拉动画
首先是如下的效果:
![下拉动画.gif](http://upload-images.jianshu.io/upload_images/1120896-347576c2e5c3a9e2.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

首先需要分析几个问题:
1. 如何绘制二阶曲线
2. 如何让曲线跟随着下拉进行变化

问题的解决有很多种,我写下的只是我的解决思路:
```
1. 创建一个视图,重写它的drawRect:方法

UIBezierPath *bezier = [UIBezierPath bezierPath];
UIColor *color = hexStrColor(@"#FF8C69");
[color set];
bezier.lineWidth = 1.0;
[bezier moveToPoint:(CGPoint){0,0}];
[bezier addQuadCurveToPoint:(CGPoint){self.width,0} controlPoint:(CGPoint){self.width/2,self.offsetY}];
[bezier fill];
```
通过上面的方法,我们绘制了二阶曲线,那么就是解决第二个问题了。

想让我们的表格下拉的时候，视图也跟着变化,那么肯定需要知道我们表格下拉的具体数值,所以需要实现UIScrollViewDelegate的scrollViewDidScroll方法
```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{}
```

通过该方法我们实时的知道表格的Y轴偏移量,所以我们还需要在之前的视图中添加一个变量:
`@property (nonatomic,readwrite,unsafe_unretained)CGFloat offsetY;`
通过该变量来实时的控制二阶曲线的controlPoint参数的Y值。

接下来,我们在scrollViewDidScroll方法中写下如下代码。
```
self.headerView.offsetY = -scrollView.contentOffset.y;
[self.headerView setNeedsDisplay];
[self.headerView setFrame:(CGRect){0,statusBarHeight,kSCREENWIDTH,- scrollView.contentOffset.y}];
```
因为我们需要在滑动的时候实时的改变曲线的拐点,然后生成对应的曲线。也就是我们需要实时的重新绘制视图,也就是调用drawRect: 方法.所以我们需要调用setNeedsDisplay方法。通过在外部调用setNeedsDisplay可以是视图重新调用drawRect方法

>setNeedsLayout会默认调用layoutSubViews, setNeedsDisplay会调用drawRect:方法

##加载动画
接下来我们再做个加载动画，具体的效果如下:
![加载动画.gif](http://upload-images.jianshu.io/upload_images/1120896-f72fff75363d19ee.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是个根据路径变化的动画，也是我在很多软件上经常看到的一种加载动画。所以就自己试试能不能做不出来。
先创建三个球,哈哈，先把我们要玩的球画出来:
```
CGFloat radius = 20;
CGFloat marginLeft = 10;
//第一颗球
CAShapeLayer *fLayer = [CAShapeLayer new];
fLayer.backgroundColor =  getColor(102, 170, 238, 1).CGColor;
[fLayer setFrame:(CGRect){self.width/2-(radius*2+marginLeft*2)/2,self.height/2,radius,radius}];
fLayer.cornerRadius = radius/2;
//第二颗球
CAShapeLayer *sLayer = [CAShapeLayer new];
sLayer.backgroundColor = getColor(102, 170, 238, 0.5).CGColor;
[sLayer setFrame:(CGRect){marginLeft(fLayer)+marginLeft,self.height/2,radius,radius}];
sLayer.cornerRadius = radius/2;
//第三颗球
CAShapeLayer *tLayer = [CAShapeLayer new];
tLayer.backgroundColor = getColor(102, 170, 238, 0.2).CGColor;
[tLayer setFrame:(CGRect){marginLeft(sLayer)+marginLeft,self.height/2,radius,radius}];
tLayer.cornerRadius = radius/2;
    
    
self.layer_list = [NSArray arrayWithObjects:fLayer,sLayer,tLayer,nil];
[self.layer addSublayer:fLayer];
[self.layer addSublayer:sLayer];
[self.layer addSublayer:tLayer];
```
球画好了我们就要开始研究它的运动轨迹了。通过一些观察我们分析得出了下面的运动轨迹:
![第一课球的运动动画.gif](http://upload-images.jianshu.io/upload_images/1120896-abd93f18d21599cf.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![第二颗球运动动画.gif](http://upload-images.jianshu.io/upload_images/1120896-8cd887b449a4c6bd.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![第三颗球运动动画.gif](http://upload-images.jianshu.io/upload_images/1120896-5ac742af64652897.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过观察，第一颗球的运动轨迹其实就是一个上半圆加上一个下半圆的路径动画。那么只需要画两个半圆就搞定的事咯？我们来试一下

绘制圆的参考图:
![弧线参考图.png](http://upload-images.jianshu.io/upload_images/1120896-14d1c6d4de788342.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一个半圆是以两个圆的中心点为圆心绘制的半圆,
`第一个半圆是 π -> 0° 顺时针运动,然后π -> 0°逆时针运动`
![第一颗球的运动轨迹.png](http://upload-images.jianshu.io/upload_images/1120896-0f3102e3e05981ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
CAShapeLayer *fLayer = self.layer_list[0];
CAShapeLayer *testLayer = [CAShapeLayer new];
testLayer.fillColor = [UIColor clearColor].CGColor;
testLayer.borderWidth = 1.0f;
testLayer.strokeColor = hexStrColor(@"#AA8C69").CGColor;

UIBezierPath *semicirclePath = [UIBezierPath bezierPath];
//上半圆
[semicirclePath addArcWithCenter:(CGPoint){fLayer.frame.origin.x+25,fLayer.frame.origin.y+10}
                           radius:15
                       startAngle:M_PI
                         endAngle:0
                        clockwise:YES];
//下半圆
[semicirclePath addArcWithCenter:(CGPoint){sLayer.frame.origin.x+25,sLayer.frame.origin.y+10}
                           radius:15
                       startAngle:M_PI
                         endAngle:0
                        clockwise:NO];
testLayer.path = semicirclePathA.CGPath;
[self.layer addSublayer:testLayer];
```


第一颗球的轨迹我们清楚之后我们来设置第二颗球的轨迹:
`0° -> π 顺时针运动`
![第二颗球的运动轨迹.png](http://upload-images.jianshu.io/upload_images/1120896-d2920032746f5893.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
    CAShapeLayer *fLayer = self.layer_list[0];
    CAShapeLayer *sLayer = self.layer_list[1];
    //第二段动画
    CAShapeLayer *testLeftLayer = [CAShapeLayer new];
    testLeftLayer.fillColor = [UIColor clearColor].CGColor;
    testLeftLayer.borderWidth = 1.f;
    testLeftLayer.strokeColor = [UIColor redColor].CGColor;
    UIBezierPath *leftPath = [UIBezierPath bezierPath];
    [leftPath addArcWithCenter:(CGPoint){fLayer.frame.origin.x+25,fLayer.frame.origin.y+10} 
                        radius:15 startAngle:0 endAngle:M_PI clockwise:YES];
    testLeftLayer.path = leftPath.CGPath;
    [self.layer addSublayer:testLeftLayer];
```

我们再来看看第三颗球:
`0° -> π 逆时针运动`
![第三颗球的运动轨迹.png](http://upload-images.jianshu.io/upload_images/1120896-a9dcde2912e87468.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
    CAShapeLayer *sLayer = self.layer_list[1];
    CAShapeLayer *tLayer = self.layer_list[2];
    //第三段动画
    CAShapeLayer *testRightLayer = [CAShapeLayer new];
    testRightLayer.fillColor = [UIColor clearColor].CGColor;
    testRightLayer.borderWidth = duration;
    testRightLayer.strokeColor = [UIColor blueColor].CGColor;
    
    UIBezierPath *rightPath = [UIBezierPath bezierPath];
    [rightPath addArcWithCenter:(CGPoint){sLayer.frame.origin.x+25,tLayer.frame.origin.y+10} 
                         radius:15 startAngle:0 endAngle:M_PI clockwise:NO];
    testRightLayer.path = rightPath.CGPath;
    [self.layer addSublayer:testRightLayer];
```


最后的静态效果图如下:
![静态效果图.png](http://upload-images.jianshu.io/upload_images/1120896-74f9042f3aca98ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设定好轨迹之后我们就来设置它们的动画效果,动画效果就要用到我们上一篇中提到的关键帧动画了.
由于三个动画具有相同的属性设置，所以我们最好是封装一下,提取出它们会有变化的数值。
例如，动画的依赖视图会变，它们的运动path会变,所以提取出这两个参数。
```
- (void)keyFrameAnimationWithLayer:(CALayer *)layer path:(UIBezierPath *)path;
```

```
    CAKeyframeAnimation *keyAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    keyAnimation.path = path.CGPath;
    keyAnimation.fillMode = kCAFillModeForwards;
    keyAnimation.calculationMode = kCAAnimationPaced;
    keyAnimation.removedOnCompletion = NO;
    keyAnimation.duration = duration;
    keyAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    keyAnimation.rotationMode = kCAAnimationRotateAuto;
    keyAnimation.repeatCount = MAXFLOAT;
    keyAnimation.calculationMode = kCAAnimationCubic;
    [layer addAnimation:keyAnimation forKey:@"keyFrameAnimation"];
```
具体的参数信息已经在上一篇中提过了，这里就不说了。
然后我们开始调用:
```
//调用第一颗球
[self keyFrameAnimationWithLayer:fLayer path:semicirclePath];
```

![第一颗球卡顿的动画.gif](http://upload-images.jianshu.io/upload_images/1120896-b70717a25600a727.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们运行之后会发现在两个圆的交界处，卡顿了一下下（关于这个问题，我尝试调整了动画的timingFunction属性还有calculationMode都无法很好的解决这个问题,我猜想是由于两个路径是分开绘制的原因，具体的原因还在研究当中。）
不过我还是找到了解决办法:
设置两个路径，然后将路径进行合并成一个路径:
```
    UIBezierPath *semicirclePathA = [UIBezierPath bezierPath];
    [semicirclePathA addArcWithCenter:(CGPoint){fLayer.frame.origin.x+25,fLayer.frame.origin.y+10} radius:15 startAngle:M_PI endAngle:0 clockwise:YES];
    UIBezierPath *semicirclePathB = [UIBezierPath bezierPath];
    [semicirclePathB addArcWithCenter:(CGPoint){sLayer.frame.origin.x+25,sLayer.frame.origin.y+10} radius:15 startAngle:M_PI endAngle:0 clockwise:NO];
    [semicirclePathA appendPath:semicirclePathB];
    testLayer.path = semicirclePathA.CGPath;
    [self.layer addSublayer:testLayer];
```
运行之后,可以看到效果，如巧克力般丝滑的感觉:
![第一课球的运动动画.gif](http://upload-images.jianshu.io/upload_images/1120896-abd93f18d21599cf.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

画好第一个球之后,我们再来画其他的两个球，基本就相同了,只要调用
```
- (void)keyFrameAnimationWithLayer:(CALayer *)layer path:(UIBezierPath *)path;
```
方法就可以了。

效果如下:
![三颗球运动动画.gif](http://upload-images.jianshu.io/upload_images/1120896-86df865f35635ee7.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是实现的效果并不如意，它们的颜色变化并不是渐变的，看起来并不好，所以我们还需要添加颜色变换的动画
观察原来的效果总结如下:
1. 第一颗球从开始到转到最后的时候，颜色从深色变为了浅色。所以我们可以设定它的变化值为 1 -> 0.2.
2. 第二颗球原本是浅色然后旋转到第一的位置时是深色。所以我们可以设定它的变化值为 0.5 -> 1.
3. 第三颗球原本是最浅的然后旋转到第二的位置时,颜色变深了一点。所以我们设定它的变化值 0.2 -> 0.5.

动画因为只是两个值得变化 , FromValue -> toValue , 我们采用CABasicAnimation就可以轻松实现效果.
我们同样来封装该动画:
```
- (void)colorGradientAnimationWithLayer:(CALayer *)layer fromValue:(id)fromValue toValue:(id)toValue{
    CABasicAnimation *alphaAnimation = [CABasicAnimation animationWithKeyPath:@"backgroundColor"];
    alphaAnimation.fromValue = fromValue;
    alphaAnimation.toValue = toValue;
    alphaAnimation.duration = duration;
    alphaAnimation.repeatCount = MAXFLOAT;
    [layer addAnimation:alphaAnimation forKey:@"anmiationAlpha"];
}
```

调用:
```
//第一颗球的变化
[self colorGradientAnimationWithLayer:fLayer
                                    fromValue:(__bridge id _Nullable)(getColor(102, 170, 238, 1)).CGColor
                                      toValue:(__bridge id _Nullable)(getColor(102, 170, 238, 0.2)).CGColor];
//第二颗球的变化
[self colorGradientAnimationWithLayer:sLayer
                                    fromValue:(__bridge id _Nullable)(getColor(102, 170, 238, 0.5)).CGColor
                                      toValue:(__bridge id _Nullable)(getColor(102, 170, 238, 0.1)).CGColor];
//第三颗球的变化
[self colorGradientAnimationWithLayer:tLayer
                                fromValue:(__bridge id _Nullable)(getColor(102, 170, 238, 0.2)).CGColor
                                  toValue:(__bridge id _Nullable)(getColor(102, 170, 238, 0.5)).CGColor];

```

####在制作过程中出现的情况:
1.动画闪烁
keyAnimation.fillMode = kCAFillModeForwards; 可以添加该属性,该属性表示保留动画结束时的效果。
但添加之后发现还是不行,最后发现其实动画结束的坐标点有问题。

>__bridge 主要是因为我们在编写程序的时候还会用到 CoreFoundation(CF) 框架的对象,CF和 OC 对象之间的类型转换就需要用到__bridge

最后就是我们需要的效果了。
![加载动画.gif](http://upload-images.jianshu.io/upload_images/1120896-f72fff75363d19ee.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

DEMO:
https://github.com/yanggenwei/GWAnimation/tree/master
</small>


