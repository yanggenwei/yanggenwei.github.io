---
layout: blog
technology: true 
background: purple
background-image: http://p4wibncwo.bkt.clouddn.com/lum3n-187468-unsplash.jpg
title:  "iOS中的形变动画"
date:   2017-12-27
category: 技术
tags:
- iOS
- Animation
- 动效
- 形变动画
---

<small>
形变动画即形状的旋转,位移,缩放和形状的变化。

官网地址:
>https://developer.apple.com/documentation/coregraphics/cgaffinetransform-rb5?language=objc

要想达到我们所需要的结果，需要使用到CGAffineTransform
CGAffineTransform很多人认为这是个类。抱歉这不是类。官网说的很清楚。
>An affine transformation matrix for use in drawing 2D graphics.
一个用于绘制2D图形的仿射变换矩阵。

是的。一种矩阵.
打开CGAffineTransform.h 文件我们就可以看到的。
```
struct CGAffineTransform {
  CGFloat a, b, c, d;
  CGFloat tx, ty;
};
```
在程序内部我们对其进行变化的时候，其实内部代码改的就是这个矩阵的数据。
具体的矩阵原理 传送门:
https://www.zhihu.com/question/21351965

原理看不懂，我们就来看看源码:
> CGPointApplyAffineTransform
返回一个已存在点的仿射变换所产生的点。

方法定义:
```
CG_EXTERN CGPoint CGPointApplyAffineTransform(CGPoint point,
  CGAffineTransform t) CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);


CG_INLINE CGPoint
__CGPointApplyAffineTransform(CGPoint point, CGAffineTransform t)
{
  CGPoint p;
  p.x = (CGFloat)((double)t.a * point.x + (double)t.c * point.y + t.tx);
  p.y = (CGFloat)((double)t.b * point.x + (double)t.d * point.y + t.ty);
  return p;
}
```

可以看到,参数中的坐标点的计算方式
即ax+cy+tx负责坐标x
即bx+dy+ty负责坐标y


>CGSizeApplyAffineTransform
通过对现有高度和宽度的转换来返回高度和宽度。

```
方法定义
CG_EXTERN CGSize CGSizeApplyAffineTransform(CGSize size, CGAffineTransform t)
  CG_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);


CG_INLINE CGSize
__CGSizeApplyAffineTransform(CGSize size, CGAffineTransform t)
{
  CGSize s;
  s.width = (CGFloat)((double)t.a * size.width + (double)t.c * size.height);
  s.height = (CGFloat)((double)t.b * size.width + (double)t.d * size.height);
  return s;
}
```
aw+ch可以得到宽度width
bw+dh可以得到高度height
这样想的其实也简单多了


要是还不行，那只能好好学学高中的矩阵入门基础了,大学的线性代数。

> CG_INLINE 表示内联,即在编译的时候将函数体替换函数调用，从而不需要将parameter，return address进行push/pop stack的操作，从而加速app的运行，然而，会增加二进制文件的大小。


CGAffineTransform它所定义的类型很多,我们来介绍一下:

>Creating an Affine Transformation Matrix
直接创建一个新的变换矩阵

```
//设置完整的变形矩阵
CGAffineTransformMake
//旋转
CGAffineTransformMakeRotation
//缩放
CGAffineTransformMakeScale
//位移
CGAffineTransformMakeTranslation
```


```
CGAffineTransformMake
它表示一个新的仿射变换矩阵
```

>方法定义:CGAffineTransform CGAffineTransformMake(CGFloat a, CGFloat b, CGFloat c, CGFloat d, CGFloat tx, CGFloat ty);

