---
layout: post
title: "Swift中的字符串和字符"
date: 2018-01-03
excerpt: "Swift基础"
tags: [Swift]
post: true
comments: true
---
<small>
> Swift的字符串类型与Foundation的NSString类连接。Foundation还扩展了字符串来公开由NSString定义的方法。这意味着，如果您导入Foundation，您可以在字符串中访问这些NSString方法，而不需要进行强制转换。

####单行字符串
` let someString = "Some stirng" `
####多行文本
多行文本使用 `  """ 内容 """ `  没错，就是三个引号开头，三个引号结尾。
多行字符串文字包含了它的起始引号和结束引号之间的所有行

```swift

let lyric = """
那是你的眼神，明亮又美丽
啊啊啊~~~
"""
print(lyric)

结果:
那是你的眼神，明亮又美丽
啊啊啊~~~
```
好吧，这个我一开始以为只是在格式上可以写成多行，没想到结果就是多行，效果等同于\n,而且这样写法更加方便。
还有就是当我们想在多行文本中想控制某行不换行，可以选择在某行的末尾写上 \ 那么下一行就不会换行了
```swift 
let lyric = """
那是你的眼神，明亮又美丽\
啊啊啊~~~
"""

结果:
那是你的眼神，明亮又美丽啊啊啊~~~
```

