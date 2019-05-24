# 列表

Python内置的列表list是一种有序集合，可随时添加或删除其中元素，可当做队列或栈来使用。用方括号([])来表示列表。如：

```
>>> fruits = ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
```

如要定义一个空列表，则如下：

```
>>> empty_list = []
>>> empty_list
[]
>>> len(empty_list)
0
```

列表内部结构由两部分组成，保存元素数量和内存分配计数的头部，以及存储元素指针的独立数据。所有元素项使用该数组保存指针引用，并不嵌入实际内容。

不同于数组，列表仅存储指针，而对元素内容不关心，故可用不同类型混合。

```
>>>l=[1,"abc",3.14]
>>>list("abc")
['a', 'b', 'c']
>>>[x**2 for x in range(3)]
[0, 1, 4]
>>>[x for x in range(10) if x%2 ==0]  #for循环初始数据，if表达式过滤
[0, 2, 4, 6, 8]
```



### 列表基本操作

#### 访问列表

列表第一个元素为0，要访问列表只需如数组般访问即可，如：

```
>>>print(fruits[0])
orange
```

要访问最后一个元素，可指定索引为-1，如:

```
>>>print(fruits[-1])
banana
```

可对访问的值进行操作，比如：

```
>>>print(fruits[-2].title())
Apple
```

可使用len来获取列表的长度，如

```
>>>len(fruits)
7
```

判断元素是否存在，使用in，而非index方法。

```
>>>2 in [1,2]
True
```



#### 修改、添加元素

修改列表的元素值很简单，直接将访问的值进行修改就可以了，如：

```
>>>fruits[0] = 'grape'
>>>print(fruits)
 ['grape', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
```

添加元素，可使用append，如

```
fruits.append('abc')
```

这样就将元素添加到了列表的最后。而其他元素并没有受到影响。

插入元素到指定位置，使用insert,如

```
fruits.insert(1,'abc')
```

将元素添加到了指定位置，该位置以前元素没有任何影响，该位置以后的索引右移了一位。

也可使用加法或乘法进行添加元素。

```
>>>a=[1,2]
>>>b=a
>>>a=a+[3,4]  #新建列表对象，然后与a关联
>>>a          #a,b结果不同，a指向了新对象 
[1,2,3,4]
>>>b
[1,2]
>>>a=[1,2]
>>>b=a
>>>a +=[3,4]   #直接修改a内容
>>>a		   #a,b结果相同，确认修改原对象 
[1,2,3,4]
>>>b
[1,2,3,4]
>>>a is b
True
```

“+=”运算符编译器处理为INPLACE_ADD操作，也就是修改原数据，而非创建新对象。效果类似list.extend方法。

#### 删除元素

- 使用del语句删除元素,比如：

```
del fruits[1]
```

del可删除任何位置的列表元素，但必须有索引。

del也可以删除整个列表，注意删除后列表则不可访问了。

```
>>>del fruits
>>print(fruits)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'fruits' is not defined
```

- 使用pop()来删除元素，比如：

```
poped_fruits = fruits.pop()
```

pop删除元素后会返回删除的元素。而且pop一定是删除最后一个元素。如果要pop指定位置元素，则比如传入索引，比如：

```
fruits.pop(2)
```

那么，到底是该使用del语句还是pop()方法了，一个简单判断标准就是：如从列表中删除一个元素，且不再使用它了，就是用del语句；如删除元素后还要继续使用它，则使用方法pop()。

- remove删除元素，如果知道要删除元素的值，但不知道位置，则使用remove了。比如：

```
fruits.remove('abc')
```

remove只删除第一个指定的值，如要删除的值在列表中多次出现，则需要使用循环判断来删除了。

#### 列表排序

- 永久排序

使用方法sort()进行永久排序，比如定义一个列表及排序:

```
>>>cars = ['bmw', 'audi', 'toyota', 'subaru']
>>>cars.sort()
>>>print(cars)
['audi', 'bmw','subaru','toyota']
```

如要反排序，则使用方法sort()时传入参数reverse=True,如下：

