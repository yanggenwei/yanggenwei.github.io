---
layout: post
title:  "iOS中的粒子动画"
date:   2017-12-28
categories: 动画
---
<small>
>CAEmitterLayer
释放、动画和呈现粒子系统。

它是比较完善的粒子渲染引擎了。所以我们在开始的时候需要去熟悉的它的属性，因为我们只需要设置它的属性就可以做到我们所想要的效果。

```swift
粒子发射器的形状
@property(copy) NSString *emitterShape

以下值:
CA_EXTERN NSString * const kCAEmitterLayerPoint
    CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerLine
     CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerRectangle
     CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerCuboid
     CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerCircle 
     CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerSphere
     CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
```

```
发射模式，这个字段规定了在特定形状上发射的具体形式是什么
@property(copy) NSString *emitterMode;

CA_EXTERN NSString * const kCAEmitterLayerPoints
    CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerOutline
    CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerSurface
    CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
CA_EXTERN NSString * const kCAEmitterLayerVolume
    CA_AVAILABLE_STARTING (10.6, 5.0, 9.0, 2.0);
```

具体的参数效果，看这篇文章，本来录了一些，好吧。比较懒，而且对于做重复的事情没啥意义。
https://www.cnblogs.com/densefog/p/5424155.html

属性参数:

alphaRange | 一个粒子的颜色alpha能改变的范围
---|---
alphaSpeed|粒子透明度在生命周期内的改变速度
birthrate	|每秒发射的粒子数量
blueRange	|一个粒子的颜色blue 能改变的范围
blueSpeed|	粒子blue在生命周期内的改变速度
color	|粒子的颜色
contents	|是个CGImageRef的对象,既粒子要展现的图片
contentsRect	|应该画在contents里的子rectangle
emissionLatitude	|发射的z轴方向的角度
emissionLongitude|	x-y平面的发射方向
emissionRange	|周围发射角度
emitterCells|	粒子发射的粒子的数组
enabled	|粒子是否被渲染
greenrange	|一个粒子的颜色green 能改变的范围
greenSpeed	|粒子green在生命周期内的改变速度
lifetime|	生命周期
lifetimeRange	|生命周期范围 lifetime= lifetime(+/-) lifetimeRange
magnificationFilter|	增加自己的大小
minificatonFilter|	减小自己的大小
minificationFilterBias|	减小大小的因子
name	|粒子的名字
redRange	|一个粒子的颜色red 能改变的范围
redSpeed	|粒子red在生命周期内的改变速度
scale|	缩放比例
scaleRange|	缩放比例范围
scaleSpeed|	缩放比例速度
spin	|子旋转角度
spinrange	|子旋转角度范围
velocity	|速度
velocityRange|	速度范围
xAcceleration|粒子x方向的加速度分量
yAcceleration	|粒子y方向的加速度分量
zAcceleration|	粒子z方向的加速度分量

