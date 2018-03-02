---
layout: blog
technology: true 
background: purple
background-image: http://p4wibncwo.bkt.clouddn.com/john-luke-laube-459401-unsplash.jpg
title:  "Swift - 集合 - Set and Dictionary"
date:   2018-01-15
category: 技术
tags:
- iOS
- Swift
- Set
- 集合
- Dictionary
---

# Set
Set 也是集合的一种,很多方法与Array类似,但似仍然具有一些Array不具备的特性。

Set具有无序且不重复的特性.我们在创建的时候,设定的值是:`1,2,3`,但是出来的结果有可能是`2,3,1`,也有可能是`3,1,2`。其值并不像Array那样井然有序。

我们首先来创建一个空的Set
```
let numSet = Set<String>()
print(numSet)
```
Set<type> 对于Swift的安全性来说,创建的时候没有数据用来推导数据类型的话,我们就要指定类型了.
Set 指定了数据类型之后，也就意味着,它的值就必须是该类型，这一点与其它的集合类型都是一致的。

创建一个具有数据的Set集合
```
let numSet = Set(arrayLiteral: 1,3,4)
let wordSet:Set = ["A","B","C"]
let wordTypeSet:Set<String> = ["A","B","C"]
```


# Set的内置方法
## insert 插入
Set中想加入一个新元素,无法使用apped,要使用insert进行插入新的元素

```
var numSet = Set<String>()
numSet.insert("A")
print(numSet)
结果:
["A"]
```

## update
也可以通过update的方式将新元素更新到Set中
```
var wordSet:Set = ["A","B","C"]
wordSet.update(with: "D")
print(wordSet)
结果:
["B", "A", "C", "D"]
```

