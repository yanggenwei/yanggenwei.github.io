---
layout: post
title: "利用WKWebview实现大图浏览"
date: 2017-06-23
excerpt: "WKWebView"
tags: [iOS]
post: true
comments: true
---
<small>
本章内容:
```
1. 为WKWebview添加进度条
2. 实现一个图片浏览器,并且可以保存本地
3. 实现打开网页,获取所有可用图片
```
![网页加载.gif](http://upload-images.jianshu.io/upload_images/1120896-7423632db8f296f7.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/200)

![最终效果.gif](http://upload-images.jianshu.io/upload_images/1120896-1850881c521f1cf0.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/200)

## 为WKWebview添加进度条
  这个其实比较简单：
```
1. 将UIProgressView添加到WKWebview中
2. 监听WKWebview的estimatedProgress属性值变化
3. 将estimatedProgress的数据同步到progressView的progress值
4. 移除监听 
5. 自定义一点特效啥的。这个大家怎么开心怎么来
```

```
第一步:
初始化UIProgressView
- (UIProgressView *)progressView{
    if(!_progressView){
        _progressView = [[UIProgressView alloc] initWithFrame:(CGRect){0,0,kSCREENWIDTH,0.5}];
        CGAffineTransform transform = CGAffineTransformMakeScale(0.5f, 0.5f);
        _progressView.transform = transform;//设定宽高 
       //设置进度条视图的背景色
        [_progressView setTrackTintColor:hexStrColor(@"#FAFAFA")];
    }return _progressView;
}

第二步:
[self addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:nil];

第三步:
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary<NSKeyValueChangeKey,id> *)change
                       context:(void *)context{
    
    if ([keyPath isEqualToString:@"estimatedProgress"]) {
       //设置进度条视图的值等于网页加载进度
        self.progressView.progress = self.estimatedProgress;
        if (self.progressView.progress == 1) {
            //如果进度条完成了,设置动画淡出效果
            [UIView animateWithDuration:0.25f delay:0.3f options:UIViewAnimationOptionCurveEaseOut animations:^{
                self.progressView.transform = CGAffineTransformMakeScale(1.0f, 0.5f);
            } completion:^(BOOL finished) {
                [UIView animateWithDuration:0.2 animations:^{
                    self.progressView.alpha = 0; 
                }];
            }];
        }
    }
}

第四步:移除监听
- (void)dealloc {
    [self removeObserver:self forKeyPath:@"estimatedProgress"];
}
```
这里要说的就是为什么要用到transform形变.因为你直接在frame中设置它的宽，对于它来说没有影响，它是有默认值的。
那么通过transform形变的设置,我们就可以设置它的高度了。

##实现图片浏览器
我们先看一下我们最终的效果
![图片浏览器.gif](http://upload-images.jianshu.io/upload_images/1120896-49bfdd7e7bd88b3a.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

####分析
整个图片浏览器的组成是:
![层次.png](http://upload-images.jianshu.io/upload_images/1120896-652ed97354a4357e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/200)
从图中就可以简单看出来了:
```
UIScrollView+若干UIImageView 形成了图片的浏览
UIView+UIButton形成了底层按钮的返回和保存按钮。
```
视图拼凑起来很简单,所以我们就来分析它的功能:
#####对外方法的实现
一切都要根据具体的工作需求来定，而我目前的需求是 得到网页中的所有可用图片,并且显示当前点击的图片。
所以,我们需要导入所有的图片地址,和当前点击的图片地址。
######为什么是图片地址而不是加载好所有图片传过来呢？
因为,在实际使用中,流量可是很值钱的啊，你一下全给加载好了，很多人家又不需要看的，你噼里啪啦的全给搞好了，这就有点尴尬了。
所以最终的结果就是这样的:
```

/**
 快速构建图片浏览器

 @param images 图片列表（储存图片下载地址）
 @param selectedImagePath 图片地址
 @return 
 */
+ (instancetype)browseImages:(NSArray *)images
           selectedImagePath:(NSString *)selectedImagePath;
```
而为什么选中的图片传的是路径却不是坐标尼,因为我哪知道你点的是哪个,除非你网页中设置的好好的。但是我自己实现比较麻烦尼，所以我还缺个前端高手，最好是个萌妹子尼（请祝福我回去不要跪榴莲）。

#####UIImageView图片如何自适应
关于这个问题,说白就是多读书,多看源码,少玩游戏,少睡觉。因为这玩意完全就是属性值啊。
```
//适配屏幕
[imageView setContentScaleFactor:[[UIScreen mainScreen] scale]];
```
```
//设置图片的显示模式 
 imageView.contentMode =  UIViewContentModeScaleAspectFit;

下面附送的是具体的模式枚举
typedef NS_ENUM(NSInteger, UIViewContentMode) {
//图片拉伸填充至整个UIImageView(图片可能会变形)默认的属性
    UIViewContentModeScaleToFill,
//图片拉伸至完全显示在UIImageView(图片不会变形)
    UIViewContentModeScaleAspectFit,
//图片拉伸至图片的的宽度或者高度等于UIImageView的宽度或者高度为止.看图片的宽高哪一边最接近UIImageView的宽高,一个属性相等后另一个就停止拉伸.
    UIViewContentModeScaleAspectFill,
//调用setNeedsDisplay 方法时,就会重新渲染图片
//下面的属性都是不会拉伸图片的
    UIViewContentModeRedraw,
//中间模式
    UIViewContentModeCenter, 
//顶部
    UIViewContentModeTop,
//底部
    UIViewContentModeBottom,
//左边
    UIViewContentModeLeft,
//右边
    UIViewContentModeRight,
//左上
    UIViewContentModeTopLeft,
//右上
    UIViewContentModeTopRight,
//左下
    UIViewContentModeBottomLeft,
//右下
    UIViewContentModeBottomRight,
};
```
```
imageView.autoresizingMask = UIViewAutoresizingFlexibleHeight;
autoresizingMask的属性，它对应的是一个枚举的值（如下），属性的意思就是自动调整子控件与父控件中间的位置，宽高。
enum {
  //就是不自动调整
   UIViewAutoresizingNone                 = 0,
  //自动调整与superView左边的距离，保证与superView右边的距离不变。
   UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
   //自动调整自己的宽度，保证与superView左边和右边的距离不变
   UIViewAutoresizingFlexibleWidth        = 1 << 1,
   //自动调整与superView左边的距离，保证与左边的距离和右边的距离和原来距左边和右边的距离的比例不变。
   UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
   UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
   //自动调整自己的高度，保证与superView顶部和底部的距离不变
   UIViewAutoresizingFlexibleHeight       = 1 << 4,
   //自动调整与superView底部的距离，也就是说，与superView顶部的距离不变。
   UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```
```
裁剪边缘什么的就比较常见.主要用于一些奇形怪状的图片.
imageView.clipsToBounds  = YES;
```
#####

#####如何实现图片的下载
```
这个就比较简单的通过异步加载,然后设置图片的时候别忘记将其放在主线程中。不然会导致加载延迟。毕竟异步线程它比较任性吧。
- (void)downloadImageWithPath:(NSString *)imagePath
                    imageView:(UIImageView *)imageView{
    [SVProgressHUD show];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"downloadUrl:%@",imagePath);
        NSURL *downloadUrl = [NSURL URLWithString:imagePath];
        NSData *data = [NSData dataWithContentsOfURL:downloadUrl];
        UIImage *image = [UIImage imageWithData:data] ;
        dispatch_async(dispatch_get_main_queue(), ^{
            if(image){
                [SVProgressHUD dismiss];
                [imageView setImage:image];
            }else{//如果是选择一进来就全部加载的，需要把这句话去掉，不然会加载成功了
                //但仍然会显示加载失败.
                [SVProgressHUD showErrorWithStatus:@"加载失败"];
            }
        });
    });
}
```

#####图片的加载
这里可以选择，进入界面之后，加载全部内容，那么就需要在添加UIImageView的时候，利用上述方法设置图片了。
那么我选择的是每次滑动的时候去加载图片，如果已经加载的图片就不用再加载了。
那么我们就需要设置UIScrollView的代理方法
```
//滑动完成的时候执行
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
```
我们通过该方法在每次滑动完成的时候去加载,先获取当前滑动位置的图片视图对象,然后进行加载就可以了。
```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    [SVProgressHUD dismiss];
    NSInteger index = scrollView.contentOffset.x/kSCREENWIDTH;
    UIImageView *imageView = (UIImageView *)scrollView.subviews[index];
    self.selectedImageView = imageView;
    //翻动视图的时候加载图片
    if(!imageView.image)
        [self downloadImageWithPath:self.image_list[index] imageView:imageView];
}
```

#####如何保存图片
在写之前请在info.plist文件中加入访问本地图片权限,不然会崩溃的哦
```
Privacy - Photo Library Usage Description
```

将相机保存到本地的方法有很多种:
https://www.jianshu.com/p/bf20733ba19b
这位小哥总结了一下,我就不做重复的事了(虽然也做了不少,能懒就懒是我自己给自己插的flag)
然后我就使用了引用Photos库的方法

想要使用的话记得导入:
```
#import <Photos/Photos.h>
```
下面的方法使用起来就比价简单了，当然我对它的实现原理更加感兴趣，找个时间把原理实现添加到计划中吧。嗯，就这么愉快的说定了。
```
#pragma mark 下载图片
- (void)downLoadButtonAction{
    NSInteger index = self.scrollView.contentOffset.x/kSCREENWIDTH;
    UIImageView *imageView = (UIImageView *)self.scrollView.subviews[index];
    UIImage *image = imageView.image;
    [[PHPhotoLibrary sharedPhotoLibrary] performChanges:^{
            //写入图片到相册
            PHAssetChangeRequest *req = [PHAssetChangeRequest creationRequestForAssetFromImage:image];
    } completionHandler:^(BOOL success, NSError * _Nullable error) {
        if(success){
            [SVProgressHUD showSuccessWithStatus:@"保存成功"];
        }else{
            [SVProgressHUD showErrorWithStatus:@"保存失败"];
        }
    }];
}
```
关于浏览器部分基本就结束了,还有一个就是记得给UIScrollView添加一个点击就消失的手势事件:
```
//为滚动视图添加点击消失的事件
 UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(backAction)];
 [_scrollView addGestureRecognizer:tap];
```

## 如何获取网页中的所有图片
这一个步骤就得交给代理来做了。
首先我们就需要实现
```
//网页加载完成
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation;
```

在真实的情况下,有很多图片貌似获取不到,获取的里面也包含很多的空串,所以我们需要把得到的图片列表过滤一下。所以之前说的前端萌妹子是多么重要。
```
static  NSString * const imagesJS =
    @"function getImages(){\
    var objs = document.getElementsByTagName(\"img\");\
    var imgScr = '';\
    for(var i=0;i<objs.length;i++){\
    imgScr = imgScr + objs[i].src + '+';\
    };\
    return imgScr;\
    };";
    
    [webView evaluateJavaScript:imagesJS completionHandler:nil];
    [webView evaluateJavaScript:@"getImages()" completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSArray *urlArray = [NSMutableArray arrayWithArray:[result componentsSeparatedByString:@"+"]];
        //里面如果有空的需要过滤掉
        if(self.image_list.count==0){
            for (NSString *tempURL in urlArray) {
                if(!kStringIsEmpty(tempURL)){
                    [self.image_list addObject:tempURL];
                }
            }
        }
    }];
```

## 如何设置点击事件

点击事件,我觉得还是通过与前端的配合,他设置好之后我们直接调用比较好一点，然而我并没有这样可爱的萌妹子，所以,只能用js获取并且给它设置点击事件了。

```
//这段代码是显示单独的图片。
[webView evaluateJavaScript:@"function registerImageClickAction(){\
     var imgs = document.getElementsByTagName('img');\
     for(var i=0;i<imgs.length;i++){\
     imgs[i].customIndex = i;\
     imgs[i].onclick=function(){\
     window.location.href=''+this.src;\
     }\
     }\
     }" completionHandler:nil];
    
    [webView evaluateJavaScript:@"registerImageClickAction();"completionHandler:nil];
```
设置了之后，我们就可以看到点击图片然后显示的场景了。然而我们需要自己的图片浏览器啊，所有我们就要拦截点击事件，拦截到之后换上我们自己的控件进行显示。
```
//在发送请求之前，决定是否跳转,
-(void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler{
    
    //如果是跳转一个新页面
    if (navigationAction.targetFrame.request != nil) {
        NSString *selectedImgURL = navigationAction.request.URL.absoluteString;
        self.imageBrowseView = [GWImageBrowseView browseImages:self.image_list selectedImagePath:selectedImgURL];
        [self.view addSubview:self.imageBrowseView];
         decisionHandler(WKNavigationActionPolicyCancel);
    }else{
       decisionHandler(WKNavigationActionPolicyAllow);
    }
}
```

----
DEMO:https://github.com/yanggenwei/GWWebviewPro
可能还不太完善，等我完善了我会更新的哦。

</small>


