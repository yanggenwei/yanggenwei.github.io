---
layout: post
title: "关键帧动画CAKeyframeAnimation"
date: 2017-12-30
excerpt: "Animation"
tags: [iOS][Animation]
post: true
comments: true
---
<small>
贝塞尔曲线的基础我们了解了，接下来，我们开始自己做一些动画效果。
那么配合贝塞尔曲线的一般使用关键帧动画。为啥呢？因为我喜欢啊,哈哈哈哈！

>我们之前使用过CATransition,CATransition直接继承于CAAnimation。

![动画关系图.png](http://upload-images.jianshu.io/upload_images/1120896-f73c98bd6d124c6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

CAAnimation 简单来说是从一个值到另一个值得变化。（例如我们设置某个视图的颜色为红色,然后在动画中设置结果为绿色,那么就会显示这两种值得变化）

而我们今天用的动画CAKeyframeAnimation继承于CAPropertyAnimation，CAPropertyAnimation虽然也继承于CAAnimation,但是CAPropertyAnimation在CAAnimation基础上增加了一种路径属性的设置。我们不再仅限于改变两个值的变化，而是路径中不同坐标点的变化。

接下来我们就可以了解一下: CAKeyframeAnimation
附加官网文档地址:
https://developer.apple.com/documentation/quartzcore/cakeyframeanimation?language=objc

```
@property(nullable, copy) NSArray *values;
这个属性我们在之前的心心动画中已经运用到多了。简单来说就是:
@[@1,@3,@4,@5] 我们使用CAAnimation可以会设置属性值@1和@3之间的变化，而我们利用values就可以
逐个的从@1依次到@5产生变化。

这个属性是只有path属性为nil的时候才有效果
```
我们简单的使用代码实现一下:
```swift
CAKeyframeAnimation *keyAnimation = [CAKeyframeAnimation animationWithKeyPath:@"transform.scale"];
keyAnimation.values = @[@1,@2,@3,@4,@3,@2,@1];
keyAnimation.duration = 3.0f;
[self.label.layer addAnimation:keyAnimation forKey:@"transform.scale"];

```
![values Test.gif](http://upload-images.jianshu.io/upload_images/1120896-62158727514bbd52.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到,我们设置了视图的transform.scale 即缩放属性。然后设置它值得变化是@1,@2,@3,@4,@3,@2,@1，也就是从小到大，然后从大到小。


>@property(nullable) CGPathRef path;
这个属性就是我们设置了path之后，视图会跟着路径去移动。

我们做个简单的路径动画:
![path.gif](http://upload-images.jianshu.io/upload_images/1120896-2e0991a0417f3107.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

先绘制一个Z字型路径:
```
_bezier = [UIBezierPath bezierPath];
[_bezier moveToPoint:(CGPoint){0,100}];
[_bezier addLineToPoint:(CGPoint){kSCREENWIDTH,100}];
[_bezier addLineToPoint:(CGPoint){0,200}];
[_bezier addLineToPoint:(CGPoint){kSCREENWIDTH,200}];
[_bezier closePath];
[_bezier setLineWidth:0.2];
```

然后将绘制好的路径设置到动画当中
```
//路径动画
CAKeyframeAnimation *keyAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
keyAnimation.duration = 6.0f;
keyAnimation.path = _bezier.CGPath;
[self.label.layer addAnimation:keyAnimation forKey:@"position"];
```


>@property(nullable, copy) NSArray<NSNumber *> *keyTimes;

官网中这样写道:
https://developer.apple.com/documentation/quartzcore/cakeyframeanimation/1412522-keytimes?language=objc

>Each value in the array is a floating point number between 0.0 and 1.0 that defines the time point (specified as a fraction of the animation’s total duration) at which to apply the corresponding keyframe value. 
数组中的每个值都是介于0.0和1.0之间的浮点数，它定义了时间点(指定为动画总长度的一小部分)，用于应用对应的关键帧值。

>Each successive value in the array must be greater than, or equal to, the previous value.
数组中的每一个连续值都必须大于或等于前面的值

该属性是一个数组，每个值分别对应每个子路径(AB,BC,CD,DE)的时间。
如果你没有显式地对keyTimes进行设置，则系统会默认每条子路径的时间为：ti=duration/(帧数)，即每条子路径的duration相等  

我们在上述的动画中加入该属性之后，观看一下效果:
![keyTimes.gif](http://upload-images.jianshu.io/upload_images/1120896-7b589157df01194a.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//路径动画
CAKeyframeAnimation *keyAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
keyAnimation.path = _bezier.CGPath;
keyAnimation.keyTimes = @[@(0.0f),@(0.1f),@(0.2f),@(0.3f),@(0.4f)];
keyAnimation.duration = 6.0f;
[self.label.layer addAnimation:keyAnimation forKey:@"position"];
```
可以看到我们在设置了对应的速度之后,它们的时候速度明显变快了.


>@property(copy) NSString *calculationMode;
计算模式在关键帧动画中也非常重要,效果比较类似于下面说到的timingFunctions。

它一般可以配合keyTimes属性进行设置。在官网介绍keyTimes的时候有这样一段话:
>The appropriate values to include in the array are dependent on the calculationMode property.
在数组中包含的适当值依赖于计算模式属性。

> If the calculationMode is set to kCAAnimationLinear or kCAAnimationCubic, 
the first value in the array must be 0.0 and the last value must be 1.0. 
All intermediate values represent time points between the start and end times.
如果将计算模式设置为kCAAnimationLinear或kCAAnimationCubic，那么数组中的第一个值必须是0.0，
最后一个值必须是1.0。所有中间值都表示开始和结束时间之间的时间点。

>If the calculationMode is set to kCAAnimationDiscrete, 
the first value in the array must be 0.0 and the last value must be 1.0. 
The array should have one more entry than appears in the values array. For example, 
if there are two values, there should be three key times.
如果将计算模式设置为kCAAnimationDiscrete，那么数组中的第一个值必须是0.0，最后一个值必须是1.0。
该数组应该有一个比在值数组中显示的还要多的条目。例如，如果有两个值，则应该有三个关键时刻。


> If the calculationMode is set to kCAAnimationPaced or kCAAnimationCubicPaced, the values in this property are ignored.
如果将计算模式设置为kCAAnimationPaced或kCAAnimationCubicPaced，那么该属性的值就会被忽略。

>If the values in this array are invalid or inappropriate for the current calculation mode, they are ignored.
如果该数组中的值无效或不适合当前计算模式，则会忽略它们。

它的值如下所示:
```
默认值,表示当关键帧为坐标点的时候,关键帧之间线性运动; 
CA_EXTERN NSString * const kCAAnimationLinear
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

所有关键帧直接逐个运动; 
CA_EXTERN NSString * const kCAAnimationDiscrete
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

使得动画均匀进行,而不是按keyTimes设置的或者按关键帧平分时间,此时keyTimes和timingFunctions无效; 
CA_EXTERN NSString * const kCAAnimationPaced
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

对关键帧为坐标点的关键帧进行圆滑曲线相连后插值计算,对于曲线的形状还可以通过tensionValues,continuityValues,biasValues来进行调整自定义,
这里的主要目的是使得运行的轨迹变得圆滑; 
CA_EXTERN NSString * const kCAAnimationCubic
    CA_AVAILABLE_STARTING (10.7, 4.0, 9.0, 2.0);

在kCAAnimationCubic的基础上使得动画运行变得均匀,
就是系统时间内运动的距离相同,此时keyTimes以及timingFunctions也是无效的.
CA_EXTERN NSString * const kCAAnimationCubicPaced
    CA_AVAILABLE_STARTING (10.7, 4.0, 9.0, 2.0);
```


>@property(nullable, copy) NSArray<CAMediaTimingFunction *> *timingFunctions;
设置它可以影响动画的阶段速度。

```
//线性速度,各阶段保持匀速
CA_EXTERN NSString * const kCAMediaTimingFunctionLinear 
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

//淡入 也就是开头会慢一点
CA_EXTERN NSString * const kCAMediaTimingFunctionEaseIn
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

//淡出 结束的时候会慢
CA_EXTERN NSString * const kCAMediaTimingFunctionEaseOut
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

//淡入淡出 开头和结束的时候会慢
CA_EXTERN NSString * const kCAMediaTimingFunctionEaseInEaseOut
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

//默认
CA_EXTERN NSString * const kCAMediaTimingFunctionDefault
    CA_AVAILABLE_STARTING (10.6, 3.0, 9.0, 2.0);

```
下面的三个值都是配合 calculationMode 的 kCAAnimationCubic值进行使用的。用来调整自定义的曲线
我们可以通过官网的解释来了解一下，有空的话我也会做一些demo来直观了解一下。
>@property(nullable, copy) NSArray<NSNumber *> *tensionValues;
一组NSNumber对象定义了曲线的紧密性。

>This property is used only for the cubic calculation modes. Positive values indicate a tighter curve while negative values indicate a rounder curve. The first value defines the behavior of the tangent to the first control point, the second value controls the second point’s tangents, and so on. If you do not specify a value for a given control point, the value 0 is used.
该属性仅用于立方计算模式。正值表示一条更紧的曲线，而负值则表示一条圆曲线。第一个值定义了第一个控制点的切线的行为，第二个值控制了第二个点的切线，以此类推。如果不指定给定控制点的值，则使用值0。


>@property(nullable, copy) NSArray<NSNumber *> *continuityValues;
一组NSNumber对象，它定义了时间曲线的锐度。

>This property is used only for the cubic calculation modes. Positive values result in sharper corners while negative values create inverted corners. The first value defines the behavior of the tangent to the first control point, the second value controls the second point’s tangents, and so on. If you do not specify a value for a given control point, the value 0 is used.
该属性仅用于cubic calculation modes。正值会导致更尖锐的角落，而负值则会产生倒角。第一个值定义了第一个控制点的切线的行为，第二个值控制了第二个点的切线，以此类推。如果不指定给定控制点的值，则使用值0。

>@property(nullable, copy) NSArray<NSNumber *> *biasValues;
一组NSNumber对象，它定义了相对于控制点的曲线的位置。

>This property is used only for the cubic calculation modes. Positive values move the curve before the control point while negative values move it after the control point. The first value defines the behavior of the tangent to the first control point, the second value controls the second point’s tangents, and so on. If you do not specify a value for a given control point, the value 0 is used.
该属性仅用于cubic calculation modes。正值在控制点前移动曲线，而负值则在控制点之后移动。第一个值定义了第一个控制点的切线的行为，第二个值控制了第二个点的切线，以此类推。如果不指定给定控制点的值，则使用值0。

```
@property(nullable, copy) NSString *rotationMode;
旋转样式

根据路径自动旋转
CA_EXTERN NSString * const kCAAnimationRotateAuto
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);

根据路径自动翻转
CA_EXTERN NSString * const kCAAnimationRotateAutoReverse
    CA_AVAILABLE_STARTING (10.5, 2.0, 9.0, 2.0);
```
好吧，图片总是上传失败，简单来说就是，图像根据路径移动的时候是会调整自身的方向的，不那么刻板的一直是摆在正中间那样的效果。

</small>