所以我们大概了解一下，然后我们以实践为准。
我们先看今天实现的这个效果,我一直想做一下这个，也比较简单。
![like.gif](http://upload-images.jianshu.io/upload_images/1120896-c8a873666382b798.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

主要包含了两种，一种利用CAKeyframeAnimation 关键帧动画还有CAEmitterLayer做出来的粒子动画。

###配置粒子动画参数
```
@property (nonatomic,readwrite,strong)CAEmitterLayer *emitterLayer;
```

```
- (void)configerEmitterLayer{
    self.emitterLayer = [[CAEmitterLayer alloc] init];
    //发射的位置
    [self.emitterLayer setEmitterPosition:self.likeButton.center];
    //发射源的尺寸大小
    self.emitterLayer.emitterSize  = (CGSize){80,80};
    //发射源的形状
    self.emitterLayer.emitterShape = kCAEmitterLayerCircle;
    //发射源的模式
    self.emitterLayer.emitterMode = kCAEmitterLayerOutline;
    CAEmitterCell *cell = [CAEmitterCell new];
    //cell的内容,即可以设置图片，也可以自己绘制图形转换成图片
    cell.contents = (id)[PMBTools getBuddleImage:@"start" imageType:@"png"].CGImage;
    //每秒中粒子的数量
    cell.birthRate = 0.f;
    //粒子从出生到结束的时间
    cell.lifetime = 0.3f;
    //粒子生存浮动的时间, [lifetime - lifetimeRange]
    cell.lifetimeRange = 0.5f;
    //运动的速度
    cell.velocity = 30.f;
    //运动浮动速度[velocity - velocityRange]
    cell.velocityRange = 4.f;
    //Y方向的加速度 值越大每个粒子下落的速度越快
    cell.yAcceleration = 15.f;
    
    //粒子发射出来的角度
    cell.emissionLongitude = 0;
    //粒子发射出来的浮动角度,这个角度决定了粒子能够在多大角度范围内扩散
    cell.emissionRange = 180;
    
    //粒子的初始大小
    cell.scale = 0.003;
    //粒子的初始大小的浮动范围 [scal - scaleRange]
    cell.scaleRange = 0.06;
    //粒子缩放的速度
   cell.scaleSpeed = 0.01;
    
//    //粒子的颜色，这里cell可以对图片进行新颜色的填充
//    cell.color = getColor(1.f, 1.f, 1.f, 1).CGColor;
//    cell.redRange = 1.0f;
//    cell.greenRange = 1.0f;
//    cell.blueRange = 1.0f;
//    //粒子透明的范围
    cell.alphaRange = 0.8f;
//    //粒子透明的速度
    cell.alphaSpeed = -0.1f;
    
    [self.emitterLayer setEmitterCells:@[cell]];
    [self.view.layer addSublayer:self.emitterLayer];
}
```


```
关键的参数:
self.emitterLayer.emitterShape = kCAEmitterLayerCircle;
我们想要的效果是点击的时候呈现圆形扩散,
kCAEmitterLayerCircle的效果就是呈现圆形扩散，所以比较符合我们的设定

self.emitterLayer.emitterMode = kCAEmitterLayerOutline;
这个模式下整个边框都是发射点，即边框进行发射,而我们上边设置的是圆形，所以边框呈现圆形范围。

然后就是我们需要它在开始的时候没有小星星,所以设置它的
cell.birthRate = 0.f; 不设置的话默认为0。

其它的你们怎么喜欢怎么来咯。
```

然后就是那个心心点击的时候的动画:
![心心动画.gif](http://upload-images.jianshu.io/upload_images/1120896-b711ee4c3cfdd9c6.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

通过观察其实就是这个图片一个会一会小了，我们可以自己设置一下它的放大比例和缩小比例。
我们实现这个动画的方式很多，但是关键帧动画相对来说比较简单(关键帧动画会单独写一篇，这里就简单看下实践吧)：
```
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"transform.scale"];
    animation.values = @[@1.4,@0.7,@1.0,@1.3,@1.0];
    animation.duration = 0.5;
    animation.calculationMode = kCAAnimationCubic;
    [self.likeButton.layer addAnimation:animation forKey:@"transform.scale"];
```

```
    kCAAnimationCubic 对关键帧为座标点的关键帧进行圆滑曲线相连后插值计算

    animationWithKeyPath的值：
　  transform.scale = 比例轉換
    transform.scale.x = 闊的比例轉換
    transform.scale.y = 高的比例轉換
    transform.rotation.z = 平面圖的旋轉
    opacity = 透明度
    margin  四周边距
    zPosition Z轴位置
    backgroundColor    背景颜色
    cornerRadius    圆角
    borderWidth 边框宽度
    bounds  
    contents
    contentsRect
    cornerRadius
    frame 
    hidden 是否隐藏
    mask   
    masksToBounds 是否截取边缘
    opacity  不透明度
    position 位置
    shadowColor 阴影颜色
    shadowOffset 阴影偏移度
    shadowOpacity 阴影不透明度
    shadowRadius 阴影半径
```

嗯，配置也好了，动画也有了，下面就是开启粒子动画了
```
    //首先获取我们粒子动画单元。设置它的粒子数量
    CAEmitterCell *cell = self.emitterLayer.emitterCells[0];
    [cell setBirthRate:30.f];
    //然后再设置粒子layer的粒子数量，如果你只设置其中的一种，那是么有用的。
    //网上有直接通过反射设置的我试过了没有效果。还是说操作的方式不正确。不过被我发现了这个可用的了。
    [self.emitterLayer setBirthRate:30.f];
    //设置粒子动画立马显示
    self.emitterLayer.beginTime = CACurrentMediaTime();
    //想粒子动画消失其实就是将粒子设置为0,都没有粒子了自然也就结束了。
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
          [self.emitterLayer setBirthRate:0.f];
    });
```
嗯。到这里点击心心的动画也就结束了。记得把上面的粒子动画放在上面的关键帧动画里哦。 

DEMO:
https://github.com/yanggenwei/GWAnimation/tree/master
</small>


