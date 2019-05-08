# 列表

Python内置的列表list是一种有序集合，可随时添加或删除其中元素。用方括号([])来表示列表。如：

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





  







