---
layout: post
title: "iOS中的转场动画"
date: 2017-12-26
excerpt: "Animation"
tags: [iOS][Animation]
post: true
comments: true
---
<small>
我们常用的动画效果属于CoreAnimation库中,CoreAnimation 虽然翻译过来是核心动画，但其实它不仅仅是动画,动画只是layer层级中的特效操作而已。在图层渲染，图形变动基础上，它提供了很多自由且方便的API和类供我们使用。
关于这方面的描述，比较推荐的就是
> iOS Core Animation: Advanced Techniques

它对于CoreAnimation的描述很详细很底层。如果英文看不懂，还有翻译:
https://zsisme.gitbooks.io/ios-/content/
还有就是
> iOS Animations by Tutorials v4.0

不过它没有上面那本底层。但却是非常实用的一本书。提供了很多常用的动画。(关键这本书贵啊.国外网站售价$59,还是电子书)
下面是它的链接,顺便还提供了其他的书籍,有能力请支持正版:
>链接: https://pan.baidu.com/s/1bOU4ai 密码: hx49

由于系列比较长,所以我们要慢慢来。今天就从简单的过渡动画做起来，因为系统有非常简单的API供我们使用。
最终效果如下:
![动画1.gif](http://upload-images.jianshu.io/upload_images/1120896-c8ad18bf8a493307.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![动画2.gif](http://upload-images.jianshu.io/upload_images/1120896-21b91d55e158b63a.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![动画3.gif](http://upload-images.jianshu.io/upload_images/1120896-0c4fb5b2e6af45e6.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们实现上述效果主要是使用 CATransition 类。
####CATransition是什么呢？
>https://developer.apple.com/documentation/quartzcore/catransition
> An object that provides an animated transition between a layer's states.
一个在层状态之间提供动画转换的对象。

官方解释就很简单直接了.网页中也清楚的写明了使用方法。只是官网写用Swift写的,而我使用Objective-C写咯。 所以我们直接开干。
因为动画比较多,我们一个个去了解的很难去识别是什么动画,所以我们需要制定一个枚举:
```
typedef NS_ENUM(NSInteger,GWTransitionsAnimationType){
    FadeAnimationType = 0,                   //淡入淡出
    PushAnimationType,                       //推挤
    RevealAnimationType,                     //揭开
    MoveInAnimationType,                     //覆盖
    CubeAnimationType,                       //立方体
    SuckEffectAnimationType,                 //吮吸
    OglFlipAnimationType,                    //翻转
    RippleEffectAnimationType,               //波纹
    PageCurlAnimationType,                   //翻页
    PageUnCurlAnimationType,                 //反翻页
    CameraIrisHollowOpenAnimationType,       //开镜头
    CameraIrisHollowCloseAnimationType,      //关镜头
    CurlDownAnimationType,                   //下翻页
    CurlUpAnimationType,                     //上翻页
    FlipFromLeftAnimationType,               //左翻转
    FlipFromRightAnimationType,              //右翻转
};
```

```swift
//创建CATransition对象
    CATransition *animation = [CATransition animation];
    //设置运动时间
    animation.duration = DURATION;
    animation.delegate = self;
    //设置运动type
    animation.type = type;
    if(animation.subtype !=nil){
        animation.subtype = subType;
    }
    //设置运动速度
    animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    [forView.layer addAnimation:animation forKey:@"animation"];

type(动画类型)的定义:
CA_EXTERN NSString * const kCATransitionFade     //淡入
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
CA_EXTERN NSString * const kCATransitionMoveIn   //覆盖
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
CA_EXTERN NSString * const kCATransitionPush     //推入
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
CA_EXTERN NSString * const kCATransitionReveal   //揭开
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

subtype(动画方向)的定义:
CA_EXTERN NSString * const kCATransitionFromRight 
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
CA_EXTERN NSString * const kCATransitionFromLeft
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
CA_EXTERN NSString * const kCATransitionFromTop
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
CA_EXTERN NSString * const kCATransitionFromBottom
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

timingFunction(动画速度类型)的定义：
kCAMediaTimingFunctionLinear  //线性动画
kCAMediaTimingFunctionEaseIn  //先快后慢
kCAMediaTimingFunctionEaseOut //先慢后快
kCAMediaTimingFunctionEaseInEaseOut// 先慢后快再慢
kCAMediaTimingFunctionDefault //默认，也属于中间比较快
```

这段很长，我们不可能拿来直接使用，需要包装一下，那么我就需要暴露出参数。
```
duration(可选): 运动时间
type:动画类型
subType:动画方向
timingFunction(可选):动画速度
view:动画依赖的对象

- (void)animationWithType:(NSString *)type 
                  subType:(NSString *)subType 
                  forView:(UIView *)forView;
```

调用:
```
[self animationWithType:kCATransitionFade subType:subStr forView:self.fView];
```

除了上述的动画之外，还有上下左右的翻页动画需要使用另外一种方式:
```swift 

  [UIView animateWithDuration:DURATION animations:^{
        [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
        [UIView setAnimationTransition:transition forView:view cache:YES];
    }];

参数UIViewAnimationCurveEaseInout 是属于UIViewAnimationCurve枚举:

typedef NS_ENUM(NSInteger, UIViewAnimationCurve) {
    UIViewAnimationCurveEaseInOut,         // 开始和结束时慢
    UIViewAnimationCurveEaseIn,            // 开始时慢
    UIViewAnimationCurveEaseOut,           // 结束时慢
    UIViewAnimationCurveLinear, //线性速度。保持匀速
};

参数transition引用的是一个枚举值UIViewAnimationTransition:

typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {
    UIViewAnimationTransitionNone,     //无动画
    UIViewAnimationTransitionFlipFromLeft, //左翻页
    UIViewAnimationTransitionFlipFromRight, //右翻页
    UIViewAnimationTransitionCurlUp, //上翻页
    UIViewAnimationTransitionCurlDown, //下翻页
};
```

同理，需要包装一下:
``` swift
- (void)animationWithView:(UIView *)view transitionType:(UIViewAnimationTransition) transition;
```
调用:
```swift
[self animationWithView:self.view transitionType:UIViewAnimationTransitionCurlDown];
```

CATransition 还有个代理CAAnimationDelegate
里面有两个方法
```
- (void)animationDidStart:(CAAnimation *)anim{
    NSLog(@"动画已经开始了");
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
    NSLog(@"动画已经结束了");
}
```


DEMO:
https://github.com/yanggenwei/GWAnimation/tree/master
</small>


