---
layout: blog
technology: true 
background: purple
background-image: http://p4wibncwo.bkt.clouddn.com/calum-lewis-387612-unsplash.jpg
title:  "Swift - 类和结构体"
date:   2018-01-31
category: 技术
tags:
- iOS
- Swift
- Class
- Struct
---
在面向过程的语言中,要想实现类似类的功能只能借助结构体,其实从OC源码也能看出来,类的组成本就是复杂的结构体实现的。
而在Swift中结构体的功能被扩大化了，基本拥有了和类差不多的功能:

>* 定义属性
>* 定义方法
>* 定义getter和setter
>* 可以定义初始化器来设置初始状态
>* 实现扩展的功能
>* 遵循协议,并实现协议的方法
>* 结构总是被复制，并且不使用引用计数。

类具有结构不具备的附加功能:

>* 继承允许一个类继承另一个类的特征。
>* 类型转换使您能够在运行时检查和解释类实例的类型。
>* 初始化器使一个类的实例能够释放它所分配的任何资源。
>* 引用计数允许多个引用到一个类实例。

# 定义结构体和类
```
class className {

}

struct structName {

}
```
我们为类和结构体添加一些变量和方法:
```
struct Resolution {
    var width = 0
    var height = 0
    
    func resolutionFun() {
        print("method of struct Resolution")
    }
}

class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
    
    func VideoMode() {
        print("method of class VideoMode")
    }
}

```

# 类和结构体的初始化
定义是一回事,初始化又是另一回事。我们来看如何初始化它们:
let `object Name` = `Class Name`()
let  `variable Name` = `Struct Name`() 

```
let someResolution = Resolution()
let someVideoMode = VideoMode()
```

# 访问属性
Swift中访问属性使用的是链式结构,访问属性的时候使用 `.`
`object name`.`propertyName`
```
print("The width of someResolution is \(someResolution.width)")
print("The width of someVideoMode is \(someVideoMode.resolution.width)")
```
类或者结构中的属性默认情况是可写可读的:
```
someVideoMode.resolution.width = 1280
print("The width of someVideoMode is now \(someVideoMode.resolution.width)")
```
>关于直接修改类中结构的数据,这一点与OC是不同的，OC中并没有处理好这一层级关系,导致类的中结构体是只读的，无法去修改。而Swift允许您直接设置结构属性的子属性。

# 结构体的初始化
所有结构都有一个自动生成的成员智能初始化器，您可以使用它来初始化新结构实例的成员属性:
```
struct Resolution {
    var width = 0
    var height = 0
}

let resolution = Resolution(width:100,height:120)
print("width:\(resolution.width),height:\(resolution.height)")
结果:
width:100,height:120
```
而类并不具备这样的初始化器,如果需要特定的初始化需要去自定义,那么如何创建类似这样的初始化器呢?
首先要知道初始化器是系统本身的方法,我们只能去重写或者重载它
```
class VideoMode {
    var _name: String
    
    init(){
        _name = "Jin"
    }
}

let videoModeNormal = VideoMode()
print("name:\(videoModeNormal._name)")
结果:
name:Jin
```
我们重写了系统默认的init()初始化方法,并且设定我们想定义的功能。

那么我们如何去实现像结构体那样的初始功能呢?
这时候我们需要用到重载功能.
**重载就是将父类已有的方法通过扩充参数的形式重新定义一遍。**
我们可以看一下对比:
```
class VideoMode {
    var _name: String
    
    init(){
        _name = "Jin"
    }
    
    init(name:String){
        _name = name;
    }
    
}

let videoModeNormal = VideoMode()
print("name:\(videoModeNormal._name)")
let videoModeCustom = VideoMode(name: "Jack")
print("name:\(videoModeCustom._name)")
结果:
name:Jin
name:Jack
```

