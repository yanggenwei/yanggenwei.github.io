---
layout: post
title: "iOS中防止数组越界崩溃的解决方案"
date: 2017-06-23
excerpt: "runtime"
tags: [iOS][runtime]
post: true
comments: true
---
<small>
项目中经常遇到数组越界的情况，这是个很烦人的问题，所以，就思考了下如何避免这样的问题。
首先我们获取数组元素的方式分为:
```
NSArray *array = @[@1,@2];
NSLog(@"arrary: %@",[array objectAtIndex:2]);
NSLog(@"arrary: %@",array[2]);
```
是的，通过objectAtIndex和[] 方式。

然后我们是不是第一个想法就是写分类然后重写？
嗯，我的确是试了一下，然后发现并没有用，系统这样提示我:
> Category is implementing a method which will also be implemented by its primary class

WTF? 警告，说这里重写也没有用？那怎么办？
而且还有一个问题就是，objectAtIndex方法很明确了，那么[] 这是个什么鬼？这也能用方法来表示？是的。
有两种方式可以知道：
第一种直接看错误:
```
2017-12-29 19:25:36.510723+0800 OCTest[39152:4167702] *** Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndexedSubscript:]: index 3 beyond bounds [0 .. 1]'
*** First throw call stack:
(
	0   CoreFoundation                      0x00007fff4150700b __exceptionPreprocess + 171
	1   libobjc.A.dylib                     0x00007fff680e5c76 objc_exception_throw + 48
	2   CoreFoundation                      0x00007fff41548514 _CFThrowFormattedException + 202
	3   CoreFoundation                      0x00007fff415b9201 -[__NSArrayI objectAtIndexedSubscript:] + 97
	4   OCTest                              0x0000000100000e55 main + 245
	5   libdyld.dylib                       0x00007fff68cd5115 start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

我们人为越界了一下，发现了错误所在
` [__NSArrayI objectAtIndexedSubscript:]`
是的，这里告诉我们就是它发生了错误.
第二种：
```
在main.m文件中写下如下代码:
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSArray *array = @[@1,@2];
        NSLog(@"arrary: %@",[array objectAtIndex:2]);
        NSLog(@"arrary: %@",array[3]);
    }
    return 0;
}

然后通过命令行:
clang -rewrite-objc main.m
得到
main.cpp
拉倒最后:
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
        NSArray *array = ((NSArray *(*)(Class, SEL, ObjectType  _Nonnull const * _Nonnull, NSUInteger))(void *)objc_msgSend)(objc_getClass("NSArray"), sel_registerName("arrayWithObjects:count:"), (const id *)__NSContainer_literal(2U, ((NSNumber *(*)(Class, SEL, int))(void *)objc_msgSend)(objc_getClass("NSNumber"), sel_registerName("numberWithInt:"), 1), ((NSNumber *(*)(Class, SEL, int))(void *)objc_msgSend)(objc_getClass("NSNumber"), sel_registerName("numberWithInt:"), 2)).arr, 2U);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_2d_q947d5pn4z3dfyq0j4vqsq840000gn_T_main_9976d7_mi_0,((id (*)(id, SEL, NSUInteger))(void *)objc_msgSend)((id)array, sel_registerName("objectAtIndexedSubscript:"), (NSUInteger)1));
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_2d_q947d5pn4z3dfyq0j4vqsq840000gn_T_main_9976d7_mi_1,((id (*)(id, SEL, NSUInteger))(void *)objc_msgSend)((id)array, sel_registerName("objectAtIndex:"), (NSUInteger)1));
    }
    return 0;
}
```

通过代码我们也知道了[]调用的方式:
`sel_registerName("objectAtIndexedSubscript:")`
里面的`objectAtIndexedSubscript `方法就是当我们使用[]的时候底层调用的方法。
当然，它也无法重写。

知道了原因，我们却无法重写，真是一个悲伤的故事。于是我找啊找啊，就找到了一种可以替代方法的方法。不给上，哥就不上了? 那也太怂了是吧。
我找到的就是runtime中的替换方法。
官方解释网址:
>https://developer.apple.com/documentation/objectivec/1418530-class_getinstancemethod

>Method class_getInstanceMethod(Class cls, SEL name);
Returns a specified instance method for a given class.
为给定的类返回指定的实例方法。

我们先通过这个方法获取指定的实例方法.

>https://developer.apple.com/documentation/objectivec/1418769-method_exchangeimplementations

>void method_exchangeImplementations(Method m1, Method m2);
Exchanges the implementations of two methods.
交换两种方法的实现。

然后我们再自己实现个方法，这方法里面我们做一下规避操作。比如用try catch把异常给捕捉起来，然后打上日志。就不用担心会崩溃，也不知道哪里发生了错误。当然具体的方法要根据业务场景自我实现。

下面的__NSArrayI 和__NSArrrayM分别代表不可变数组和可变数组的真实类型
```
NSLog(@"type of array:%@",[array class]);
NSLog(@"type of mutableArray:%@",[mArray class]);