在Set中,我们有以下四个办法高效地执行基本的集操作，例如将两个集合组合在一起，确定两个集合具有相同的值，或者确定两个集合是否包含所有、一些或没有相同的值。
![setVennDiagram_2x.png](http://upload-images.jianshu.io/upload_images/1120896-7a104e4770532e2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

## ntersection  
交集,获取两个集合中都具有的元素

```
var oldSet:Set<String> = ["A","B","C"]
var newSet:Set<String> = ["C","D","E"]
let tempSet  =  oldSet.intersection(newSet)
print(tempSet)
结果:
["C"]
```

## symmetricDifference 
将两个集合结合起来，但排除了两者都有的元素

```
var oldSet:Set<String> = ["A","B","C"]
var newSet:Set<String> = ["C","D","E"]
let tempSet  = oldSet.symmetricDifference(newSet);
print(tempSet)
结果:
["B", "A", "D", "E"]
```

## union
将两个集合连接起来,去除重复的数据.
```
var oldSet:Set<String> = ["A","B","C"]
var newSet:Set<String> = ["C","D","E"]
let tempSet  = oldSet.union(newSet);
print(tempSet)
结果:
["B", "A", "C", "D", "E"]
```

## subtracting
创建一个没有在指定集中的值的新集合。
看一下结果对比:
```
结果一:
var oldSet:Set<String> = ["A","B","C"]
var newSet:Set<String> = ["D","E","F"]
let tempSet  = oldSet.subtracting(newSet);
print(tempSet)
结果:
["C", "B", "A"]

结果二:
var oldSet:Set<String> = ["A","B","C"]
var newSet:Set<String> = ["C","B","F"]
let tempSet  = oldSet.subtracting(newSet);
print(tempSet)
结果:
["A"]
```
通过结果我们可以分析出:
这个方法其实就是oldSet集合排除了另一个集合的元素之后，剩下的元素会组成一个新集合。


# Set中的判断
在Set中，我们可以使用`=` 来判断两个集合是否相等:
```
let firstSet: Set = ["A","B","C"];
let secondSet: Set = ["A","B","C"];
let thirdSet: Set = ["B","C","D"];
print("firsSet is equal secondSet : \(firstSet == secondSet)")
print("firsSet is equal thirdSet : \(firstSet == thirdSet)")
结果:
firsSet is equal secondSet : true
firsSet is equal thirdSet : false
```

## isSubSet
用此方法确定一个集合的所有值是否包含在指定的集合中。
也就是说，前者是否是后者的子集
```
let firstSet: Set = ["A","B","C","D","E"];
let secondSet: Set = ["D","E","F"];
let thirdSet: Set = ["B","C","D"];
print(secondSet.isSubset(of: firstSet))
print(thirdSet.isSubset(of: firstSet))
结果:
false
true
```

## isSuperSet
用此确定一个集合是否包含了一个指定集合中的所有值。
也就是,前者是否是后者的超集

```
let firstSet: Set = ["A","B","C","D","E"];
let secondSet: Set = ["D","E","F"];
let thirdSet: Set = ["B","C","D"];
print(firstSet.isSuperset(of: secondSet))
print(firstSet.isSuperset(of: thirdSet))
结果:
false
true
```

## 可以用isStrictSubset或者isStrictSuperset
来确定一个集合是一个子集还是超集，但不等于一个指定的集合。

```
let firstSet: Set = ["A","B","C","D","E"];
let secondSet: Set = ["D","E"];
let thirdSet: Set = ["A","B","C","D","E"];
print(secondSet.isStrictSubset(of: firstSet))
print(thirdSet.isStrictSubset(of: firstSet))
print(firstSet.isStrictSuperset(of: secondSet))
print(firstSet.isStrictSuperset(of: thirdSet))
结果:
true
false
true
false
```
简单来说的话, isStrictSubset或者isStrictSuperset 相比较于isSubSet和isSuperSet,多了一个是否等于超集的判断。

##  isDisjoint 
用此确定两个集合是否没有共同的值。如果包含返回false,不包含返回true
```
let firstSet: Set = ["A","B","C",];
let secondSet: Set = ["D","E"];
let thirdSet: Set = ["A","B"];
print(firstSet.isDisjoint(with:secondSet))
print(firstSet.isDisjoint(with:thirdSet))
结果:
true
false
```

# Dictionary 字典
字典是一种以键值对的方式存在的集合。键是唯一的,不能重复。同样也是无序的集合。
[`key 1`: `value 1`, `key 2`: `value 2`, `key 3`:` value 3`]

我们来试着创建一个空的字典
```
let dict = [String:String]()
或者这样
let dict:[Int:String] = [:]
也可以这样
let dict = Dictionary<String,String>()
```
我们创建一个有值得字典:
```
let dict:Dictionary = ["name":"啊威","age":"18"] 
print(dict)
结果:
["name": "啊威", "age": "18"]
```
更快捷的方法:
```
let dict = ["name":"啊威","age":"18"] 
```

在实际的时候，我们value的数据类型可能并不统一,而Swift对于类型的要求却很严格,所以我们需要另一种定义的方式:
```
let dict:Dictionary = ["name":"啊威","age":18]
```
当我们这样做的时候,系统会报出这样的错误:
```
Heterogeneous collection literal could only be inferred to '[String : Any]';
 add explicit type annotation if this is intentional
异构集合文字只能被推断为“String:Any”;如果这是有意的，添加显式类型注释
```
我们不能隐式的这样写,系统要求我们显式地写出来
```
let dict:Dictionary = ["name":"啊威","age":18] as [String : Any]
print(dict)
结果:
["name": "啊威", "age": 18]
```
可以看到18的类型并不是String了。因为我们标注的类型是String和Any(任何类型)

那么,如何直接设置键的值呢?
```
var dict:Dictionary = ["name":"啊威","age":"18"]
dict["age"] = "25"
print(dict["age"] ?? "0")
结果：
25
```
我们通过这样的方式来设置字典的值。
也可以这样做

##  updateValue(value,forKey:key)

```
var dict:Dictionary = ["name":"啊威","age":"18"]
dict.updateValue("23", forKey: "age")
print(dict["age"] ?? "0")
结果:
23
```

## popFirst 获取第一对键值
```
var dict:Dictionary = ["name":"啊威","age":"18"]
print(dict.popFirst() ?? "")
print(dict.popFirst()?.key ?? "")
结果:
(key: "name", value: "啊威")
age
```

## keys,values 分别获取字典的全部键和全部值
```
var dict:Dictionary = ["name":"啊威","age":"18"]
let dictKeys = [String](dict.keys)
print(dictKeys)
let dictValues = [String](dict.values)
print(dictValues)
结果:
["name", "age"]
["啊威", "18"]
```

## removeValue(forKey:) 根据提供的key移除key和value

```
var dict:Dictionary = ["name":"啊威","age":"18"]
let removedValue = dict.removeValue(forKey: "name")!
print("removed value is : \(removedValue)")
print(dict)
结果:
removed value is : 啊威
["age": "18"]
```

##  remove(at:index) 根据提供的下标进行删除

```
var dict:Dictionary = ["name":"啊威","age":"18"]
dict.remove(at: dict.startIndex)
print(dict)
结果:
["age": "18"]
```

# 字典的遍历
字典的遍历与其他集合不同的主要在于有键值的明显区分,所以我们可以以下的方式去遍历:

##  获取字典的键和值:
```
var dict:Dictionary = ["name":"啊威","age":"18"]

for (name,age) in dict{
    print("\(name):\(age)")
}
结果:
name:啊威
age:18
```

##  获取每一对键值结果:
```
var dict:Dictionary = ["name":"啊威","age":"18"]
for name in dict{
    print("\(name)")
}
结果:
(key: "name", value: "啊威")
(key: "age", value: "18")
```

