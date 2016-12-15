---
layout: post
title: "swift 3.0 学习计划"
date: 2012-05-22
excerpt: "通往大神之路"
tags: [swift, iOS]
post: true
comments: true
---

## swift 3.0 学习计划（一）
* * *
* 基础语法的学习
* 线程和网络访问的学习
* 学习一个swift做的开源APP
* * *
* 由于已经有了OC的基础,一些基础部分就快速的过一遍了,推荐的完整学习网站: * <https://github.com/numbbbbb/the-swift-programming-language-in-chinese>

# 第一章:基础语法的学习

### 变量和常量声明:

变量定义:
```swift
var age = 0.0,height = 0.0
```
常量定义:
```
let SCREEN = 1080
```

### 类型标注:

```
var name:String
var red,green,blue:Double
var type:String = "机械" //可以在设定类型的时候赋初始值

```
这里得说明一下,如果不写类型标注,那么switf仍然可以自动识别该变量所使用的类型,而类型标注可以使得可读性更强,通过类型还是比较容易的知道该变量作用的.
3.关于命名,仍然是和OC差不多,不能包含数学符号，箭头，保留的（或者非法的）Unicode 码位，连线与制表符,也不能以数字开头,我想，有命名规范的存在，大家应该不会这样命名吧
```
var 🐶="中华田园犬,上古神兽"
```
尽管这样是可以的。
```
命名规范:
1.一般采用驼峰命名法
2.顾名思义
3.常量大写
```

### 输出

```
var name = "根威"
print(name)
print("我的名字是:\(name)")
print("我的名字是:",name)
```

### 类型推断和类型安全

Swift是一个类型安全的语言.类型安全的语言可以让你清楚的知道代码要处理的类型。如果你的代码需要一个String,你绝对不可能不小心传进去一个Int.
由于Swift是类型安全的,所以它会在编译你的代码时进行类型检查,并把不匹配的类型标记为错误.这可以让你在开发的时候发现并修复错误。
而关于类型推测,简单来说就是当你没有指定一个变量或常量具体数据类型的时候,那么Swift就会使用类型推断来选择合适的类型.

```
var name = "测试员"
//name 会被推测为String类型
let AGE = 30
//AGE 会被推测为Int类型
let SCORE = 3 + 9.0
//如果表达式中同时出现了整数和浮点数，会被推断为 Double 类型：
```

### 类型转换

```
let NUMBER = 31
let POINT = 0.4
let RESULT = Double(NUMBER) + POINT
```

转换必须是显式的,`数据类型(转换的对象)`,这种方式为强制类型转换,需要注意的是转换的时候** 不会四舍五入 **,例如3.6转换成整数就会变成3.

### 类型别名

类型别名就是给现有类型定义另一个名字.在Swift中可以用`typealias`关键字来定义类型别名。

```
typealias GWInt8 = UInt8
var age = GWInt8.min
print("age:\(age)")
```

上面的例子就是给UInt8类型定义了另一个名字GWInt8,这样我们以后就可以使用GWInt8定义UInt8类型的数据了。

### 可选类型

C和OC中并没有可选类型的概念.

`太多的概念,我看了头疼,我们直入主题`

简单来说,optional给予了变量或常量一种保护措施,防止为空的情况存在.

```
var age:Int
print(age)
//这种情况在编译时就会发生错误,会提示你需要初始化.
```

这时候就需要optional的存在

```
var age:Int?
print(age)
```

这种情况会给age设定一个nil的值.但是仍然会有警告,会让我们设定默认值,不得不说,这种方式我很喜欢,在其他语言还要if判断

```
var age:Int?
print(age ?? 0)
```

设定默认值之后,如果age没有赋值,那么就会采用0作用age的值。

##### 显式拆包

要知道opional类型的值不能被直接使用的

```
var str:String? = "Hello World"
print(str)
输出结果:
Optional("Hello World")
```

这个时候需要用到!操作符来对optional进行拆包

```
var str:String? = "Hello World"
print(str!)
输出结果
Hello World
```

至于为什么要拆包，那是因为optional类型是一个枚举:

```
enum Optional<T> : Reflectable, NilLiteralConvertible {
    case None
    case Some(T)
    init()
    init(_ some: T)

    /// Haskell's fmap, which was mis-named
    func map<U>(f: (T) -> U) -> U?
    func getMirror() -> MirrorType
    static func convertFromNilLiteral() -> T?
}
```

当Optional没有值时，返回的nil其实就是Optional.None，即没有值。除了None以外，还有一个Some，当有值时就是被Some<T>包装的真正的值，所以我们拆包的动作其实就是将Some里面的值取出来。

##### 隐式拆包

除了显式拆包，Optional还提供了隐式拆包，通过在声明时的数据类型后面加一个！来实现：
```
var str: String! = "Hello World!"
print(str)
输出结果:
Hello World
```
这种做法必须在确信你的变量能保证被正确初始化，那就可以这么做，否则还是不要这样做了

在写法方面,optional还有一种写法:
```
var str: String! = "Hello World!"
var str2: Optional<String> = "Hello World!"
```

```
注意：
Swift 的 nil 和 Objective-C 中的 nil 并不一样。在 Objective-C 中，nil 是一个指向不存在对象的指针。在 Swift 中，nil 不是指针——它是一个确定的值，用来表示值缺失。任何类型的可选状态都可以被设置为 nil，不只是对象类型。
```

关于optional更加深入的以后来探讨,当然,这里提供几个链接我觉得不错的,可以看看:

<https://www.zhihu.com/question/28026214?sort=created>
<https://www.zhihu.com/question/26512698>





