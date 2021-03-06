---
layout: post
title:  "自定义文本框"
date:   2017-12-19
categories: 技术
---
<small>


![手机号输入框.png](http://upload-images.jianshu.io/upload_images/1120896-3f3eb9b460072782.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![密码输入框.png](http://upload-images.jianshu.io/upload_images/1120896-5fe2e3c203f37546.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![提现输入框.png](http://upload-images.jianshu.io/upload_images/1120896-b7da5e5424ed711f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如大家所看到那样，就是三种输入框，这个在很多软件上都有，实现起来也比较简单。
```
第一种：透明带图的框
就是边框色没有了，宽度也没有了。leftView设置成了图片，文字往左边移动了一点点
第二种: 同理的边框透明，但leftView设置了文字
第三种区别就是多了rightView
```
仔细分析可以看出来，它们其实都是一种文本输入框，只是一些样式的变化。所以可以做成一种类起对待。
只不过我们在创建的时候区分一下就行了。
区分某种类别，一般是用枚举了，因为差异就是是否是具有下划线的。所以我们大致分为两种:
```
typedef NS_ENUM(NSInteger,GWTextFieldStyle){
    GWTextFieldStyleWithNormal,    //正常
    GWTextFieldStyleWithUnline,    //下滑线
};
```
然后由于属性的多样化，我们的需求可能根据产品汪的变化而变化，但是我们又不想随便的去更改一个已经写好的类，所以我是决定使用一个model来作为参数，这样的话，以后修改model就可以了，不会去干扰样式代码造成什么问题。
```
#import <Foundation/Foundation.h>
/**
 自定义输入框控件数据源
 */
@interface GWTextFieldModel : NSObject
//标题内容
@property (nonatomic,readwrite,strong)id titleViewContent;
//右视图内容
@property (nonatomic,readwrite,strong)id rightViewContent;
//文本颜色
@property (nonatomic,readwrite,strong)UIColor *textColor;
//边框线宽度
@property (nonatomic,readwrite,unsafe_unretained)CGFloat borderWidth;
//边框线颜色
@property (nonatomic,readwrite,strong)UIColor *borderColor;
//背景颜色
@property (nonatomic,readwrite,strong)UIColor *backgroundColor;
//默认提示语
@property (nonatomic,readwrite,copy)NSString *placeholder;
//默认提示语颜色
@property (nonatomic,readwrite,strong)UIColor *placeHolderColor;
//键盘类型
@property (nonatomic,readwrite,unsafe_unretained)UIKeyboardType keyboardType;
//是否显示密码
@property (nonatomic,readwrite,unsafe_unretained)BOOL isPassword;

@end
```
可以看到我在model当中存在了两个特殊的参数，貌似与属性无关，而且还是id类型
```
titleViewContent
rightViewContent
```
我是设想通过titleViewContent就可以直接分辨出文字或者视图,使得在调用的时候直接填入参数就可以了。
而rightViewContent也是同理,如果添加了rightView的情况下，我们加入即可。也就是说，我们只需要填入参数，其它我们在内部直接进行实现。

创建好model之后，我们就可以创建我们的自定义文本框了:
```
#import <UIKit/UIKit.h>
@class GWTextFieldModel;

typedef NS_ENUM(NSInteger,GWTextFieldStyle){
    GWTextFieldStyleWithNormal,    //正常
    GWTextFieldStyleWithUnline,    //下滑线
};


/**
 自定义输入框控件类
 */
@interface GWTransTextField : UITextField<UITextFieldDelegate>


/**
 创建一个默认样式的输入框·

 @param frame 布局
 @param textFieldModel 文本框属性设置数据源
 @return instance of GWTransTextField
 */
+ (instancetype)createNormalTextFieldWithFrame:(CGRect)frame
                                textFieldModel:(GWTextFieldModel *)textFieldModel;


/**
 创建一个带有下划线的输入框

 @param frame 布局
 @param textFieldModel 文本框属性设置数据源
 @return instance of GWTransTextField
 */
+(instancetype)createUnlineTextFieldWithFrame:(CGRect) frame
                               textFieldModel:(GWTextFieldModel *)textFieldModel;


```
这里有几个习惯需要注意的:
```
1. 在.h文件中，一般不要直接#import一个类,使用@class就可以了,这样就不用一编译时就导入文件了。
然后在.m文件中使用#import导入文件，也就是说我们只需要在运行时去导入文件。

2. 给类添加注释。有必要的话，最好还要解释一下这个类的功能。 这样方便你自己和别人第一时间知道这个类的作用。
短时间还好，时间长了，对于类没那么熟悉的时候，就会产生迟钝，造成多余的熟悉时间成本。

3.就是写注释了，这是个必须的工作。
```
从上面方法的创建，我们可以看到，我将其分为默认类型的,就是我们常用的文字开头，不透明文本框，还有一种带下划线且透明的输入框。

那么我们在是实现的时候，就可以了解到，我们在实现默认文本框的时候，自由度比较高，它们的属性基本都是由外界的model所决定。而具有下划线的文本框，很多样式已经固定了。所以就形成了如下的结果:
```
#pragma mark - 初始化默认文本框
- (instancetype)initWithNormalFrame:(CGRect)frame textFieldModel:(GWTextFieldModel *)textFieldModel{
    if(self = [super initWithFrame:frame])
    {
        textFieldModel.placeHolderColor = nil;
        [self createUILayoutWithModel:textFieldModel textFieldStyle:GWTextFieldStyleWithNormal];
    }
    return  self;
}
#pragma mark - 初始化下划线文本框
- (instancetype)initWithUnlineFrame:(CGRect)frame textFieldModel:(GWTextFieldModel *)textFieldModel{
    if(self = [super initWithFrame:frame])
    {
        textFieldModel.placeHolderColor = [PMBTools colorWithHexString:@"#BEBEBE"];
        textFieldModel.borderWidth = 0;
        textFieldModel.textColor = [UIColor whiteColor];
        textFieldModel.borderColor = [UIColor clearColor];
        textFieldModel.backgroundColor = [UIColor clearColor];
        [self createUILayoutWithModel:textFieldModel textFieldStyle:GWTextFieldStyleWithUnline];
    }
    return  self;
}
```
可以看到，我们在设定默认文本框方法的时候，里面并没有具体的参数设置，而在下划线的初始化方法中，我们设置了很多默认的属性。也就是说，将必要的东西尽量去封装。

这里要提的习惯就是:
```
使用#pragma mark 命令,通过该命令，可以形成如下图的导航效果,可以快速定位你写的方法。
使用#pragma mark - 命令可以形成分组效果
```
![导航.png](http://upload-images.jianshu.io/upload_images/1120896-37eee664eabd9540.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

下面我们就进入到重点了，如何进行识别titleViewContent的类型？
```
if([textFieldModel.titleViewContent isKindOfClass:[NSString class]]){
      //实现将文本框作为leftView
}else if([textFieldModel.titleViewContent isKindOfClass:[UIImage class]]){
      //实现将图片设为leftView
}else if([textFieldModel.titleViewContent isKindOfClass:[UIView class]]){
      //实现将普通视图设为leftView
}

if([textFieldModel.rightViewContent isKindOfClass:[UIView class]]){
      //实现将普通视图设为rightView
}
```

通过isKindOfClass方法，我们判断出我们的titleViewContent是什么类型的，我们就可以根据不同的类型进行处理。
当然咯，你会发现这样做之后，单纯的去设置leftView什么的还是显示不出来，那是因为，leftViewMode的问题
```
typedef NS_ENUM(NSInteger, UITextFieldViewMode) {
    UITextFieldViewModeNever,
    UITextFieldViewModeWhileEditing,
    UITextFieldViewModeUnlessEditing,
    UITextFieldViewModeAlways
};
```
系统默认的是UITextFieldViewModeNever,它并不会显示，所以我们要将该值设置成UITextFieldViewModeAlways，让它一直显示。
```
self.leftView = _titleView;
self.leftViewMode = UITextFieldViewModeAlways;
```
当我们做完基本的属性设置之后，就可以调用，并显示了。
```
@property (nonatomic,readwrite,strong)GWTransTextField *txtUserName;

- (GWTransTextField *)txtUserName{
    if(!_txtUserName){
        CGFloat margin = 20;
        CGFloat height = 50;
        GWTextFieldModel *textFieldModel = [GWTextFieldModel new];
        textFieldModel.placeholder = @"请输入手机密码";
        textFieldModel.titleViewContent = [UIImage imageNamed:@"Phone"];
        _txtUserName = [GWTransTextField createUnlineTextFieldWithFrame:
                       (CGRect){margin,200,kSCREENWIDTH-margin*2,height} textFieldModel:textFieldModel];
    }return _txtUserName;
}
```

简单的创建，添加之后，我们会发现，效果不太好
![开始的效果.png](http://upload-images.jianshu.io/upload_images/1120896-a491b30dc507d01b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这图片和字怎么都挤一块去了，完全不符合我们比产品汪还要好的审美啊，所以我们需要调整样式
调整样式需要重写父类方法
```
- (CGRect)textRectForBounds:(CGRect)bounds
```
这里的bounds参数，就是当前文本框的bounds，所以直接调整就可以了。
```
    bounds.origin.x = marginLeft(_titleView) + 15;
    //如果有右视图的话,文本框输入的范围要缩小到右视图的前面
    if(_rightView){
        bounds.size.width = bounds.size.width-bounds.origin.x-_rightView.bounds.size.width-15;
    }
```
如果还需要调整图标大小，方位什么的需要重写该方法:
```
- (CGRect)leftViewRectForBounds:(CGRect)bounds{
    bounds.origin.x = 10;
    bounds.origin.y = bounds.size.height/2-imageSize.height/2;
    bounds.size.width = imageSize.width;
    bounds.size.height = imageSize.height;
}
- (CGRect)rightViewRectForBounds:(CGRect)bounds;
```
如果你在实现的过程中遇到 输入的时候文本框出现样式问题就实现重写如下方法，保持与textRectForBounds方法样式一致了，反正我是遇到过的。
```
- (CGRect)editingRectForBounds:(CGRect)bounds
```
最终的样式
![手机号输入框.png](http://upload-images.jianshu.io/upload_images/1120896-87cf4edebbf8b40b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果你需要添加在尾部的清除按钮:
![清楚按钮.png](http://upload-images.jianshu.io/upload_images/1120896-f35bd31a5bd32d1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
self.clearButtonMode = UITextFieldViewModeWhileEditing;
同理,mode也是分了各种情况，包括，, 
typedef NS_ENUM(NSInteger, UITextFieldViewMode) {
    UITextFieldViewModeNever, //不显示
    UITextFieldViewModeWhileEditing,//有内容，输入的时候显示
    UITextFieldViewModeUnlessEditing,//有内容，不输入的时候显示
    UITextFieldViewModeAlways//有内容,一直显示
};
```

调用的话如下所示:
```
- (GWTransTextField *)txtUserName{
    if(!_txtUserName){
        NSInteger buttonMargin = 20;
        CGFloat buttonHeight = 50;
        GWTextFieldModel *textFieldModel = [GWTextFieldModel new];
        textFieldModel.placeholder = @"请输入手机密码";
        textFieldModel.titleViewContent = [UIImage imageNamed:@"Phone"];
        _txtUserName = [GWTransTextField createUnlineTextFieldWithFrame:(CGRect){buttonMargin,200,kSCREENWIDTH-buttonMargin*2,buttonHeight} textFieldModel:textFieldModel];
    }return _txtUserName;
}
```

然后直接添加到view中即可。

最后三种实现的效果如下:
![最终效果.png](http://upload-images.jianshu.io/upload_images/1120896-f184c7d7023bb148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


最后贴上代码地址:
https://github.com/yanggenwei/GWTextField

</small>