CGAffineTransformMake 可以设置比较完整的矩阵.通过它就可以直接设置旋转,缩放,位移的操作的矩阵
它的属性设置可以看下图:
![矩阵模型.png](http://upload-images.jianshu.io/upload_images/1120896-11a79ec4ad3dd8d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

![计算.png](http://upload-images.jianshu.io/upload_images/1120896-c9ffe0188ec7398a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

![结果.png](http://upload-images.jianshu.io/upload_images/1120896-a1e4ec4a65c5f4e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)
可以看到我们主要是设置它的6个值,最后算出它的坐标x,y。其中第三列的0,0,1是固定不动的哦。


以上的方法,在调用的时候每次都是初始值。
以下的方法,在调用的时候都会之前已经修改过的基础上进行变换。

>Modifying Affine Transformations
创建一个新的或者在原来基础上修改的变换矩阵。

```
方法:
CGAffineTransformTranslate
```
![translate.gif](http://upload-images.jianshu.io/upload_images/1120896-c770dfbbec853014.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

它的属性设置图:
![translate.png](http://upload-images.jianshu.io/upload_images/1120896-056fb684842fda44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)
根据上面的计算公式,设置tx,ty的值就直接相当于设置它的坐标x,y了。通过设置它的坐标我们就可以很轻松的实现它的位移操作了。

```
CGAffineTransformScale 
用于图形缩放
```

方法定义:
>CGAffineTransform CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy);

![scale.gif](http://upload-images.jianshu.io/upload_images/1120896-d09816b747f46224.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

sx,sy的值等于1为不变,小于1为缩小,大于1为放大

```
CGAffineTransformRotate
用于图形旋转
```
>方法定义
CGAffineTransform CGAffineTransformRotate(CGAffineTransform t, CGFloat angle);

>In iOS, a positive value specifies counterclockwise rotation and a negative value specifies clockwise rotation. In macOS, a positive value specifies clockwise rotation and a negative value specifies counterclockwise rotation.
在iOS中，一个正值指定了逆时针旋转，而负值则指定顺时针旋转。在macOS中，一个正的值指定顺时针旋转，一个负的值指定逆时针旋转。

官网的翻译意思是这样的。然后经过试验,在iOS中 正数是顺时针,负数才是逆时针的。我是不是该提点建议啥的，在线等，蛮急的。
```
case GWAnimationRotatingClockwise://顺时针
            self.showView.transform = CGAffineTransformRotate(self.showView.transform, 90);
            break;
case GWAnimationRotatingAnticlockwise://逆时针
            self.showView.transform = CGAffineTransformRotate(self.showView.transform, -90);
            break;
```
其原理图如下:
![旋转矩阵.gif](http://upload-images.jianshu.io/upload_images/1120896-c053eeb1b6720665.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

![旋转计算.gif](http://upload-images.jianshu.io/upload_images/1120896-d74d1688126769c1.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

填值参考:
 ![弧线参考图.png](http://upload-images.jianshu.io/upload_images/1120896-87655ce1e9d0a14d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![rotate.gif](http://upload-images.jianshu.io/upload_images/1120896-f0d38af10a2308fa.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

```
CGAffineTransformInvert
反转当前图形的坐标
```

>Inversion is generally used to provide reverse transformation of points within transformed objects. Given the coordinates (x,y), which have been transformed by a given matrix to new coordinates (x’,y’), transforming the coordinates (x’,y’) by the inverse matrix produces the original coordinates (x,y).
反转通常用于提供转换对象内的点的反向转换。给定的坐标(x，y)已经被一个给定的矩阵转换为新的坐标(x'，y')，通过逆矩阵变换坐标(x'，y')产生原始坐标(x，y)。

![invert.gif](http://upload-images.jianshu.io/upload_images/1120896-d14e1d1194adf499.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

```
CGAffineTransformConcat
将两个transform组合在一起。
```
>方法定义
CGAffineTransform CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);
返回一个新的transform:  t’ = t1*t2. 

![合并.gif](http://upload-images.jianshu.io/upload_images/1120896-8b64ab6fa47cfe14.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/340)

*注意，矩阵运算不是交换——你连接矩阵的顺序是很重要的。也就是说，乘以矩阵t1乘以矩阵t2的结果并不一定等于乘以矩阵t1乘以矩阵t1。*

> Evaluating Affine Transforms
判断类型
```
CGAffineTransformIsIdentity
检查仿射转换是否为恒等变换。
CGAffineTransformEqualToTransform
检查两个仿射变换是否相等。
```

了解了API之后,我们就开始实践了:
设置枚举值,我们可以方便的使用
```
typedef NS_ENUM(NSInteger,GWDeformationType){
    GWAnimationDisplacementUp,//位移
    GWAnimationDisplacementDown,
    GWAnimationDisplacementLeft,
    GWAnimationDisplacementRight,
    GWAnimationZoomEnlargement,        //缩放
    GWAnimationZoomOut,
    GWAnimationRotatingClockwise,    //旋转
    GWAnimationRotatingAnticlockwise,
    GWAnimationDeformation,  //形变
    GWAnimationInvert,       //反转
    GWAnimationConcat       //合并
};
```

调用API就很简单了:
```
        case GWAnimationDisplacementUp://上移
            self.showView.transform  = CGAffineTransformTranslate(self.showView.transform, 0,-10);
            break;
        case GWAnimationDisplacementDown://下移
            self.showView.transform  = CGAffineTransformTranslate(self.showView.transform, 0,10);
            break;
        case GWAnimationDisplacementLeft://左移
            self.showView.transform  = CGAffineTransformTranslate(self.showView.transform, -10,0);
            break;
        case GWAnimationDisplacementRight://右移
            self.showView.transform  = CGAffineTransformTranslate(self.showView.transform, 10,0);
            break;
        case GWAnimationZoomEnlargement: //放大
            self.showView.transform = CGAffineTransformScale(self.showView.transform,1.5,1.5);
            break;
        case GWAnimationZoomOut://缩小
            self.showView.transform = CGAffineTransformScale(self.showView.transform,0.5,0.5);
            break;
        case GWAnimationRotatingClockwise://顺时针
            self.showView.transform = CGAffineTransformRotate(self.showView.transform, 90);
            break;
        case GWAnimationRotatingAnticlockwise://逆时针
            self.showView.transform = CGAffineTransformRotate(self.showView.transform, -90);
            break;
```

单独的使用并无什么乐趣，我们可以做个简单的动画如下图所示:
![形变.gif](http://upload-images.jianshu.io/upload_images/1120896-645466528185f9e9.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

```
[UIView animateWithDuration:1 animations:^{
                self.showView.transform = CGAffineTransformMakeScale(1.5,1.5);
     } completion:^(BOOL finished) {
                [UIView animateWithDuration:1 animations:^{
                    self.showView.transform = CGAffineTransformScale(self.showView.transform,0.5,0.5);
                    self.showView.transform = CGAffineTransformRotate(self.showView.transform, 180);
                } completion:^(BOOL finished) {
                    [UIView animateWithDuration:1 animations:^{
                        self.showView.transform = CGAffineTransformScale(self.showView.transform,2,2);
                        self.showView.transform = CGAffineTransformRotate(self.showView.transform, -180);
                    } completion:^(BOOL finished) {
                    }];
                }];
            }];
```

DEMO:
https://github.com/yanggenwei/GWAnimation/tree/master

</small>


