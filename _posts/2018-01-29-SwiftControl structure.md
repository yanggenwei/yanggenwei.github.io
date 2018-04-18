---
layout: post
title:  "Swift - 控制结构"
date:   2018-01-29
categories: Swift
---

>Swift中的循环语句for,while,dowhile,还有分支switch相比于OC也有很多的改进和变化。这里主要是使用方式了，我们可以快速的过一下。

# for
遍历数组:
```
let names = ["Anna","Alex","Brian","Jack"];

for name in names{
    print("Hello,\(name)")
}

结果:
Hello,Anna
Hello,Alex
Hello,Brian
Hello,Jack

```
遍历字典:
```
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]

for (animalName, legCount) in numberOfLegs {
    print("\(animalName)s have \(legCount) legs")
}
结果:
ants have 6 legs
spiders have 8 legs
cats have 4 legs

```
实现范围循环
```
for index in 1...5 {
    print("\(index)", terminator:" ")
}
结果:
1 2 3 4 5
```
>* 在上面的例子中，index是一个常量，它的值在循环的每次迭代开始时自动设置。因此，索引在使用之前不需要声明。它是通过在循环声明中包含的，而不需要一个let声明关键字来声明的。
> * (...) 可以称之为封闭操作符(closed range operator),这里索引的值被设置为range(1)中的第一个数字，并执行循环中的语句,

>如果您不需要一个序列中的每个值，那么可以使用一个下划线代替一个变量名来忽略这些值。

```
let base = 3
let power = 10
var answer = 1
for _ in 1...power {
    answer *= base
}
print("\(base) to the power of \(power) is \(answer)")

结果:
3 to the power of 10 is 59049
```
简而言之就是 你需要的是一个能够循环x次数的结构,利用它的循环功能做一些事情。而不需要设定一个值来获取它每次循环的值得。这个情况我们在遍历字典的时候也可以用到.
```
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]

for (animalName, _) in numberOfLegs {
    print("animalName: \(animalName)")
}
结果:
animalName: ant
animalName: spider
animalName: cat
```
我们只用到它的key或者value,那么就没有必要去再设定一个值了，毕竟自动生成一个临时变量也有开销，虽然很小。