```
>>>cars = ['bmw', 'audi', 'toyota', 'subaru']
>>>cars.sort(reverse=True)
>>>print(cars)
['toyota','subaru','bmw','audi']
```

- 临时排序

使用sorted()对列表临时排序，如：

```
>>>cars = ['bmw', 'audi', 'toyota', 'subaru']
>>>print(sorted(cars))
['audi', 'bmw','subaru','toyota']
>>>print(cars)
['bmw', 'audi', 'toyota', 'subaru']
```

同样，可使用reverse=True来进行反排序。

* 有序列表插入元素

可利用bisect模块，要有序列表插入元素。它使用二分法查找适合位置，可用来实现优先对列或一致性哈希算法。

```
>>>d=[0,2,4]
>>>bisect.insort_left(d,1)  #插入新元素后，依然保持有序状态
>>>d
[0, 1, 2, 4]
```

对于自定义符合类型，可通过重载比较运算符(\_\_eq\_\_、\_\_lt\_\_)等实现自定义排序条件。

* 按设定条件排序

```
>>>class User:
	def __init__(self,name,age):
		self.name = name
		self.age = age
	
    def __repr__(self):
    	return f"{self.name} {self.age}"
>>>users = [User(f"user{i}", i) for i in (3,1,0,2)]
>>> users
[user3 3, user1 1, user0 0, user2 2]
>>>users.sort(key=lambda u:u.age)  #使用lambda匿名函数返回排序条件
>>> users
[user0 0, user1 1, user2 2, user3 3]
```

#### 顺序与长度

列表也可以使用reverse()进行修改顺序，调用后会永久修改列表元素的排列顺序，如要恢复，则再次调用。

```
>>>cars = ['bmw', 'audi', 'toyota', 'subaru']
>>>cars.reverse()
>>>print(cars)
['subaru','toyota','audi','bmw']
```

列表的长度使用len,如：

```
>>>cars = ['bmw', 'audi', 'toyota', 'subaru']
>>>len(cars)
4
```

请注意列表的索引，如超出这个范围则会错误，如列表的长度为n,则序号为[-n...n-1]之间。如超出，则会出现如下错误：

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
```

#### 遍历列表

要访问列表的所有元素，可使用for循环。

```
cars = ['bmw', 'audi', 'toyota', 'subaru']
for car in cars:
	print(car)
