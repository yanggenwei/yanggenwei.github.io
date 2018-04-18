---
layout: post
title:  "自定义验证码按钮的实现"
date:   2017-12-21
categories: 技术
---
 
  
今天要做的是一个倒计时按钮,也是项目当中经常遇到的一类需求。
最终实现效果如下:
![验证码按钮.png](http://upload-images.jianshu.io/upload_images/1120896-2be99230194bf93f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而实现这种验证码,方式很多,有人用NSTimer进行开发的，也有用dispatch_source_t 进行开发的。
但是NSTimer在运用过程中需要注意的地方很多，如果使用不当就会造成内存无法释放,导致内存溢出的可能。
所以，我后来又改用dispatch_source_t 完成了这一需求。
至于dispatch_source_t 的具体详解，有需要的可以去查看相关资料，毕竟今天的主题是利用它其中的一项功能dispatch_source_set_timer。

资料传送门:
https://developer.apple.com/library/content/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html

https://www.tuicool.com/articles/nIRJf2e
http://www.dreamingwish.com/article/grand-central-dispatch-basic-3.html

下面开始我们的正题:
```
由于该项目需要用到上一篇中的文本输入框,所以直接在上一篇的项目中进行使用了。
同样是选用Model作为参数的模式
其实关于模式的选用,就是根据自己的工作环境来决定的。
我选用这种模式的原因就是，很长时间没看的代码，需要时间成本去了解.
而这种模式的可读性较强，而且将属性设置和构建分离，方便以后的维护。
```

model的数据也就是常见的属性和一些需要自定义属性设置，当然，以后扩展还会很多，今天的按钮只是其中之一，所以目前属性就这些吧.
```
@interface GWButtonModel : NSObject
//标题
@property (nonatomic,readwrite,copy)NSString *title;
//标题颜色
@property (nonatomic,readwrite,strong)UIColor *titleColor;
//边框颜色
@property (nonatomic,readwrite,strong)UIColor *borderColor;
//边框宽度
@property (nonatomic,readwrite,unsafe_unretained)CGFloat borderWidth;
//刷新计时
@property (nonatomic,readwrite,unsafe_unretained)NSTimeInterval refreshTime;
//字体
@property (nonatomic,readwrite,strong)UIFont *font;
//背景色
@property (nonatomic,readwrite,strong)UIColor *backgroundColor;
//圆角
@property (nonatomic,readwrite,unsafe_unretained)CGFloat cornerRadius;
@end
```

在构建GWButton类之前，我们应该去分析我们的实际应用场景,所以我先来说说我在项目中用的场景:

```
1. 点击按钮 
2. 启用代理到控制器中判断用户是否存在(因为是可变的需求，所以不适合放在GWButton类中)
3. 如果用户存在,那么执行获取验证码的网络请求
(我将获取验证码的网络请求放在GWButton当中,因为这是一个公用方法，无论其他需求怎么变，验证码你得获取吧。也减小了控制器中代码的冗余)
4. 获取到验证码之后开始倒计时。否则就是保持原样
5. 倒计时完毕恢复原来的样式
```

分析出了完整的需求之后，事情就简单多了
1. 构建GWButton类

```
@protocol GWSecurityCodeDelegate <NSObject>

/**
 发送验证码
 */
- (void)sendSecurityCode;
@end

@interface GWButton : UIButton

@property (nonatomic,readwrite,weak)id<GWSecurityCodeDelegate>   securityCodeDelegate;

/**
 验证码

 @param frame 布局
 @param buttonModel 按钮数据模型

 */
+ (instancetype)securityCodeButtonWithFrame:(CGRect)frame
                                buttonModel:(GWButtonModel *)buttonModel;

/**
 发送获取验证码请求

 @param type 验证码类型
 @param tel 手机号码
 @param intervalTime 倒计时时间
 */
- (void)securityCodeType:(NSString *)type
           userTelephone:(NSString *)tel
              refresTime:(NSTimeInterval)intervalTime;
@end
```
```
这里的两个说明:
1. 为什么要用securityCodeDelegate命名,不用delegate.因为验证码只是这个按钮的其中一种。
以后还会添加别的按钮类型，它们的代理可能是别的类型。这样命名更加容易识别。就像WKWebview中的两个代理一样。

2. 因为获取验证码的请求，根据不同后台人员，写的可能不同，所以我只能列出个样例，这里需要的是验证码类型和手机号码两个参数。

```
接下来我们就看一下，我们的核心代码了。
其实关于为什么选用dispatch_source，我们可以看一下，官方的介绍:
```
A dispatch source is a fundamental data type that coordinates the processing of specific low-level system events. 
Grand Central Dispatch supports the following types of dispatch sources:
Timer dispatch sources generate periodic notifications.
```
好吧。有很多条，我就不展示了，上面有网址自己看了。
那么第一条就是定时器资源的使用。我们都知道dispatch系列存在都是不用我们自主管理线程和内存释放的。这个就要比NSTimer有优势多了。

而且下面这段话也说明了的Dispatch Sources方便之处:
```
Unlike tasks that you submit to a queue manually, dispatch sources provide a continuous source of events for your application. 
A dispatch source remains attached to its dispatch queue until you cancel it explicitly.
 While attached, it submits its associated task code to the dispatch queue whenever the corresponding event occurs. 
Some events, such as timer events, occur at regular intervals but most occur only sporadically as specific conditions arise. 
For this reason, dispatch sources retain their associated dispatch queue to prevent it from being released prematurely while events may still be pending.
```

简单来说就是: 
```
Unlike tasks that you submit to a queue manually, dispatch sources provide a continuous source of events for your application. 
与您手动提交到队列的任务不同，分派源为您的应用程序提供了一个连续的事件源(有道翻译，你值得拥有)
```
我们只需要设定好参数，结束和需要改变的事件就可以了，其他的它会帮我们做好。

下面来看实际的代码:
```
 __block int timeout = intervalTime; //倒计时时间

//创建dispatch_source源
1. dispatch_source_t   _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,
                                       dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
//间隔时间
uint64_t  interval = 1.0*NSEC_PER_SEC;
//获取开始的时间
dispatch_time_t  start = dispatch_walltime(NULL, 0);
//设置执行的时间
2. dispatch_source_set_timer(_timer, start, interval, 0);

//设置资源回调.
3.dispatch_source_set_event_handler(_timer, ^{
       if(timeout<=0){//倒计时结束,关闭的事件
  
       }else{  //计时中的事件
      
            timeout--;
        }
   });

//启动_timer
4. dispatch_resume(_timer);
```

上面代码中的NSEC_PER_SEC嘛意思?
```
#define NSEC_PER_SEC 1000000000ull
#define USEC_PER_SEC 1000000ull
#define NSEC_PER_USEC 1000ull

NSEC：纳秒。
USEC：微秒。
SEC：秒
PER：每
1   NSEC_PER_SEC，每秒有多少纳秒。
2   USEC_PER_SEC，每秒有多少毫秒。（注意是指在纳秒的基础上）
3   NSEC_PER_USEC，每毫秒有多少纳秒。
```

最后就是简单的执行顺序:
执行按钮事件:
```
#pragma mark 验证码刷新事件
- (void)securityCodeButtonAction:(GWButton *)button{
    
  //这里写的目的是为了防止用户频繁点击按钮造成多次请求的情况
  //网上一些其他的方法例如 分类什么的都试过了还是这个好用。
    button.enabled = NO;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        button.enabled = YES;
    });
    
    if([button.securityCodeDelegate respondsToSelector:@selector(sendSecurityCode)]){
        [button.securityCodeDelegate sendSecurityCode];
    }
}
```

然后执行代理:
```
- (void)securityCodeType:(NSString *)type
           userTelephone:(NSString *)tel
              refresTime:(NSTimeInterval)intervalTime{
    //可以将网络请求的方法放在这里,请求成功执行开始计时的方法。
    //还有这个请求只是放在这里给大家看个样例，demo中是没有的。
    kWeakSelf
    [GWVerificationCodeManager GWGetVerifyNUmberRequstWithPhone:[PMBTools convertNullOrNil:tel]
                                                             type:[PMBTools convertNullOrNil:type]
              success:^(id responseobject) {
                  NSDictionary *dict = (NSDictionary *)responseobject;
                  NSLog(@"验证码请求结果:%@",dict);
                  if([dict[@"resCode"] isEqualToString:@"020000"]){
                      [SVProgressHUD showSuccessWithStatus:dict[@"resMsg"]];
                      [weakself beginToCountDownWithRefresTime:intervalTime];
                  }else{
                      [SVProgressHUD showSuccessWithStatus:dict[@"resMsg"]];
                  }
              } failure:^(NSError *error) {
                  
              } progress:^(float progress) {
                  
              }];
}
```

最后就是执行倒计时了

```
- (void)beginToCountDownWithRefresTime:(NSTimeInterval)intervalTime{
    __block int timeout = intervalTime; //倒计时时间
    kWeakSelf
    dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,  dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
    
    dispatch_source_set_timer(_timer, dispatch_walltime(NULL, 0),1.0*NSEC_PER_SEC, 0);
    
    dispatch_source_set_event_handler(_timer, ^{
        if(timeout<=0){//倒计时结束,关闭
            dispatch_source_cancel(_timer);
            dispatch_async(dispatch_get_main_queue(), ^{
                [weakself setUserInteractionEnabled:YES];
                [weakself setTitle:@"获取验证码" forState:UIControlStateNormal];
                [weakself setBackgroundImage:[PMBTools convertUIColorToUIImage:[PMBTools colorWithHexString:@"#FF9129"]] forState:UIControlStateNormal];
            });
        }else{
            int seconds = (timeout==60?timeout-1:timeout) % 60;
            dispatch_async(dispatch_get_main_queue(), ^{
                //设置界面的按钮显示 根据自己需求设置
                [weakself setBackgroundImage:[PMBTools convertUIColorToUIImage:getColor(207, 204, 207, 1)] forState:UIControlStateNormal];
                [weakself setTitle:[NSString stringWithFormat:@"重新获取(%ds)",seconds]
                          forState:UIControlStateNormal];
                [weakself setUserInteractionEnabled:NO];
            });
            timeout--;
        }
    });
    
    dispatch_resume(_timer);
}
```

demo地址:
https://github.com/yanggenwei/GWTextField