在多行文本中，我们可以在结束引号的前面设定空格，来限定整段文本的缩进:
```swift
let lyric = """
  那是你的眼神，明亮又美丽
  啊啊啊~~~

  """
```
上面例子中,我们在最后"""前设定了2个空格,那么之前的文本必须都是缩进2个空格如果你这样写:
```swift
let lyric = """
那是你的眼神，明亮又美丽
啊啊啊~~~

  """
```
你会发现会报错
`Insufficient indentation of line in multi-line string literal 多行字符串文字的行缩不足`

##字符串中特殊字符
\\(反斜杠)
```
let test = "====\\===="
结果:
====\====
```

\t(水平选项卡)
```
let test = "\t1\t2\t3"
结果:
	1	2	3
```

\n(换行符)
```
let test = "\n1\n2\n3"
结果:

1
2
3
```

\r(回车)
```
let test = "\r1\r2\r3"
结果:

1
2
3
```

从结果来看,\r和\n似乎没有区别,但既然存在即表示肯定是有区别的
>'\r'是回车，\r 使光标到行首. '\n'是换行，\n 使光标下移一格,通常敲一个回车键，即是回车，又是换行（\r\n）。Unix中每行结尾只有“<换行>”，即“\n”；Windows中每行结尾是“<换行><回车>”，即“\n\r”；Mac中每行结尾是“<回车>”

一般遇到的情况是在解析Json字符串,网页数据,文件信息的时候会出现不同的应对.

\"(双引号)
 
```swift
let test = "\"认识自己的无知是认识世界的最可靠的方法。——《随笔集》\""
结果:
"认识自己的无知是认识世界的最可靠的方法。——《随笔集》"
```
\'(单引号)

```
let test = "\'曦\'"
结果:
'曦'
```
Unicode 字符就直接采用官网的了
```
let dollarSign = "\u{24}"        // $,  Unicode scalar U+0024
let blackHeart = "\u{2665}"      // ♥,  Unicode scalar U+2665
let sparklingHeart = "\u{1F496}" // 💖, Unicode scalar U+1F496

结果:
$ ♥ 💖
```

##空字符串
我们来看一下如果定义一个空字符串:
```
var emptyString = ""
或者
var emptyString = String()
```

判断字符串是否为空:
> isEmpty 方法

```swift
var emptyString = ""

if emptyString.isEmpty {
  print("emptyStirng is empty")
}

```

##字符串可变性
```
var variableString = "Horse"
variableString += " and carriage"

结果:
variableStrin:Horse and carriage
```

```
let constantString = "Highlander"
constantString += " and another Highlander"
发生错误。
Left side of mutating operator isn't mutable: 'constantString' is a 'let' constant
变异操作符的左侧是不可改变的:“常量字符串”是一个“let”常量
```
简单来说就是variableString 是变量,可以通过+操作符来追加字符串, constantString 为常量,不可修改。

这个我不得不吐槽一下,官网的这个解释,不就是常量不可修改嘛,说什么可变性。


##字符串是值类型
>If you create a new String value, that String value is copied when it’s passed to a function or method, or when it’s assigned to a constant or variable. In each case, a new copy of the existing String value is created, and the new copy is passed or assigned, not the original version.
如果您创建一个新的字符串值，那么当它传递给一个函数或方法时，或者当它被分配给一个常量或变量时,该字符串值将被复制。
在不同情况下，都会创建一个现有字符串值的新副本，并分配新副本，而不是原始版本。

>Swift’s copy-by-default String behavior ensures that when a function or method passes you a String value, it’s clear that you own that exact String value, regardless of where it came from. You can be confident that the string you are passed won’t be modified unless you modify it yourself.
Swift的 copy-by-default 行为可以确保当函数或方法传递给你一个字符串值时，不管它是从哪里来的，你都能清楚地知道你的值是什么。您可以确信，您所传递的字符串不会被修改，除非您自己修改它。

>Behind the scenes, Swift’s compiler optimizes string usage so that actual copying takes place only when absolutely necessary. This means you always get great performance when working with strings as value types.
在后台,Swift的编译器优化了字符串的使用，所以只有在绝对必要的情况下才会进行实际的复制。这意味着在使用字符串作为值类型时，您总是获得很好的性能。

这个就有别于OC了，OC中有明确的分为NSString(不可变)和NSMutableString(可变),swift中只有这么一种了，通过对此的优化更加确保了数据的安全。

##字符
定义一个字符(Character)类型的常量charA
```
let charA:Character = "A"
print("charA:\(charA)")
结果:
charA:A
```

您可以通过使用for循环遍历字符串来访问字符串的单个字符值:
```
let character = ""
let str = "窗外的麻雀"
for character in str {
    print(character)
}
结果:
窗
外
的
麻
雀
```

创建一个字符值数组:
```
let catCharacters: [Character] = ["C", "a", "t", "!", "🐱"]
print(catCharacters)
结果:
["C", "a", "t", "!", "🐱"]
```

可以通过`String(字符数组) `来将字符数组转换成字符串
```
let catCharacters: [Character] = ["C", "a", "t", "!", "🐱"]
let catString = String(catCharacters)
print(catString)
结果:
Cat!🐱
```

字符串的连接:
我们可以用 `+` 操作符连接字符串
```
let string1 = "hello"
let string2 = " there"
var welcome = string1 + string2
print("welcome \(welcome)")
结果:
welcome hello there
```

除了`+`操作符之外,还有append方法
```
let string1 = "hello"
let string2 = " there"
var welcome = string1 + string2

let exclamationMark: Character = "!"
welcome.append(exclamationMark)
print("welcome \(welcome)")
结果:
welcome hello there!
```
append是将字符串追加到原本字符串的末尾

以上的添加方法是通过连接两个字符串的值形成一个新的值
而在有的时候我们其实只需要临时的展示而已，不需要去改变原有的值,那么这个时候其实只需要这样做就可以了:
```
let badStart = """
one
two
"""
let end = """
three
"""
print(badStart + end)
print(badStart)

结果:
one
twothree

one
two
```

可以看到我们在输出print中将其相加,虽然显示了，但其实badStart的结果并没有改变。

###字符串的插值
在使用字符串的时候，我们可能需要利用其它的变量来改变我们目前的数据,而这个时候我们可以使用 `\(value)` 来将我们需要的值
```
let multiplier = 3
let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)"

结果:
3 times 2.5 is 7.5
```
当然了，这个也不是万能，它是不能将未转义的`\`直接放置于`()`中的:
错误的方式
```
let message = "\(\)"
print(message)

let message = "\(\')"
print(message)

```

正确的方式是这样的:
```
let message = "\("\'")"
print(message)
结果:
'
```

##计算字符的个数
计算字符的个数是我们比较常用的方法`count`
```
let unusualMenagerie = "Koala 🐨, Snail 🐌, Penguin 🐧, Dromedary 🐪"
print("unusualMenagerie has \(unusualMenagerie.count) characters")
结果:
unusualMenagerie has 40 characters
```

当然了,count并不总是能够计算清楚我们的字符个数:
```
var word = "cafe"
print("the number of characters in \(word) is \(word.count)")
这个计算结果是4
word += "\u{301}"  
print("the number of characters in \(word) is \(word.count)")
当我们计算的会发现结果仍然是4
```
这是为什么呢?
因为swift中的字符串运用了`扩展字元簇（Extended Grapheme Clusters)` 

扩展字元簇可以由多个Unicode标量组成。
**这意味着不同的字符和相同字符的不同表示可以要求不同的内存数量来存储。
因此，Swift中的字符不会在字符串的表示中占用相同数量的内存。**
结果，字符串中的字符数无法计算.
如果您使用的是特别长的字符串值，请注意，count属性必须遍历整个字符串中的Unicode标量，以便确定该字符串的字符。

比如：字母é可以是一个单独的Unicode scalar：U+00E9，也可以是多个纯量的组合：U+0065U+0301 （其中U+0065就是字母e）。在Swift中，这两种情况都认为是一个字符，因此获取字符串长度的时候(用全局函数count())，返回的值是相同的，这意味着字符串的改变并不一定意味着其长度发生变化。

count属性返回的字符数并不总是与包含相同字符的NSString的长度属性相同。
NSString的length是基于字符串的utf-16表示的16位代码单元的数量，而不是字符串中Unicode扩展的grapheme集群的数量

##字符串索引
```
let greeting = "Guten Tag!"
//字符串中的开头字母
print(greeting[greeting.startIndex])
//index()表示获取字符串中某个下标的元素,before意味着获取最后一位的之前的下标
//endIndex表示字符串最后一位
print(greeting[greeting.index(before: greeting.endIndex)])
//after表示获取某个下标之后
print(greeting[greeting.index(after: greeting.startIndex)])
//这个方法表示以startIndex下标开始，偏移7位
let index = greeting.index(greeting.startIndex, offsetBy: 7)
print(greeting[index])
结果:
G
!
u
a
```

使用索引属性来访问字符串中各个字符的所有索引:
```
for index in greeting.indices {
    print("\(greeting[index]) ", terminator: "")
}
结果:
G u t e n   T a g ! 
```
这里的terminator是print方法中参数,默认是添加换行符\n
```
let greeting = "Guten Tag!"

for index in greeting.indices {
    print("\(greeting[index]) ", terminator: "\n")
}
G 
u 
t 
e 
n 
  
T 
a 
g 
! 
```
感觉跟print没有区别吧。
```
let greeting = "Guten Tag!"

for index in greeting.indices {
    print("\(greeting[index]) ", terminator: "1")
}
结果:
G 1u 1t 1e 1n 1  1T 1a 1g 1! 1
```
这个其实就是在每次获取单个字符之后添加一个字符。


##字符串的插入和删除

####插入 
>insert(value,at:index)

```
var welcome = "hello"
welcome.insert("!", at:welcome.endIndex)
print(welcome)
结果:
hello!

var welcome = "hello!"
welcome.insert(contentsOf: " world", at: welcome.index(before: welcome.endIndex))
print(welcome)
结果:
hello world!
```

####移除 
>remove(at:index)
根据下标进行移除

>removeSubrange
根据提供的范围进行移除

```
var welcome = "hello!"
welcome.remove(at: welcome.index(before: welcome.endIndex))
print(welcome)
结果:
hello

var welcome = "hello world"
//空格的位置 - 末尾
let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
welcome.removeSubrange(range)
print(welcome)
结果:
hello 
```
后续更新,敬请期待...
</small>