```

#### 创建列表

- 使用函数range()

  可使用range()来生成一系列数字，如

  ```
  for value in range(1,5)
  	print(value)
  ```

  range(1,5)会打印数字1-4，range()有3个参数，开始数字，结束数字以及步长，如range(1,10,3)则会打印1,4,7这3个数。

  如要生成数字列表，可如下：

  ```
  >>>nums = list(range(1,6))
  >>>print(nums)
  [1,2,3,4,5]
  ```

  

- 数字列表的简单统计

  可使用min,max,sum来对数字列表进行统计。

  ```
  >>>digits = list(range(1,10))
  >>>min(digits)
  1
  >>>max(digits)
  9
  >>>sum(digits)
  45
  ```

  

- 列表解析

  列表解析就是将for循环以及创建新元素代码合并成一行，病自动附件新元素。如

  ```
  >>>squares = [values**2 for value in range(1,11)]
  >>>print(squares)
  [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
  ```

  

- 列表切片

  如果访问列表部分元素，可使用切片，一般要指定第一个元素及最后一个元素的索引，和range()函数差不多吧。如：

  ```
  >>>cars = ['bmw', 'audi', 'toyota', 'subaru']
  >>>print(cars[0:3])
  ['bmw', 'audi', 'toyota']
  ```

  如没有指定第一个索引，则从列表开头开始，如没有指定第二个索引，则到列表结尾结束;当然，索引也可以是负数，如:

  ```  
  cars[:3]  #等同cars[0:3] 复制后依然指向同一个对象
  cars[2:]  #等同cars[2:4]
  cars[:]   #等同cars或cars[0:4]
  cars[-3:] #等同cars[-3,0],打印最后3个
  ```

  当然切边后的列表也是列表，所以还是可以遍历的，可试下？

- 复制列表

  复制列表，可使用切片方法来复制。如

  ```
  cars = ['bmw', 'audi', 'toyota', 'subaru']
  new_cars = cars[:]
  ```

  为啥不直接赋值呢？比如new_cars=cars不行么？好像是可以，但还是有问题。赋值后指向同一个地址，当修改一个列表，则另一个列表也修改了，这是你所预期的？
  
  返回切片时创建新列表对象，并复制相关指针数据到新数组。除所引用目标相同外，对列表本身的修改(插入、删除等)并不影响。
  
  注意复制的时指针（引用），而非目标元素对象。对列表自身的修改互不影响，但对目标元素对象的修改是共享的。
  
  ```
  >>>a=[0,2,4,6]
  >>>b=a[:2]
  >>>a[0] is b[0]   #复制引用，依然指向同一对象
  True
  >>>a.insert(1,1) #对a列表操作，不影响b
  >>>a
  [0,1,2,4,6]
  >>>b
  [0,2]
  ```
  
  

# 元组

列表是可变的数据集，那么元组呢？元组就是不可变的数据集。元组一旦修改，则不可改变。元组和列表类似，元组是使用圆括号来标识的。定义元组后，可如列表一样进行访问，遍历等，但不可以修改，删除。

因元组为不可变类型，它的指针数组无需变动，故一次性完成内存分配。系统会缓存复用一定长度的元组内存(含指针数组)。元组性能要优于列表。



#### 定义元组

元组定义很列表类似，如：

```
dimensions = (200,500)
```

如果要定义一个空元组，可如下定义：

```
empty = ()
```

那定义一个有一个元素的元组呢？

```
>>>t = (1)
>>>print(t)
1
```

怎么打印出来不是元组，是个数字1呢？这是因为()既可以标识元组，也可以表示数学中的小括号，为了消除歧义，必须加一个逗号，，如下定义一个元素的元组：

```
>>>t=(1,)
>>>print(t)
(1,)
>>>type(t)
<class 'tuple'>
>>>b=(1)
>>>type(b)
<class 'int'>
```

支持与列表类似运算符操作。但不能修改，总返回新对象，

```
>>> (1,2)+(3,)
(1, 2, 3)
>>> (1,2)*2
(1, 2, 1, 2)
>>> a=(1,2,3)
>>> b=a
>>> a+=(4,5)  #与列表不同，创建新tuple，而不是修改原内容。等同a=a+(4,5)
>>> a
(1, 2, 3, 4, 5)
>>> b
(1, 2, 3)
```

因列表可变，故序号无法与元素对象构成固定映射。但元组不同，相同序号总是同一个对象。故可为序号取“别名”。

```
>>> User = collections.namedtuple("User","name, age") #创建并指定字段
>>> issubclass(User,tuple)  #tuple子类
True
>>> u=User("qyuhen",60)
>>> u.name,u.age   #使用字段名访问
('qyuhen', 60)
>>> u[0] is u.name  #序号与“别名”相同
True
```

## 数组

数组和列表、元组的本质区别在于：元素单一类型和内容嵌入。

```
>>> import array
>>> a= array.array("b",[0x11,0x22,0x33,0x44])
>>> memoryview(a).hex()
'11223344'
>>> a=array.array("i")
>>> a.append(100)
>>> a.append(1.23)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: integer argument expected, got float
```

可直接存储包括Unicode字符在内的各种数字。如要复合类型，则需用struct,marsha1,pickle等转换二进制字节后再行存储。

与列表类似，数组长度不固定，可按需扩张或收缩内存。

```
>>> a=array.array("i",[1,2,3])
>>> a.buffer_info()   #返回缓冲区内存地址及长度
(2811820040688, 3)
>>> a.extend(range(10000))   #追加大量内容后，内存地址和长度变化了。
>>> a.buffer_info()
(2811851399072, 10003)
```

由于可指定数字类型，故数组可节约内存。内容嵌入也避免了对象额外开销，减少了活跃对象数量和内存分配次数。

  







