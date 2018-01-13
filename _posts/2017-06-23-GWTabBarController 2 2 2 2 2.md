---
layout: post
title: "构建自定义TabBarController"
date: 2017-06-23
excerpt: "自定义控件"
tags: [iOS][自定义控件]
post: true
comments: true
---
<small>
构建自定义TabBarController需要满足以下几个基本步骤:

1.  移除本身自带的tabBar,选用自定义tabBar
2.  tabBar上的button也需要自定义
3.  设定协议实现点击tabBar上的button实现控制器的跳转

能够满足以上的功能,我们得TabBarController就可以开始干活了。

满足之后,扩展的功能就因项目而定了。

## 第一步:构建三个类
1. XXTabBarController
2. XXTabBar
3. XXTabBarButton
名字就自己爱好取了。

## GWTabBarButton
我们先构建最简单的按钮,只需要设置它的标题,默认状态图标,选中状态图标,默认前景色,选中状态前景色就可以了。

```
import <UIKit/UIKit.h>

@interface GWTabBarButton : UIButton

/**
根据UITabBarItem属性进行初始化
@param tabBarItem tabBarItem对象
@return GWTabBarButton对象
*/
- (instancetype)initWithTabBarItem:(UITabBarItem *)tabBarItem;
@end
```

```
#import "GWTabBarButton.h"

@implementation GWTabBarButton

#pragma mark 初始化按钮
- (instancetype)initWithTabBarItem:(UITabBarItem *)tabBarItem{
if(self = [super init])
{
[self setTitle:tabBarItem.title forState:UIControlStateNormal];
[self setImage:tabBarItem.image forState:UIControlStateNormal];
[self setImage:tabBarItem.selectedImage forState:UIControlStateSelected];
[self setTitleColor:[UIColor grayColor] forState:UIControlStateNormal];
[self setTitleColor:[UIColor blueColor] forState:UIControlStateSelected];
}
return self;
}

@end
```
UITabBarItem 并不是一个控件,因为它是继承于UIBarItem,而UIBarItem继承于NSObject。 它更像是个临时储存站,通过它我们将对应的数据储存在它的对象中,最后在需要的地方提取出来赋值给相应的控件。因为它刚好拥有title,image,selectedImage三个属性,其实也是为tabBar而生的一种Model,所以利用它我们能够很好的传递数据。

