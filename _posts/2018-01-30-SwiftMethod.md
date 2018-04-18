---
layout: post
title:  "Swift - 函数"
date:   2018-01-30
categories: Swift
---

>是的，函数。
总的来说方法和函数并没有什么区别。因为本质相同，都是为了处理独立的工作而存在的。而要说有区别，在以前，函数是面向过程中的概念，方法是面向对象的概念。因为没有类的概念,函数可以直接定义,直接使用。而方法必须在类当中,依托于对象才能执行。
而Swift的统一函数语法足够灵活，可以表达任何东西，从简单的c风格函数，没有参数名，到复杂的objective-c风格的方法，每个参数都有名称和参数标签,既可以独立存在使用,也可以依托于对象,更加灵活和强大。

# 函数的定义:

func `function Name`(`param Name`:`param Type`,...) -> `return Type`{
`function Body`
 return `return Value`;
}

学习代公式就简单多了。
我们写一个有返回值带参数的函数:
```
func greet(person: String) -> String {
    let greeting = "Hello, " + person + "!"
    return greeting
}

let name = greet(person: "Jack")

print(name)
结果:
Hello, Jack
```
我们再写一个没有返回值没有参数的函数:
```
func greet() {
    print("nothing")
}
greet()
```
就跟组装和拆分玩具一样。


# 函数参数和返回值
>Function parameters and return values are extremely flexible in Swift. You can define anything from a simple utility function with a single unnamed parameter to a complex function with expressive parameter names and different parameter options.
函数参数和返回值在Swift中非常灵活。您可以定义任何东西，从一个简单的实用函数，到一个不知名的参数，再到具有表达性参数名称和不同参数选项的复杂函数。

>Swift中的参数是常量,无法直接在函数中修改。

>Functions can have multiple input parameters, which are written within the function’s parentheses, separated by commas.函数可以有多个输入参数，这些参数是在函数的圆括号中编写的，用逗号分隔。

```
func greet(person: String, alreadyGreeted: Bool) -> String {
    let paramValue = "person:"+person+",alreadyGreeted:"+alreadyGreeted.description;
    return paramValue;
}
print(greet(person: "Tim", alreadyGreeted: true))
```

官方文档中有这样一种写法:
```
func printAndCount(string: String) -> Int {
    print(string)
    return string.count
}
func printWithoutCounting(string: String) {
    let _ = printAndCount(string: string)
}
printAndCount(string: "hello, world")
printWithoutCounting(string: "hello, world")
结果:
hello, world
hello, world
```
简单来说就是在函数中调用其它的函数。在printWithoutCounting函数中,调用printAndCount函数，但却使用缺省值得方式忽略了printAndCount的返回值。这倒是个不错的忽略警告的函数(😀)。


接下来开始尝试更加复杂的操作,例如下面的🌰是在一组值中获取最大值和最小值
```
func minMax(array: [Int]) -> (min: Int, max: Int) {
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}
let result = minMax(array: [34,53,645,23,12,4,2])
print("minValue:\(result.min)")
print("maxValue:\(result.max)")
结果:
minValue:2
maxValue:645
```
从上述例子中可以引出一个问题,如果返回值为nil怎么办?在这个json数据频繁交互的世界里,有时候数据异常还是比较常见的。而Swift对于数据安全还是很重视的,所以我们将程序修改一下.
```
func minMax(array: [Int]) -> (min: Int, max: Int)? {
    if array.isEmpty { return nil }

    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}

let array = Array<Int>();
let result = minMax(array: array)
print("min is \(result?.min ?? 0) and max is \(result?.max ?? 0)")
```
可以看到我们在minMax函数中添加了对空数组的判断,而这个时候我们就需要用到Swift中的可选类型(Optional?)。
当我们将返回值`(min: Int, max: Int)`设定为`(min: Int, max: Int)?`的时候,程序本身就会对我们提出很多要求的。如果代码不明确，那么是通不过编译的。例如我们在输出的时候原本的`result.min`并没有考虑值为空的情况,最后需要改成`result?.min ?? 0`才可以。


