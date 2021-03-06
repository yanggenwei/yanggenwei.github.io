---
layout: post
title:  "学生生管理系统 - Python3.0小例子"
date:   2017-06-23
categories: Python
---

**采用Python3.0,使用Python3.0以下编译会报错的。**

以前在学习Python的时候,写的一个小例子,简单的说是为了锻炼一下面向对象而写的,分享出来:

这是一个简单的学生信息管理系统,通过不同的模块构建，最后达成我们得目的,先看下最终效果:
```
------登录------
账号:
admin
密码:
123
#------ 欢迎进入XXX系统 ----
1.添加学生
2.修改学生信息
3.根据姓名查找学生 代码:name
根据年龄查找学生 代码:age
根据科目查找学生 代码:subject
根据成绩查找学生 代码:score
4.根据姓名删除学生
5.显示所有学生信息
```


大概的效果就是这样了,因为是初学的时候做的,很多东西没有去考虑,毕竟在开始的时候考虑太多,只会比较迷茫。我们先一步一步来咯。

首先先定义个数据源来存放数据
```
# coding=utf-8
dict1 = {'name':'小明','age':18,'subject':'数学','score':99}
list = [dict1]
```

然后我的习惯是先考虑下一步的情况,所以要分析我们接下来要做的事情:登录,选择以及相应的处理方法，通过结果示例,我们需要做的是添加学生,修改学生，查找学生，删除学生，显示所有学生:
那么我就可以先定义几种方法(只是设想,并不是具体的代码):
```
登录: login
主控制选择台: selectInfo
添加学生信息: addStudent
修改学生信息: editStudent
具体的修改学生信息:editStudentInput
这里有两种查询方式:一种是根据后面提示的代码进行查询
一种是直接给出4个选项进行选择:
查询方式1：searchStudentInfoA
查询方式2:  searchStudentInfoB
显示列表中所有学生信息: showStudentInfo
用于将dict格式的显示成具体的格式: showStudentInfoWithDict
删除学生信息: delStudentInfo
```
以上基本就是我们需要用的方法了，知道了我们需要做什么,那么接下来就是具体的实施了,首先就是我们需要一个开始系统的方法:
```	
begin(list)
```
简单暴力无污垢,也就是我们写到最后,就是为了这么一个begin.

好了,接下来就是将begin进行完善。
```
def begin(list):
login()
isEnd = False #用于判断是否结束系统
#循环输入
while(not isEnd):
isEnd = selectedInfo(list)
print('退出系统...')
```
为什么要这样写,因为我们需要做的就是开始的时候需要先登录,登录都过不去,那是肯定是进入不了操作台的。所以登录里面需要的操作就是一个界面用于输入账号和密码，还有的就是保证输入是正确。
```
#login 登录模块，用于用户登录，如果登录失败，会重复登录
def login():
isLogin = True#控制循环是否结束
while(isLogin):
print('------登录------')
print('账号:')
username = input()#输入用户名
print('密码:')
password = input()#输入密码
if(username=='admin' and password=='123'):#判断用户名和密码是否正确
isLogin = False
```
在保证了登录一定成功之后,那么就需要考虑的是登录操作台的功能。这个可以说是个大集合,集成了所有的功能。我们需要给出提示,然后分别去实现不同的功能。

```
#主页面选择模块
def selectedInfo(list):
#手动先设定学生的信息
code = ['name','age','subject','score']
print('#------ 欢迎进入XXX系统 ----')
print('1.添加学生')    
print('2.修改学生信息')
print('3.根据姓名查找学生 代码:name')
print('根据年龄查找学生 代码:age')
print('根据科目查找学生 代码:subject')
print('根据成绩查找学生 代码:score')
print('4.根据姓名删除学生')
print('5.显示所有学生信息')
print('请输入您的选择:')
sel = int(input()) #输入选择
if(sel == 1):
addStudent(list)#添加学生
elif sel==2:
editStudent(list)#修改学生信息
elif sel==3:
#------这是方式1------------
# print('请输入对应的查找代码:')
# myCode = input()
# if myCode in code:	#判断输入的代码是否是合格的代码
# searchStudentInfoA(list,myCode)#如果代码输入正确那么就开始查找
# else:
# print('您输入的代码不正确')
#------这是方式2------------	
print('1.姓名  2.年龄   3.成绩   4.科目 ')
selected = int(input())
searchStudentInfoB(list,selected)

elif sel==4:
delStudentInfo(list)#删除学生信息
elif sel==5:
showStudentInfo(list)#显示所有学生
print('是否退出系统 1/0:') 
return int(input())
```