之前提到了[Range Operators](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/BasicOperators.html#//apple_ref/doc/uid/TP40014097-CH6-ID73). 它除了之前提到的方式,还可以这样:
```
let minutes = 60
for tickMark in 0..<minutes {
    print("\(tickMark)", terminator: " ")
}

结果:
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 
```
其实从这一点来说,通过 **range operators** for in 循环实现将for in与for循环的功能结合起来了,不得不说这样的方式还是很棒的。
当然了,光这样还不能提现出与for循环的结合之后的强大,毕竟for循环可以通过设定参数i,实现i++,i+=5,随心所欲的控制,所以Swift中也设置了这样的方式:
```
let minuteInterval = 5
let minutes = 60
for tickMark in stride(from: 0, to: minutes, by: minuteInterval) {
    print(tickMark, terminator: " ")
}
结果:
0 5 10 15 20 25 30 35 40 45 50 55
```
## stride(from:to:by:) 
Swift的方法定义还是很有好的,只要会点英文还是能看得懂的,from:开始的数值, to:结束的数值,by:每次循环的步数

除了以上方法之外,还有另一个方法:

## stride(from:through:by:) 
```
let minuteInterval = 5
let minutes = 15
for tickMark in stride(from: 0, to: minutes, by: minuteInterval) {
    print(tickMark, terminator: " ")
}
print("")
for tickMark in stride(from: 0, through: minutes, by: minuteInterval) {
    print(tickMark, terminator: " ")
}
结果:
0 5 10 
0 5 10 15
```
那么这个方法是干什么的呢? 从结果来说,我们应该已经明白了,其实就是 stride(from:to:by:) 设定的结果范围我们是不包括的,而stride(from:through:by:) 中参数through设定的结果值范围是包括的。

# While
while与OC中并没有什么区别,但是却多出了一个repeat-while
```
repeat {
    statements
} while condition
```
其实吧,这个跟do-while没啥区别,因为do关键字在Swift被设定为捕捉异常了。


# Switch
switch相比于OC,或者说绝大多数语言来说都要强大了,因为switch-case分支支持的类型更加多了.
曾经只有整型,字符串,字符才可以，如今还加入了很多不一样的组合:

```
let anotherCharacter: Character = "a"

switch anotherCharacter {
case "a", "A":
    print("The letter A")
default:
    print("Not the letter A")
}
结果:
The letter A
```
在判断的时候,我们将类似集合的方式写入设定在case分支上了,这样就避免了冗长的判断,还有更加灵活的写法
```
let approximateCount = 62
let countedThings = "moons orbiting Saturn"
let naturalCount: String
switch approximateCount {
case 0:
    naturalCount = "no"
case 1..<5:
    naturalCount = "a few"
case 5..<12:
    naturalCount = "several"
case 12..<100:
    naturalCount = "dozens of"
case 100..<1000:
    naturalCount = "hundreds of"
default:
    naturalCount = "many"
}
print("There are \(naturalCount) \(countedThings).")
结果:
There are dozens of moons orbiting Saturn.
```
可以是范围,也可以是数组:
```
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("\(somePoint) is at the origin")
case (_, 0):
    print("\(somePoint) is on the x-axis")
case (0, _):
    print("\(somePoint) is on the y-axis")
case (-2...2, -2...2):
    print("\(somePoint) is inside the box")
default:
    print("\(somePoint) is outside of the box")
}
结果:
(1, 1) is inside the box
```
##  _ 通配符模式，以匹配任何可能的值。

switch 还设定了一种**值绑定(Value Bindings)**的方式来匹配case分支
```
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
结果:
on the x-axis with an x value of 2

let stillAnotherPoint = (9, 0)
switch stillAnotherPoint {
case (let distance, 0), (0, let distance):
    print("On an axis, \(distance) from the origin")
default:
    print("Not on an axis")
}
结果:
On an axis, 9 from the origin
```
那么什么是**值绑定**呢？

>A switch case can name the value or values it matches to temporary constants or variables, for use in the body of the case. This behavior is known as value binding, because the values are bound to temporary constants or variables within the case’s body.
switch case可以将它匹配的值或值命名为临时常量或变量，以便在案例的主体中使用。这种行为被称为值绑定，因为值被绑定到在case的主体内的临时常量或变量。

拿上述例子来说,我们设置了(2,0),于之匹配的case是(let x,0),这时候，就会将2赋值给常量x,那么在这个case分支中，我们就可以使用该常量x进行一些操作了。简单来说就是将2与常量x进行了绑定。

除了以上设定之外,switch还可以通过**where**关键来设定case分支:
```
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
结果:
(1, -1) is on the line x == -y
```
where 关键字用于条件判断,let (x, y) where x == y 例如此处的运算时如果x == y 那么执行 let (x, y)


# Control Transfer Statements
* continue
* break
* return
* throw
* fallthrough

前四个都是比较常见的

## continue是忽略循环中的单次执行

```
for index in 1...5{
    if index == 2{
        continue
    }
    print("\(index)", terminator:" ")
}

结果:
1 3 4 5
```
## break 
结束当前循环,也就是说，如果是多重循环嵌套,结束的只是其中一个循环而已,并不会结束所有循环

```
for index in 1...5{
    if index == 2{
        break
    }
    print("\(index)", terminator:" ")
}
结果:
1
```

## return
return 如果方法中具有返回值,需要再return后面设定返回值,如果没有返回值,那么其实返回的是nil. 也就是return是作用在方法中的，所以它会直接结束,不会再继续运行接下来的代码了。具体在方法中演示。

## throw
 是当发生错误的时候,手动的在相应的位置抛出异常。具体的我们再异常中演示

##  fallthrough
这个其实就比较有意思了,因为可以看出Swift在设计时的用心程度。为什么这么说呢?

我们曾经是这样写的:
```
switch (result){
  case 1:
     XXX
    break;
  case 2:
     XXX
   case3:
     XXX
    break;
default:
    break;
}
```
每个case分支需要利用break来分割成个体,要想顺序执行需要将break去掉，像case2,case3那样.而swift中默认是不需要break,直接就是个体执行。
```
switch (result){
  case 1:
     XXX
  case 2:
     XXX
  case 3:
     XXX
default:
    break;
}
```
那这样的话是不是感觉就不好顺序执行了?当然不是了,这时候**fallthrough**的作用就出来了:
```
let integerToDescribe = 5

switch integerToDescribe {
case 5:
     print("integerToDescribe:\(integerToDescribe)")
    fallthrough
case 1:
     print("1")
default:
     print("other")
}
结果:
integerToDescribe:5
1
```
利用fallthrough我们就可以让两个case联系起来了。
这个就是简单将多余的需求去除,变得更加贴切我们的实际使用。正常使用中将case联合起来的情况就跟你在大街上遇到刘亦菲跟你说hello一样困难。所以为啥我们每次都要多写那该死的break呢,用到加个fallthrough不就好了嘛。

## Labeled Statements
翻译过来是标签声明,这个是什么意思呢？官网的比较晦涩难懂,我们用实践来初步了解一下:
首先它是这样一个结构:
`label name`: while `condition` {
    `statements`
}

那么这个标签有啥特殊的地方呢?
```
var index = 10

while index < 20 {
    switch index {
    case 11:
        print("11")
        break
    case 12:
        print("12")
    default:
        break
    }
    index+=1
}

结果:
11
12

```
我们改一下程序,将标签gameloop设置到程序当中:
```
var index = 10

gameloop: while index < 20 {
    switch index {
    case 11:
        print("11")
        break gameloop
    case 12:
        print("12")
    default:
        break
    }
    index+=1
}
结果:
11
```
通过结果还有官网的说明来看:

>* In Swift, you can nest loops and conditional statements inside other loops and conditional statements to create complex control flow structures. However, loops and conditional statements can both use the break statement to end their execution prematurely. Therefore, it is sometimes useful to be explicit about which loop or conditional statement you want a break statement to terminate. Similarly, if you have multiple nested loops, it can be useful to be explicit about which loop the continue statement should affect.
>* To achieve these aims, you can mark a loop statement or conditional statement with a statement label. With a conditional statement, you can use a statement label with the break statement to end the execution of the labeled statement. With a loop statement, you can use a statement label with the break or continue statement to end or continue the execution of the labeled statement.
>* 在Swift中，您可以在其他循环和条件语句中嵌套循环和条件语句，以创建复杂的控制流结构。但是，循环和条件语句都可以使用break语句来提前结束它们的执行。因此，有时需要显式地明确哪些循环或条件语句，您想要一个break语句终止。类似地，如果您有多个嵌套循环，那么可以很清楚地说明continue语句应该影响哪些循环。
>* 为了实现这些目标，您可以用一个语句标签来标记一个循环语句或条件语句。使用条件语句，您可以使用break语句的语句标签来结束标记语句的执行。使用循环语句，您可以使用带有中断或continue语句的语句标签来结束或继续执行标记的语句。

简单来说就是标签运用于多中控制结构的情况,可以清晰的知道我们在控制结构中设置的某些控制节点的存在。算是比较灵活。

最后的是检查版本或者API情况
if #available(`platform name` `version`,`...`, *) {
    `statements to execute if the APIs are available`
} else {
  `fallback statements to execute if the APIs are unavailable`
}

以往我们在判断版本的时候需要用到宏定义去判断,现在嘛，有现成的判断方法了
```
if #available(iOS 10, macOS 10.12, *) {
    // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
} else {
    // Fall back to earlier iOS and macOS APIs
}
```