结果如下:
2017-12-29 19:46:08.048887+0800 OCTest[39319:4202006] type of array:__NSArrayI
2017-12-29 19:46:08.048905+0800 OCTest[39319:4202006] type of mutableArray:__NSArrayM
```


代码如下:
```
#import "NSArray+BoundsOfRang.h"
#import <objc/runtime.h>

@implementation NSArray (BoundsOfRang)

+ (void)load{
    [super load];
    // 替换不可变数组中的方法 objectAtIndex
    Method oldObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(objectAtIndex:));
    Method newObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(newObjectAtIndex:));
    method_exchangeImplementations(oldObjectAtIndex, newObjectAtIndex);
    // 替换不可变数组中的方法 []调用的方法
    Method oldMutableObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(objectAtIndexedSubscript:));
    Method newMutableObjectAtIndex =  class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(newObjectAtIndexedSubscript:));
    method_exchangeImplementations(oldMutableObjectAtIndex, newMutableObjectAtIndex);
    
    // 替换可变数组中的方法 objectAtIndex
    Method oldMObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayM"), @selector(objectAtIndex:));
    Method newMObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayM"), @selector(newMutableObjectAtIndex:));
    method_exchangeImplementations(oldMObjectAtIndex, newMObjectAtIndex);
    // 替换可变数组中的方法  []调用的方法
    Method oldMMutableObjectAtIndex = class_getInstanceMethod(objc_getClass("__NSArrayM"), @selector(objectAtIndexedSubscript:));
    Method newMMutableObjectAtIndex =  class_getInstanceMethod(objc_getClass("__NSArrayM"), @selector(newMutableObjectAtIndexedSubscript:));
    method_exchangeImplementations(oldMMutableObjectAtIndex, newMMutableObjectAtIndex);
}

- (id)newObjectAtIndex:(NSUInteger)index{
    if (index > self.count - 1 || !self.count){
        @try {
            return [self newObjectAtIndex:index];
        } @catch (NSException *exception) {
            NSLog(@"不可数组越界了");
            return nil;
        } @finally {

        }
    }
    else{
        return [self newObjectAtIndex:index];
    }
}

- (id)newObjectAtIndexedSubscript:(NSUInteger)index{
    if (index > self.count - 1 || !self.count){
        @try {
            return [self newObjectAtIndexedSubscript:index];
        } @catch (NSException *exception) {
            NSLog(@"不可数组越界了");
            return nil;
        } @finally {
        }
    }
    else{
        return [self newObjectAtIndexedSubscript:index];
    }
}



- (id)newMutableObjectAtIndex:(NSUInteger)index{
    if (index > self.count - 1 || !self.count){
        @try {
            return [self newMutableObjectAtIndex:index];
        } @catch (NSException *exception) {
            NSLog(@"可变数组越界了");
            return nil;
        } @finally {
            
        }
    }
    else{
        return [self newMutableObjectAtIndex:index];
    }
}

- (id)newMutableObjectAtIndexedSubscript:(NSUInteger)index{
    if (index > self.count - 1 || !self.count){
        @try {
            return [self newMutableObjectAtIndexedSubscript:index];
        } @catch (NSException *exception) {
            NSLog(@"可变数组越界了");
            return nil;
        } @finally {
        }
    }
    else{
        return [self newMutableObjectAtIndexedSubscript:index];
    }
}
@end
```
</small>