我们类中申明了两个初始化方法,通过我们在实例化的时候使用不同的初始化方式,系统会自动识别使用哪个方法。
例如调用`VideoMode()`的时候系统会调用init()方法,调用`VideoMode(name: "Jack")`的时候，系统会调用 `init(name:String)`方法。
当然,struct的初始化方法也是可以这样实现的:
```
struct Resolution {
    var width: Double
    
    init(){
        width = 100
    }
    
    init(width:Double){
        self.width = width;
    }
    
}

let resolutionNormal = Resolution()
print("width:\(resolutionNormal.width)")
let resolutionCustom = Resolution(width: 120)
print("width:\(resolutionCustom.width)")
结果:
width:100.0
width:120.0
```

> 调用init初始化方法,必须初始化结构体或者类中所有的属性。当然,如果该属性定义的时候已经初始化可以不用。
初始化可以参数下面的设定

```
    var width: Double = 0
    var height: Double = 0
    
    init(){
        width = 100
    }
```

如果你像下面这样写,会报出:Return from initializer without initializing all stored properties(提示你返回的时候,并没有将所有属性初始化)的错误

```
    var width: Double = 0
    var height: Double 
    
    init(){
        width = 100
    }
```

# 类的初始化
上面大致了解了结构体的初始化,还有一些类的初始化.虽然两者很多地方是相同.但总有一些不同的地方.
class 的设定中,初始化分为了指定初始化(Designated initializers)和便利初始化(Convenience initializers).
指定初始化就是我们上面讲的:

>init(`parameters`) {
    `statements`
}

而便利初始化是无法独立运作的。它设定是这样的:

>convenience init(`parameters`) {
    `statements`
}

我们可以是实验一下:
```
class Resolution {
    var width: Double = 0
    var height: Double = 0
    
    init(){
        width = 100
    }
    
   convenience init(height:Double){
        self.height = height;
    }  
}
```
会报错:
Use of 'self' in property access 'height' before self.init initializes self
Self.init isn't called on all paths before returning from initializer
这两个错误简单来说反应的就是这个init目前来说并不具备系统的init功能,它其实只是我们自定义的方法,但因为使用了init关键字,系统就无法理解这种行为了。 而要想让它发生作用,我们需要它本身加入init方法的功能。
```
   convenience init(height:Double){
        self.init()
        self.height = height;
    }  
```
这样就可以了。
而Swift本生也指定了这样的规则:
>* Rule 1
A designated initializer must call a designated initializer from its immediate superclass.
指定的初始化器必须从其直接的超类调用指定的初始化器。
>* Rule 2
A convenience initializer must call another initializer from the same class.
一个convenience修饰的初始化器必须从同一个类调用另一个初始化器。
>* Rule 3
A convenience initializer must ultimately call a designated initializer.
一个convenience修饰的初始化器必须最终调用一个指定的初始化器。
这些规则如下图所示:

![initializerDelegation.png](http://upload-images.jianshu.io/upload_images/1120896-1aff71fcf52922b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

在上图中，超类有一个指定的初始化器和两个方便的初始化器。一个方便初始化器调用另一个方便初始化器，它依次调用单个指定的初始化器。这满足了上面的规则2和3。超类本身没有进一步的超类，因此规则1不适用。

这个图中的子类有两个指定的初始化器和一个方便的初始化器。方便初始化器必须调用两个指定的初始化器中的一个，因为它只能从同一个类调用另一个初始化器。这满足了上面的规则2和3。两个指定的初始化器都必须从超类调用单个指定的初始化器，以满足上面的规则1。

>这些规则不会影响您的类的用户如何创建每个类的实例。上图中的任何初始化器都可以用来创建它们所属的类的完全初始化实例。这些规则只会影响您如何编写类的初始化器的实现。

下图显示了四个类的更复杂的类层次结构。它演示了这个层次结构中指定的初始化器如何充当类初始化的“漏斗”点，简化了链中的类之间的相互关系:

![initializerDelegation02.png](http://upload-images.jianshu.io/upload_images/1120896-15ca4ea02f16d07f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

看似复杂的关系,其实也很简单。
父类的init方法是会被继承的,而它的子类如果选择继承父类的初始化方法,而想创建convenience修饰的初始化器，就必然需要使用各自类中的init方法。

# 两阶段初始化
>Class initialization in Swift is a two-phase process. In the first phase, each stored property is assigned an initial value by the class that introduced it. Once the initial state for every stored property has been determined, the second phase begins, and each class is given the opportunity to customize its stored properties further before the new instance is considered ready for use.
Swift中的类初始化是一个两阶段过程。在第一阶段中，每个存储的属性都由引入它的类分配一个初始值。一旦确定了每个存储属性的初始状态，第二个阶段就开始了，并且每个类都有机会在新实例准备好使用之前进一步定制它的存储属性。

>The use of a two-phase initialization process makes initialization safe, while still giving complete flexibility to each class in a class hierarchy. Two-phase initialization prevents property values from being accessed before they are initialized, and prevents property values from being set to a different value by another initializer unexpectedly.
使用两阶段的初始化过程使初始化更加安全，同时仍然为类层次结构中的每个类提供完全的灵活性。两阶段初始化防止在初始化前访问属性值，并防止属性值被另一个初始化器设置为不同的值。

>Swift’s two-phase initialization process is similar to initialization in Objective-C. The main difference is that during phase 1, Objective-C assigns zero or null values (such as 0 or nil) to every property. Swift’s initialization flow is more flexible in that it lets you set custom initial values, and can cope with types for which 0 or nil is not a valid default value.
Swift的两阶段初始化过程类似于在Objective-C中初始化。主要的区别在于，在第1阶段，Objective-C中对每个属性都赋值0或null值(例如0或nil)。Swift的初始化流更灵活，它允许您设置自定义的初始值，并且可以处理0或nil不是有效的默认值的类型。

Swift的编译器执行四个有用的安全检查，以确保两阶段的初始化是没有错误的完成的:
>* Safety check 1
A designated initializer must ensure that all of the properties introduced by its class are initialized before it delegates up to a superclass initializer.
一个指定的初始化器必须确保它的类所引入的所有属性在它委托给一个超类初始化器之前被初始化。

>* As mentioned above, the memory for an object is only considered fully initialized once the initial state of all of its stored properties is known. In order for this rule to be satisfied, a designated initializer must make sure that all of its own properties are initialized before it hands off up the chain.
正如上面所提到的，只有当所有存储属性的初始状态被知道时，对象的内存才会被完全初始化。为了让这个规则得到满足，一个指定的初始化器必须确保所有的属性都在它被连接到链之前被初始化。

>* Safety check 2
A designated initializer must delegate up to a superclass initializer before assigning a value to an inherited property. If it doesn’t, the new value the designated initializer assigns will be overwritten by the superclass as part of its own initialization.
指定的初始化器必须在将一个值赋值给继承属性之前将其委托给一个超类初始化器。如果没有，指定初始化器分配的新值将被超类作为其自身初始化的一部分重写。

>* Safety check 3
A convenience initializer must delegate to another initializer before assigning a value to any property (including properties defined by the same class). If it doesn’t, the new value the convenience initializer assigns will be overwritten by its own class’s designated initializer.
一个便利的初始化器必须委托给另一个初始化器，然后将一个值赋给任何属性(包括由同一个类定义的属性)。如果它不这样做，那么方便初始化器分配的新值将被它自己的类的指定初始化器重写。

>
>* Safety check 4
An initializer cannot call any instance methods, read the values of any instance properties, or refer to self as a value until after the first phase of initialization is complete.
一个初始化器不能调用任何实例方法，读取任何实例属性的值，或者将self作为一个值，直到初始化的第一阶段完成。

>* The class instance is not fully valid until the first phase ends. Properties can only be accessed, and methods can only be called, once the class instance is known to be valid at the end of the first phase.
类实例在第一个阶段结束之前是不完全有效的。只有当类实例在第一阶段的末尾是有效的时，才可以访问属性，并且只能调用方法。

下面是第一阶段如何查找一个假设的子类和超类的初始化调用:
![第一阶段安全检查.png](http://upload-images.jianshu.io/upload_images/1120896-103ca2b2df88cc20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)


下面是两阶段的初始化是如何进行的，基于上面的四个安全检查:
阶段 1

>* A designated or convenience initializer is called on a class.
在类上调用指定的或方便的初始化器。

>* Memory for a new instance of that class is allocated. The memory is not yet initialized.
为该类的一个新实例的内存分配。内存还没有初始化。

>* A designated initializer for that class confirms that all stored properties introduced by that class have a value. Thememory for these stored properties is now initialized.
这个类的指定初始化器确认了该类所引入的所有存储属性都有一个值。这些存储属性的内存现在已经被初始化了。

> * The designated initializer hands off to a superclass initializer to perform the same task for its own stored properties.
指定的初始化器交给一个超类初始化器，为它自己的存储属性执行相同的任务。

>* This continues up the class inheritance chain until the top of the chain is reached.
这将继续继承类继承链，直到到达链的顶端。

>* Once the top of the chain is reached, and the final class in the chain has ensured that all of its stored properties have a value, the instance’s memory is considered to be fully initialized, and phase 1 is complete.
一旦到达链的顶端，并且链中的最后一个类确保其所有的存储属性都有一个值，实例的内存就被认为是完全初始化的，而阶段1是完整的。

>**这里的链就是`class.classProperty` 通过 `.`连接起来的结构**

阶段2

>* Working back down from the top of the chain, each designated initializer in the chain has the option to customize the instance further. Initializers are now able to access self and can modify its properties, call its instance methods, and so on.
从链的顶部向下工作，链中的每个指定初始化器都可以选择进一步自定义实例。初始化器现在能够访问self并可以修改它的属性，调用它的实例方法，等等。

>* Finally, any convenience initializers in the chain have the option to customize the instance and to work with self.
最后，链中的任何方便的初始化器都可以选择自定义实例并与self一起工作。

下面是第2阶段如何查找相同的初始化调用:
![第二阶段安全检查.png](http://upload-images.jianshu.io/upload_images/1120896-af426fd67630ee38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

官网给出的解释很是详尽的解释了初始化的工作步骤,虽然枯燥,但确实必须的。从这层工作中也看到了Swift相比于OC的安全和稳定性更加出色,比起OC更多时候因为意外导致的崩溃情况,Swift在这方面的检查机制要更加全面一点。

# 初始化器继承和重写
与Objective-C中的子类不同，Swift子类在默认情况下不会继承父类初始化器。

>Swift’s approach prevents a situation in which a simple initializer from a superclass is inherited by a more specialized subclass and is used to create a new instance of the subclass that is not fully or correctly initialized.
Swift的方法防止了一种情况，即一个超类的简单初始化器是由一个更专门化的子类继承而来的，它被用来创建一个未完全或正确初始化的子类的新实例。

当您编写一个与超类指定初始化器相匹配的子类初始化器时，您实际上对指定的初始化器进行重写。因此，您必须在子类的初始化器定义之前编写`override`修饰符。
```
class Vehicle {
    var numberOfWheels = 0
}

class Bicycle: Vehicle {
    override init() {
        super.init()
        numberOfWheels = 2
    }
}
```

# 自动初始化继承
As mentioned above, subclasses do not inherit their superclass initializers by default. However, superclass initializers are automatically inherited if certain conditions are met. In practice, this means that you do not need to write initializer overrides in many common scenarios, and can inherit your superclass initializers with minimal effort whenever it is safe to do so.
正如上面提到的，子类在默认情况下不会继承父类初始化器。但是，如果满足某些条件，超类初始化器就会自动继承。在实践中，这意味着您不需要在许多常见的场景中重写初始化器，并且可以在安全的情况下以最小的工作量来继承您的超类初始化器。

假设您为您在子类中引入的任何新属性提供了默认值，下面的两个规则适用:

>* 规则1
如果你的子类没有定义任何指定的初始化器，它会自动继承所有父类指定的初始化器。

>* 规则2
如果您的子类提供了所有超类指定的初始化器的实现，或者通过它们实现继承并遵守规则1，或者通过提供自定义实现作为其定义的一部分，那么它就会自动继承所有超类的方便初始化器。


# 总结
>* You can use both classes and structures to define custom data types to use as the building blocks of your program’s code.
您可以使用类和结构来定义定制的数据类型，以作为程序代码的构建块。

>* However, structure instances are always passed by value, and class instances are always passed by reference. This means that they are suited to different kinds of tasks. As you consider the data constructs and functionality that you need for a project, decide whether each data construct should be defined as a class or as a structure.
但是，结构实例总是通过值传递，而类实例总是通过引用传递。这意味着它们适用于不同类型的任务。当您考虑一个项目需要的数据结构和功能时，决定每个数据结构是否应该被定义为一个类或一个结构。

As a general guideline, consider creating a structure when one or more of these conditions apply:
指导方针，考虑在一个或多个条件适用的情况下创建一个结构:

>* The structure’s primary purpose is to encapsulate a few relatively simple data values.
结构体的主要目的是封装一些相对简单的数据值。

>* It is reasonable to expect that the encapsulated values will be copied rather than referenced when you assign or pass around an instance of that structure.
这是当您合理的分配或传递该结构的实例时，封装的值将被复制，而不是被引用。

>* Any properties stored by the structure are themselves value types, which would also be expected to be copied rather than referenced.
结构中存储的任何属性都是值类型，它也可以被复制，而不是被引用。

>* The structure does not need to inherit properties or behavior from another existing type.
结构体不需要从另一个现有类型继承属性或行为。

优秀的结构体例子包括:
>* The size of a geometric shape, perhaps encapsulating a width property and a height property, both of type Double.
几何形状的大小，可能是封装了宽度属性和高度属性，两者都是Double类型。

>* A way to refer to ranges within a series, perhaps encapsulating a start property and a length property, both of type Int.
一种表示范围内的范围的方法，可能是封装一个起始属性和一个长度属性，这两种类型都是Int类型。

>* A point in a 3D coordinate system, perhaps encapsulating x, y and z properties, each of type Double.
In all other cases, define a class, and create instances of that class to be managed and passed by reference. In practice, this means that most custom data constructs should be classes, not structures.
三维坐标系中的一个点，可能是封装x、y和z的属性，每一种类型都是双精度的。
在所有其他情况下，定义一个类，并创建该类的实例，以便通过引用来管理和传递。在实践中，这意味着大多数自定义数据结构应该是类，而不是结构。

>In Swift, many basic data types such as String, Array, and Dictionary are implemented as structures. This means that data such as strings, arrays, and dictionaries are copied when they are assigned to a new constant or variable, or when they are passed to a function or method.
在Swift中，许多基本的数据类型，如字符串、数组和字典都是作为结构实现的。这意味着当将字符串、数组和字典等数据分配给新的常量或变量时，或者当它们被传递给函数或方法时，就会复制这些数据。

>This behavior is different from Foundation: NSString, NSArray, and NSDictionary are implemented as classes, not structures. Strings, arrays, and dictionaries in Foundation are always assigned and passed around as a reference to an existing instance, rather than as a copy.
这种行为与Foundation库不同:NSString、NSArray和NSDictionary都是作为类实现的，而不是结构。Foundation中的字符串、数组和字典总是被分配和传递，作为对现有实例的引用，而不是作为副本。