# 函数参数标签和参数名

>Each function parameter has both an argument label and a parameter name. The argument label is used when calling the function; each argument is written in the function call with its argument label before it. The parameter name is used in the implementation of the function. By default, parameters use their parameter name as their argument label.
每个函数参数都有一个参数标签和一个参数名。参数标签是在调用函数时使用的;每个参数都在函数调用中使用它的参数标签。在函数的实现中使用了参数名。默认情况下，参数使用参数名作为参数标签。

公式:
func `function Name`(`param label Name` `param Name`:`param Type`,...) -> `return Type`{
`function Body`
 return `return Value`;
}

这也算是Swift的特性了,不知道是哪个倒霉孩子写的。
正常写的时候是这样的
```
func someFunction(firstParameterName: Int, secondParameterName: Int) {
    print("firstParameterName:"+String(firstParameterName))
    print("secondParameterName:"+String(secondParameterName))
}
someFunction(firstParameterName: 1, secondParameterName: 2)
```
有标签的时候是这样的:
```
func greet(person: String, from hometown: String) -> String {
    return "Hello \(person)!  Glad you could visit from \(hometown)."
}
print(greet(person: "Bill", from: "Cupertino"))
结果:
Hello Bill!  Glad you could visit from Cupertino.
```
可以看到程序中from是作为调用函数时的参数名称显示的。而hometown也是可以作为参数名称在函数内部使用的。
那大家是不是有疑惑from能不能在函数内部使用？我帮你们试过了，不能使用。也就是说:**标签名不能在函数内部使用** 
如果非要我来理解这样设计有啥用的话,可能就是为了安全吧。（即使你得到了我的人,但却得不到我的心的赶脚。）

# 省略标签名称
不得不说 `_ ` 操作符真是想哪哪都可以省，省出心飞扬。我们还可以借此省略标签。
```
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
    print("firstParameterName:"+String(firstParameterName))
    print("secondParameterName:"+String(secondParameterName))
}
someFunction(1, secondParameterName: 2)
```

这样我们可以在调用的时候就不用写标签名称了。虽然这样可读性不高,毕竟设计标签的时候就是为了可读性。真是上有政策下有对策，这个对策还是上头给的。

# 默认参数值
通过在函数中直接给参数赋值,我们就可以实现参数的默认值。在调用的时候，如果没有传递参数,就可以使用默认值进行操作。这无形给那些经常会遗忘的程序员小哥哥,或者是有默认值情况存在的程序提供了很方便的设计。
让我们来看看具体的🌰：
```
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
    print("firstParameterName:"+String(firstParameterName))
    print("secondParameterName:"+String(secondParameterName))
}
someFunction(parameterWithoutDefault: 3, parameterWithDefault: 6) // parameterWithDefault is 6
someFunction(parameterWithoutDefault: 4) // parameterWithDefault is 12
结果:
firstParameterName:3
secondParameterName:6
firstParameterName:4
secondParameterName:12
```

# 可变参数

> variadic parameter accepts zero or more values of a specified type. You use a variadic parameter to specify that the parameter can be passed a varying number of input values when the function is called. Write variadic parameters by inserting three period characters (...) after the parameter’s type name.
可变参数接受指定类型的0或多个值。您可以使用可变参数来指定参数在调用函数时可以传递不同数量的输入值。通过在参数的类型名称后面插入三个周期字符(`...`)来编写可变参数。

简单来说,你使用`...` 就可以在填写参数的时候随便写多少参数了。妈妈再也不用担心我设计参数的时候太长了。
这里强调的是类型是需要指定的。不过有个Any类型，你也不用怕类型太限制你的想象。
具体的写法可以看下面的🌰:
```
func arithmeticMean(numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
print(arithmeticMean(numbers: 1, 2, 3, 4, 5))
print(arithmeticMean(numbers: 3, 8.25, 18.75))
```