## GWTabBar
自定义的tabBar我们选用UIView,而自定义tabBar作为一个View,可塑性更强,可以利用drawRect: 方法构建出不规则tabBar:
![不规则tabBar.png](http://upload-images.jianshu.io/upload_images/1120896-553098135d74c36c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面的其实就是在中间画了个圆.
下面的这个其实就是在中间加了个按钮
![不规则tabBar.png](http://upload-images.jianshu.io/upload_images/1120896-10e26cf06e77338e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如上所示,程序只有想不到,没有你办不到。
而我们今天只是做一个简单的自定义tabBar:
```
#import <UIKit/UIKit.h>
#import "GWTabBarButton.h"
@class GWTabBar;
@protocol GWTabBarDelegate<NSObject>
/**
实现切换页面

@param tabBar tabBar对象
@param oldIndex 曾经选中的按钮
@param newIndex 当前选中的按钮
*/
- (void)tabBar:(GWTabBar *)tabBar didSelectItemFormIndex:(NSInteger)oldIndex toIndex:(NSInteger)newIndex;
@end

@interface GWTabBar : UIView
//代理
@property (nonatomic,readwrite,weak)id<GWTabBarDelegate> delegate;
//设定当前选中的按钮
@property (nonatomic,readwrite,unsafe_unretained)GWTabBarButton   *selectButton;
/**
添加一个按钮

@param tabBarButton 按钮
*/
- (void)addTabBarItem:(UITabBarItem *)tabBarButton;

@end
```
要想实现点击tabBar上的按钮,然后去切换页面的功能,首先得利用代理去完成。因为我们将TabBarController上的tabBar给移除了,所以这方面功能只能我们自己实现了。而这个方法的原理也比较简单:
**我们为每个按钮绑定tag,通过不同的tag来识别我们是否点击了不同的按钮。当我们点击不同的按钮的时候,TabBarController也需要同时去设置它的selectedIndex属性来实现页面的切换。**

除了设置按钮点击事件之外,我们还需要向外部抛出一个方法(addTabBarItem:),来实现添加按钮的功能。有了按钮之后,我们再对按钮进行排列和绑定tag。

```
#import "GWTabBar.h"
@interface GWTabBar()

@end

@implementation GWTabBar

- (instancetype)initWithFrame:(CGRect)frame{
if(self = [super initWithFrame:frame])
{
[self setBackgroundColor:[UIColor whiteColor]];
[self setAlpha:0.7];
}
return  self;
}

#pragma mark 初始化tabBar
- (void)addTabBarItem:(UITabBarItem *)tabBarItem{

GWTabBarButton *tabBarButton = [[GWTabBarButton alloc]initWithTabBarItem:tabBarItem];

[tabBarButton addTarget:self
action:@selector(tabBarButtonAction:)
forControlEvents:UIControlEventTouchUpInside];

[self addSubview:tabBarButton];
}


#pragma mark 跳转页面设置
- (void)tabBarButtonAction:(GWTabBarButton *) tabBarButton{
if([self.delegate respondsToSelector:@selector(tabBar:didSelectItemFormIndex:toIndex:)]){
[self.delegate tabBar:self didSelectItemFormIndex:self.selectButton.tag toIndex:tabBarButton.tag];
}
//重新设置按钮的选中状态
self.selectButton.selected = NO;
//由于点击该方法后,表明已经切换了按钮对象,所以之前的按钮对象需要放弃,重新指定按钮对象
self.selectButton = tabBarButton;
self.selectButton.selected = YES;
}

#pragma mark 设置按钮位置和绑定tag
- (void)layoutSubviews{
[super layoutSubviews];

for (NSInteger index = 0;index<self.subviews.count;index++){
//从之前添加的子视图列表中获取按钮对象
GWTabBarButton *tabBtn = self.subviews[index];
//设定按钮布局,实现平均排列
tabBtn.frame = CGRectMake(index*self.frame.size.width/self.subviews.count, 2, self.frame.size.width/self.subviews.count, tabBarHeight);
tabBtn.titleLabel.font = [UIFont systemFontOfSize:GWTabBarFontSize];

//设定图标和标题偏移,实现图上字下的布局方式
CGSize btnImageBounds = tabBtn.imageView.bounds.size;
CGSize btnTitleBounds = tabBtn.titleLabel.bounds.size;
tabBtn.titleEdgeInsets = UIEdgeInsetsMake(0.0, -btnImageBounds.width/2, - btnImageBounds.height, btnImageBounds.width/2);
tabBtn.imageEdgeInsets = UIEdgeInsetsMake(-btnTitleBounds.height-5,btnTitleBounds.width/2, 0.0, -btnTitleBounds.width/2);

//绑定tag,便于tabBar代理方法的实现
tabBtn.tag = index;
}
}
@end
```
```
其中的layoutSubviews是对子视图重新布局时使用的，本身在父类是没有什么作用的,需要在子类中重写功能.而且layoutSubviews方法调用先于drawRect。

而layoutSubviews只有在以下情况下会被调用：
1. init初始化不会触发layoutSubviews,但是是用initWithFrame 进行初始化时，当rect的值不为CGRectZero时,也会触发
2. addSubview会触发layoutSubviews
3. 设置view的Frame会触发layoutSubviews，当然前提是frame的值设置前后发生了变化
4. 滚动一个UIScrollView会触发layoutSubviews
5. 旋转Screen会触发父UIView上的layoutSubviews事件
6.  改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件
```

## GWTabBarController
tabBar和tabBarButton都做完了之后,我们只需要将东西放在tabBarController上面就可以了。
```
#import <UIKit/UIKit.h>
#import "GWTabBar.h"
@interface GWTabBarController : UITabBarController<GWTabBarDelegate>

@end
```

```
#import "GWTabBarController.h"
#import "FirstViewController.h"
#import "SecondViewController.h"
#import "ThirdViewController.h"
#import "FourthViewController.h"

@interface GWTabBarController ()
@property (nonatomic,readwrite,strong)GWTabBar  *gwTabBar;
@end

@implementation GWTabBarController

#pragma mark 移除自带的tabBar
- (void)viewWillAppear:(BOOL)animated{
[super viewWillAppear:animated];

for(UIView *child in self.tabBar.subviews){
if([child isKindOfClass:[UIControl class]]){
[child removeFromSuperview];
}
}

}


- (void)viewDidLoad {
[super viewDidLoad];
[self initlizeTabBar];
[self initializeAllControllers];
//设定默认启动第一页
self.gwTabBar.selectButton = self.gwTabBar.subviews[0];
self.gwTabBar.selectButton.selected = YES;
}

#pragma mark 初始化自定义tabBar
- (void)initlizeTabBar{

//移除自带的tabBar,在之前先获取它的布局
CGRect rect = self.tabBar.frame;
//这里我只是感觉默认的高度有点高,微调了一下,如果不需要调整的话这里不需要重新设置
rect.origin.y = FRAMEHEIGHT - tabBarHeight;
rect.size.height = tabBarHeight;
[self.tabBar removeFromSuperview];
//创建tabBar
self.gwTabBar = [[GWTabBar alloc] initWithFrame:rect];
self.gwTabBar.delegate = self;
[self.view addSubview:self.gwTabBar];

}

#pragma mark 实现点击切换页面
- (void)tabBar:(GWTabBar *)tabBar didSelectItemFormIndex:(NSInteger)oldIndex toIndex:(NSInteger)newIndex
{
self.selectedIndex = newIndex;
}


#pragma mark 为TabBar初始化页面
- (void)initializeAllControllers{

[self setTabBarItemWithController:[FirstViewController new] title:@"第一页" defaultImageName:@"camera_n" selectedImageName:@"camera_s"];

[self setTabBarItemWithController:[SecondViewController new] title:@"第二页" defaultImageName:@"camera_n" selectedImageName:@"camera_s"];

[self setTabBarItemWithController:[ThirdViewController new] title:@"第三页" defaultImageName:@"camera_n" selectedImageName:@"camera_s"];

[self setTabBarItemWithController:[FourthViewController new] title:@"第四页" defaultImageName:@"camera_n" selectedImageName:@"camera_s"];

}


/**
将页面添加到TabBar中

@param controller 页面控制器
@param title 标题
@param defaultName 默认状态图标
@param selectedName 选中状态图标
*/
- (void)setTabBarItemWithController:(UIViewController *)controller
title:(NSString *)title
defaultImageName:(NSString *)defaultName
selectedImageName:(NSString *)selectedName{
//设置tabBarItem
UITabBarItem *tabBarItem = [[UITabBarItem alloc]initWithTitle:title image:[UIImage imageNamed:defaultName] selectedImage:[UIImage imageNamed:selectedName]];
controller.tabBarItem = tabBarItem;

//将页面加载到TabBar中
UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:controller];
[self addChildViewController:nav];

//将tabBarItem传入到TabBar中
[self.gwTabBar addTabBarItem:tabBarItem];
}

- (void)didReceiveMemoryWarning {
[super didReceiveMemoryWarning];
}

@end
```

最后附上Demo地址:https://github.com/yanggenwei/GWTabBarController


</small>


