---
layout: post
title:  "Swift - 数组和集合"
date:   2018-01-13
categories: Swift
---

在Swift中仍然保留了OC中的三种集合类型:
Array:数组是有序的值集合。
Set:集合是惟一值的无序集合。
Dictionary:字典是键值关联的无序集合。
![CollectionTypes.png](http://upload-images.jianshu.io/upload_images/1120896-0bda4fc9895f9fc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

>Arrays, sets, and dictionaries in Swift are always clear about the types of values and keys that they can store. This means that you cannot insert a value of the wrong type into a collection by mistake. It also means you can be confident about the type of values you will retrieve from a collection.
在Swift中，数组、集合和字典的值得数据类型很明确。这意味着您不能将错误类型的值插入到集合中。
它还意味着您可以对从集合中检索到的值类型有信心。

>Swift’s array, set, and dictionary types are implemented as generic collections.
Swift的数组、集合和字典类型被实现为泛型集合。

>If you create an array, a set, or a dictionary, and assign it to a variable, the collection that is created will be mutable. This means that you can change (or mutate) the collection after it’s created by adding, removing, or changing items in the collection. If you assign an array, a set, or a dictionary to a constant, that collection is immutable, and its size and contents cannot be changed.
如果您创建一个数组、一个集合或一个字典，并将其赋值给一个变量，那么所创建的集合将是可变的。这意味着您可以通过在集合中添加、删除或更改项目来更改(或更改)集合。如果您将数组、集合或字典赋值给常量，那么该集合是不可变的，它的大小和内容不能被更改。

看到了官网对于Swift中集合的各种特性解释，接下来我们进入实践阶段,来验证它是否是这样的。
# Arrays
## Arrays的初始化
在创建之初,我们首先要想好,是需要可变的还是不可变的。
一般来说我们使用不可变的会增加我们程序的安全性,避免了我们在编写程序时无意中添加或者删除了某值造成不好的效果,但系统却没有给出相应的提示。

```
let someNumber = [3,42,5]
print(someNumber)
结果:
[3, 42, 5]
```
在这一段程序中,我们直接将3,42,5三个int类型的值加入到了someNumber数组中去了。
在这过程中swift还需要动态的去判断值的类型。
所以我们也可以直接指定它的数据类型:
```
let someNumber:[Int] = [3,42,5]
```
这样的一个好处就是我们可以很容易的去识别一个数组当中值得类型是否正确:
```
let someNumber:[Int] = [3,"Str",5]
```
当你这样写的时候,编译就会发生错误,因为"Str"是String类型并不是Int类型,程序就会及时的给予你提示。

然而上面的方式其实是一种简写的方式,完整的写法应该如下所示:
```
let someInts = [Int](arrayLiteral: 3,3,5)
print(someInts)

let arrayInts = Array(arrayLiteral: "3","3","3")
print(arrayInts)
```


我们再看其他的初始化方式:
创建一个空的数组
```
var someInts = [Int]()
```
这里要注意的就是[Int]它的数据类型不能省.不然会报错。
因为Swift中严格的数据类型控制。可以设想,这里没有数据类型控制的话,那么我们在之后的追加元素的操作中,
追加了不同的数据,而我们之前又没有显示的去设定它的数据类型，就会造成数据类型不可控的情况,这个在Swift并不被允许。
你可以通过append去追加元素(当然了,首先你的数组是个变量,而不是常量):
```
someInts.append(3)
```
同时,如果你已经初始化完毕,那么你可以直接设定你的数组为空:
```
someInts = []
```
你可能会有疑问,为什么一开始的时候不能这样设置为空呢？
因为在这里虽然你设为了空,someInts的数据类型还是Int,你仍然是无法去添加其他类型的值的。
```
var someInts = [Int]()
someInts.append(3)
someInts = []
someInts.append("string")  //error:Cannot convert value of type 'String' to expected argument type 'Int'
print(someInts)
```

我们再看看其它的初始化方式:
```
let threeDoubles = Array(repeatElement(0.0, count: 3))
print(threeDoubles)
结果:
[0.0, 0.0, 0.0]
```
其实从参数名称上我们就可以知道,我们创建了3个重复值用于数组的初始化。
repeatElement方法中可以填写两个参数,一个是需要重复的数据,count表示重复多少次。
我想,这是为了方便我们在特定情况下的初始化吧。

嗯,继续看看别的初始化方式:
我们在使用的时候,默认的数据的类型的确是确保了我们的数据类型安全,但同时也限制了我们编程的灵活性。
在很多实际的运用场景中,一个数组集合中包含的并不可能只有一种数据类型的值,如果每一种都需要定义个数组来进行操作的话,也会增加开发成本。当然咯，这个还是根据具体的环境来决定成本的舍取性的。
那么如何去定义一个什么类型都可以存的数组呢？
```
let anyData = ["String",34,53] as [Any]
print(anyData)
结果:
["String", 34, 53]
```
通过结果我们可以看到,这样定义的数组可以容纳其他类型的值。


初始化我们大概了解了之后,我们还需要了解它的内置方法:

# Array的内置方法
## first 获取第一个元素
```
var shopping_list = ["A","B","C","D"]
print(shopping_list.first)
结果:
"A"
```

## first where 根据判断条件返回结果
```
var shopping_list = [53,4,2,45]
let result = shopping_list.first(where: { (intResult) -> Bool in
    return intResult==54
})
print(result ?? 0)
```
这里的意思是如果第一个元素是54,那么返回正确结果,如果不是的话本来是应该返回nil的，但是我们给它做了判断,如果为nil,那么结果为0。

## last 获取最后一个元素
```
var shopping_list = ["A","B","C","D"]
print(shopping_list.last)
结果:
"D"
```

## count 数量  

```
let shopping_list = [3,4,2,3]
print("The shopping_list list contains \(shopping_list.count) items.")
结果:
The shopping_list list contains 4 items.
```
## isEmpty 判空 

```
let shopping_list = [3,4,2,3]

if shopping_list.isEmpty {
    print("The shopping_list list is empty.")
} else {
    print("The shopping_list list is not empty.")
}
结果:
The shopping_list list is not empty.
```

## contains 判断数组中是否包含某个元素

```
var shopping_list = ["A","B","C","D"]
print(shopping_list.contains("C"))
结果:
true
```

## append 追加 

```
var shopping_list = [1]
shopping_list.append(4)
print(shopping_list)
结果:
[1,4]
```

## insert(newElement:  at: ) 插入单个元素  
将新元素插入到指定的位置,但不能插入到数组最后一位的下一位。

```
var shopping_list = [1]
shopping_list .insert(3, at: 1)
print(shopping_list)
结果:
[1,3]
```

## insert(contentsOf:at:) 插入整个集合 

```
var shopping_list = [1,2]
var otherShopping_list = [3,4,5]
shopping_list.insert(contentsOf: otherShopping_list, at: 2)
print(shopping_list)
结果:
[1,2,3,4,5]
```

##  replaceSubrange(Range<Int>, with: Collection) 将某个范围的元素替换为指定的集合
```
var shopping_list = ["A","B","C","D"]
shopping_list.replaceSubrange(0...1,with:["F","E"])
print(shopping_list)
结果:
 ["F","E","C","D"]
```

## remove(at:) 移除某个下标的元素 
```
var shopping_list = [1,2,3,4]
shopping_list.remove(at: 0)
print(shopping_list)
结果:
[2,3,4]
```

## removeFirst(count) 从开始位置开始移除指定数量的元素 

```
var shopping_list = [1,2,3,4]
shopping_list.removeFirst(2)
print(shopping_list)
结果:
[3,4]
```

## removeLast(count) 从末尾位置开始向前移除指定数量的元素 

```
var shopping_list = [1,2,3,4]
shopping_list.removeLast(2)
print(shopping_list)
```

## removeFirst() 移除第一个元素 

```
var shopping_list = [1,2,3,4]
shopping_list.removeFirst()
print(shopping_list)
结果:
[2,3,4]
```
## removeLast() 移除最后一个元素

```
var shopping_list = [1,2,3,4]
shopping_list.removeLast()
print(shopping_list)
结果:
[1,2,3]
```



## removeSubrange(Range) 移除某个范围内的元素

```
var shopping_list = [1,2,3,4]
shopping_list.removeSubrange(0...1)
print(shopping_list)
结果:
[3,4]
```
0...1 表示 [0,1] 的区间,即包含0和1的范围

## removeAll() 移除所有 

```
var shopping_list = [1,2,3,4]
shopping_list. removeAll()
print(shopping_list)
结果:
[]
```

>移除数组中全部元素，有一个可选参数，keepCapacity。如果keepCapacity = YES的话，那么数组移除元素后，其存储空间还是存在的，在此往里存储值，不需再给他分配存储空间了。如果keepCapacity=NO的话，那么数组的存储空间就会被回收掉。

```swift
var shopping_list = [1,2,3,4]
shopping_list. removeAll(keepCapacity: true)
print(shopping_list)
结果:
[]
```

# 数组的遍历
比较常见的就是for-loop遍历了。
```
var shopping_list = [1,2,3,4]
for item in shopping_list {
    print(item, terminator: "")
}
结果:
1234
```

还有一种方式是枚举遍历法,我们有时也会称之为迭代器遍历

官网中这样解释到:
>If you need the integer index of each item as well as its value, use the enumerated() method to iterate over the array instead. For each item in the array, the enumerated() method returns a tuple composed of an integer and the item. The integers start at zero and count up by one for each item; if you enumerate over a whole array, these integers match the items’ indices. You can decompose the tuple into temporary constants or variables as part of the iteration:
如果您需要这个项目的每一个值和索引，那么可以使用enumerated()方法来迭代该数组。对于数组中的每个项，enumerated()方法返回一个由一个整数和项组成的元组。整数从零开始，每一项都是一;如果您对整个数组进行枚举，那么这些整数将匹配项目的索引。您可以将元组分解为临时常量或变量，作为迭代的一部分:

```
var shopping_list = ["A","B","C","D"]
let enumerated_list = shopping_list.enumerated()
for (index, value) in  enumerated_list {
    print("Item \(index + 1): \(value)")
}
结果:
Item 1: A
Item 2: B
Item 3: C
Item 4: D
```
其中可以看到index就是它的索引,value是它的值。

## sorted 排序
```
var shopping_list = [53,4,2,45]
for item in shopping_list.sorted(){
    print(item,terminator: " ")
}
结果:
2 4 45 53 
```

## sorted by 有条件的排序
```
var shopping_list = [53,4,2,45]
shopping_list = shopping_list.sorted { (old, new) -> Bool in
    return old<new;
}
print(shopping_list)
如果old<new为升序:
[2, 4, 45, 53]
old>new为降序:
[53, 45, 4, 2]
```