# In - Out 参数
>Function parameters are constants by default. Trying to change the value of a function parameter from within the body of that function results in a compile-time error. This means that you can’t change the value of a parameter by mistake. If you want a function to modify a parameter’s value, and you want those changes to persist after the function call has ended, define that parameter as an in-out parameter instead.
默认情况下，函数参数是常量。试图从该函数的主体内部更改函数参数的值会导致编译时错误。这意味着您不能错误地更改一个参数的值。如果您想要一个函数来修改一个参数的值，并且您希望在函数调用结束后这些更改继续存在，那么将该参数定义为in-out参数。

>You write an in-out parameter by placing the inout keyword right before a parameter’s type. An in-out parameter has a value that is passed in to the function, is modified by the function, and is passed back out of the function to replace the original value.
通过在参数类型之前放置inout关键字来编写in-out参数。in-out参数具有传递给函数的值，由函数进行修改，并从函数中返回，以替换原来的值

>You can only pass a variable as the argument for an in-out parameter. You cannot pass a constant or a literal value as the argument, because constants and literals cannot be modified. You place an ampersand (&) directly before a variable’s name when you pass it as an argument to an in-out parameter, to indicate that it can be modified by the function.
您只能传递一个变量作为in-out参数的参数。不能将常数或文字值作为参数传递，因为常量和文本不能被修改。当将变量名作为参数传递给in-out参数时，您可以直接在变量名前面放置一个&(&)，以表明它可以通过函数进行修改。

官网讲的很清楚了,我们在定义函数参数的时候,参数是常量,那么有些情况可能需要改动参数,就需要用`inout` 参数（虽然我感觉重新创建临时变量来获取参数值,替代它进行操作会更好,能够设定不可变的我们尽量不去改变它,可变的操作很容易造成代码编写的散漫性.）

下面我们举个🌰:
```
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
```
这是一个简单的两个数交换的函数.里面的参数a和b都直接参与了计算,如果将inout去除,你会发现程序会提示`Cannot assign to value: 'a' is a 'let' constant 不能为a赋值,a 是 let'常量`

# 方法类型

>Every function has a specific function type, made up of the parameter types and the return type of the function.
每个函数都有一个特定的函数类型，由参数类型和函数的返回类型组成。

这个概念其实是一直有的。在C或者OC我们可以定义函数指针来获取函数的地址,而我们将该地址赋值给别的函数的时候,就必须保证结构相同,其实也就是数据类型一致了。

那么如何在Swift中实现呢？
```
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}

var mathFunction: (Int, Int) -> Int = addTwoInts

print(mathFunction(1,2))
```
通过上面的🌰,应该就知道了吧。设定了函数addTwoInts,类型是`(Int,Int)->Int`,然后直接创建一个同等类型的函数接收它的地址。我们可以通过函数名称直接获取函数相应的地址和数据类型等对应的数据。

# 方法类型作为参数类型
在Swift函数中,参数类型也可以说是很灵活了,我们还可以将方法类型直接设定为参数类型。
```
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}

func printMathResult(mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
    print("Result: \(mathFunction(a, b))")
}

printMathResult(mathFunction: addTwoInts, 3, 5)
结果:
Result: 8
```
上面的参数mathFunction接收了一个函数addTwoInts.然后直接在函数体中使用了该函数的功能.

# 嵌套函数
嵌套函数并不算是Swift中独有的,其他的语言中也有很多解决方案。不过这里的嵌套函数,比较直接明了。直接在函数体中定义一个函数来使用,算是一个临时变量,并不能在外界直接被调用。这种设计的好处就是在于自用的函数可以归类在一起,不至于像其他语言那样,散乱的放置在文件当中。

```

func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}
var currentValue = -4
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)

while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}

print("zero!")
```