接下来的就是具体的实现方法了。
```
#添加学生模块
添加学生主要就是对list内容的追加,需要注意的可能就是输错值会崩溃,需要用到try except去捕捉可能发生的错误,当然更具体的捕获可以自己去自定义。
def addStudent(list):
newDict = {} #用于创建一个新学生
isError = False #用于判断输入的时候，是否会发生错误
while(not isError):
try: #捕捉可能会发生的错误
print('----添加学生----')
print('学生姓名：')
stuName = input()#输入学生姓名
print('学生年龄:')
stuAge  = int(input())#输入学生年龄
print('学生科目:')
stuSubject = input()#输入学生科目
print('学生成绩:')
stuScore = float(input())#输入学生成绩
#创建一个字典，用于表示学生信息
newDict = {'name':stuName,'age':stuAge,'subject':stuSubject,'score':stuScore}
isError = True	#如果没有错误发生，那么就会设置isError的值为True,就表示可以结束循环了
except ValueError:#只捕捉值的错误
print('您输入的数据类型不匹配，请重新输入!')
else:#如果没有错误的发生，那么将字典追加到列表中
list.append(newDict)	#将字典添加到list中
print('学生添加成功!')
```

修改学生主要是从目标字典当中将目标取出来，然后对当前取出来的字典进行修改就可以了。
```
#修改学生信息模块
def editStudentInput(dict):
print('请选择如下选项进行修改:')
print('1.年龄  2.科目 3.成绩')
sel = int(input())
if(sel == 1):
print('请输入新年龄:')
newAge = int(input())
dict['age'] = newAge	#重新给age赋值
elif sel == 2:
print('请输入新科目:')
newSubject = input()
dict['subject'] = newSubject#重新给subject赋值
elif sel == 3:
print('请输入新成绩:')
newScore = float(input())#重新给Score赋值
dict['score'] = newScore
```

```	
#修改学生信息 原理，在学生列表中寻找每一个学生对象，判断学生名称是否相同	
def editStudent(list):
print('---修改学生信息----')
print('请输入需要修改的学生名称: ')
stuName = input()
for dict in list:	#在学生列表里面，寻找每一个学生信息
if stuName == dict['name']:
editStudentInput(dict)
break;

```
显示学生就在遍历整个list,然后在根据具体的格式输出就可以了
```			
#显示所有学生信息
def showStudentInfo(list):
for dict in list:
showStudentInfoWithDict(dict)

```

```
#用于将dict格式的显示成具体的格式
def showStudentInfoWithDict(dict):
print('名称:',dict['name'],'年龄:',dict['age'],'科目:',dict['subject'],'成绩:',dict['score'])
```

删除也就是在整个list列表中去寻找,如果寻找到了之后,那么删除当前找到的dict字典就可以了
```	
#删除学生信息
def delStudentInfo(list):
print('请输入需要删除的学生姓名:')
name = input()	#输入学生姓名
for dict in list:	#遍历整个学生列表
if name == dict['name']: #判断当前的学生是否是寻找的学生
list.remove(dict)			#如果是的，那么就删除当前的学生所代表的字典
break
```

查找的也是同理,遍历整个list列表,然后找到输出就可以了
```			
#查找方式一
def searchStudentInfoA(list,type):#通过代码来识别不同的选择类型
print('请输入查找学生的',type)
newType = input()
if type == 'age':		#由于age和score不是字符串类型，需要将其转换成对应的类型
newType = int(newType)
elif type == 'score':
newType = float(newType)

for dict in list:		#遍历整个学生列表
if newType == dict[type]: #判断当前的学生是否是寻找的学生
showStudentInfoWithDict(dict)
```

```				
#查找方式二				
def searchStudentInfoB(list,type):
#1.姓名  2.年龄   3.成绩   4.科目
if type ==1:
print('请输入查找学生的姓名')
name = input()
for dict in list:		#遍历整个学生列表
if name == dict['name']: #判断当前的学生是否是寻找的学生
showStudentInfoWithDict(dict)
elif type==2:
print('请输入查找学生的年龄')
age = int(input())
for dict in list:		#遍历整个学生列表
if age == dict['age']: #判断当前的学生是否是寻找的学生
showStudentInfoWithDict(dict)
elif type==3:
print('请输入查找学生的成绩')
score = float(input())
for dict in list:		#遍历整个学生列表
if score == dict['score']: #判断当前的学生是否是寻找的学生
showStudentInfoWithDict(dict)
elif type==4:
print('请输入查找学生的科目')
subject = input()
for dict in list:		#遍历整个学生列表
if subject == dict['subject']: #判断当前的学生是否是寻找的学生
showStudentInfoWithDict(dict)
```

总的来说,这也是一种面向对象思想的简单表现吧。我们通过将功能模块化,将功能不要全部写在一个方法当中。这样分化了个体功能之后，使得在分配的任务的时候，大家都能各司其职。
比如,XX去写登录,XX去写添加学生模块,XX去写修改之类的。让分工更加明确化。
维护起来也容易。比如我们运行程序，一直都正常，突然走到修改的时候发生了错误，那么我就会知道是editStudentInfo的错误了，而不会在整个文件中去盲目的寻找。

上面的讲解可以直接创建一个test.py，然后把代码全部贴进去编译就可以了。

有任何问题大家都可以提出来,欢迎讨论。 
</small>




