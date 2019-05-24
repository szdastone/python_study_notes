# 字典

字典为内置类型中唯一的映射结构，基于哈希表存储键值对数据。字典与列表和元组不同，存储的信息会更多，更准确。字典使用花括号({})来表示，使用一系列的键-值，可使用键来访问对应的值。值可以是任意数据，但主键必须可哈希类型，故列表、集合等都不可以作为主键使用。

```
>>> issubclass(list,collections.Hashable)
False
>>> issubclass(int,collections.Hashable)
True
>>> issubclass(tuple,collections.Hashable)
True
```

元组为不可变类型，但如引用了可变类型元素，则不能为主键，比如tuple引用了dict,set,list中任意一种则不可为主键。

```
>>> hash((1,2,3))
2528502973977326415
>>> hash((1,2,[3,4]))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'list'
```

创建字典，比如:

```
product = {'color': 'red', 'price': 51.00}
print(product[color])
print(product[price])
>>>dict(a=1, b=2)
{'a': 1, 'b': 2}
```

初始化键值参数可以元组、列表等可迭代对象方式提供。

```
>>> kvs=(("a",1),["b",2])
>>> dict(kvs)
{'a': 1, 'b': 2}
```

基于动态数据创建，多用zip、map函数或推导式完成。

```
>>> dict(zip("abc",range(3)))
{'a': 0, 'b': 1, 'c': 2}
>>> dict(map(lambda k,v:(k, v+10),"abc",range(3)))
{'a': 10, 'b': 11, 'c': 12}
>>> {k: v+10 for k, v in zip("abc", range(3))} #推导式处理数据
{'a': 10, 'b': 11, 'c': 12}
```

基于已有字典内容扩展或者初始化零值等。

```
>>> a = {"a": 1}
>>> b = dict(a, b=1)        # 在复制 a 内容的基础上，新增键值对
>>> b
{'a': 1, 'b': 1}
>>> c = dict.fromkeys(b, 0)     # 仅用 b 的主键，内容另设
>>> c
{'a': 0, 'b': 0}
>>> d = dict.fromkeys(("counter1", "counter2"), 0)  # 显示提供主键
>>> d
{'counter1': 0, 'counter2': 0}
```



#### 添加、修改、删除键值

字典是动态的，可随时添加键值以及修改、删除键值。如：

```
product = {'color': 'red', 'price': 51.00}
product['qty'] = 5
product['disc'] = -3
product[price] = 53
del product['disc']
```

如主键不存在会出现异常，可先用in、not in语句判断，或用get方法返回默认值。

```
>>> x = dict(a=1)
>>> x["b"]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'b'
>>> "b" in x
False
>>> x.get("b",100)  # 主键b不存在，返回默认值100
100
>>> x.get("a",100)  # 主键a存在，返回实际内容
```

方法get的默认参数仅返回，不影响字典内容。但如想字典插入默认值，则可使用setdefault。

```

>>> x = {}
>>> x.setdefault("a", 0)   #如果有a,返回实际内容，否则新增{a:0}键值对
0
>>> x
{'a': 0}
>>> x["a"] = 100
>>> x.setdefault("a", 0)
100
```

字典不支持加法、乘法、大小等运算，但可比较内容是否相同。

```
>>> {"a": 1, "b": 2} == {"a": 1, "b": 2}
True
```



#### 遍历字典

如列表、元组一样，可遍历字典的所有键值。

```
product = {'color': 'red', 'price': 51.00}
for key, value in product.items:
	print("Key:"+key)
	print("\nValue:"+value)
```

字典的所有key实际组成了一个列表，你可如列表一样进行遍历：

```
for key in product.keys():
	print("Key:"+key)
```

同样，所有的value也组成了列表，可通过product.values()访问。

既然字典的keys()、values()都是列表，那一样适用列表的属性，比如可使用sorted进行临时排序。

#### 嵌套

字典内的元素不一定就是固定一种类型，其值可以是字符串，数字，布尔值，元组，列表，甚至字典。也就是字典中可嵌套列表或字典，同样，列表也可嵌套字典、列表等。

```
product1 = {'color':'red','price':53}
product2 = {'color':'blue','price':44}
products = [product1,product2]
for product in products:
	print(product)

pizza = {
	'crust': 'thick',
	'toppints': ['mushrooms', 'extra cheese'],
}

users = {
	'john': {
		'first': 'jack',
		'last': 'zhao'
	},
	'rose': {
		'first': 'rose',
		'last': 'li'
	},
}
```

#### 视图

Python3默认以视图关联字典内容，这样既可避免复制开销，又可同步观察字典变化。

```
>>> x = dict(a=1,b=2)
>>> ks=x.keys()    #主键视图
>>> "b" in ks      #判断主键是否存在
True
>>> for k in ks: print(k, x[k])   #利用视图迭代字典
...
a 1
b 2
>>> x["b"]=200                #修改字典内容
>>> x["c"]=3                  #没有则新增 
>>> for k in ks: print(k, x[k])  #视图同步变化
...
a 1
b 200
c 3
```

字典如直接引用则可能被接收方修改，如复制则无法获取字典变化。如使用视图，则可同步读取字典内容而无法修改。且可选择不同粒度内容进行传递，如此可将接收方限定为指定模式下的观察者。

```
>>> def test(d):          #传递键值视图(items)，只能读取，不可修改
...     for k, v in d:
...             print(k, v)
...
>>> x=dict(a=1)
>>> d=x.items()
>>> test(d)
a 1
```

视图还支持集合运算，以弥补字典功能上的不足。

```
>>> a = dict(a=1,b=2)
>>> b = dict(c=3,b=2)
>>> ka=a.keys()
>>> kb=b.keys()
>>> ka & kb    #交集：在a、b中同时存在
{'b'}
>>> ka | kb    #并集：在a或b中存在
{'b', 'c', 'a'}
>>> ka - kb    #差集：仅在a中存在
{'a'}
>>> ka ^ kb    #对称差集：仅在a或仅在b中出现，等同“并集-交集”
{'c', 'a'}
```

利用视图集合运算，可简化某些操作。如，只更新，不新增。

```
>>> a = dict(a=1, b=2)
>>> b = dict(b=20, c=3)
>>> ks = a.keys() & b.keys()  # 交集，也就是a中必须存在的主键
>>> a.update({k:b[k] for k in ks})
>>> a
{'a': 1, 'b': 20}
```

#### 扩展

标准库collections的默认字典(defaultdict)类似setdefault包装。当主键不存在，调用构造参数提供的工厂函数返回默认值。

```
>>> d = collections.defaultdict(lambda: 100)
>>> d["a"]
100
>>> d["b"] +=1
>>> d
defaultdict(<function <lambda> at 0x00000171FD7AA840>, {'a': 100, 'b': 101})
```

有序字典(OrderedDict)会记录主键首次插入的次序。

```
>>> d = collections.OrderedDict()
>>> d["z"] =1
>>> d["a"] =2
>>> d["x"] =3
>>> for k, v in d.items(): print(k, v)
z 1
a 2
x 3
```

与前面介绍不同，计算器(Counter)对不存在主键返回0，但不会新增。

```
>>> d = collections.Counter()
>>> d["a"]
0
>>> d["b"]+=1
>>> d
Counter({'b': 1})
```

链式字典(ChainMap)以单一接口访问多个字典内容，其自身并不存储数据。读操作按参数顺序依次查找各字典，但修改操作(增、改、删)仅针对第一字典。

```
>>> a = dict(a=1, b=2)
>>> b = dict(b=20, c=30)
>>> x = collections.ChainMap(a, b)
>>> x["b"], x["c"]    #按顺序命中
(2, 30)
>>> for k,v in x.items(): print(k, v) #遍历所有字典
...
b 2
c 30
a 1
>>> x["b"] = 999  #更新，命中第一字典
>>> x["z"] = 888  #新增，命中第一字典
>>> x
ChainMap({'a': 1, 'b': 999, 'z': 888}, {'b': 20, 'c': 30})
```

链式字典查找次序可被调用链的后续函数读取，这也是继承的体现。而修改操作限定在当前第一字典中，故不会影响父级字典的同名主键设置。

```
>>> root = collections.ChainMap({"a": 1})
>>> child = root.new_child({"b": 200})
>>> child["a"] =100
>>> child
ChainMap({'b': 200, 'a': 100}, {'a': 1})
>>> child.parents
ChainMap({'a': 1})
```



# 集合

集合存储非重复对象。这里的非重复是指不是同一对象外，值也不能相等。

> 判重公式：(a is b) OR (hash(a) == hash(b) AND a==b )
>
> 如不是同一对象，则先判断哈希值，然后比较内容。受限哈希算法，不同内容可能返回相同哈希值(哈希碰撞)。为啥要先比较哈希值呢？首先，与内容相比，整数哈希值比较性能要高很多，其次，哈希值不同，内容肯定不同，则不需继续比较内容。	

```
>>> a=1234
>>> b=1234
>>> a is b   #内容同，但不是相同对象
False
>>> s={a}    #创建集合，初始化元素a
>>> b in s   #用内容相同的b进行判重 
True
```

按操作方式，集合分可变(set)和不可变(frozenset)两种，内部实现相同。用数组实现的哈希表存储元素对象引用，这也就要求元素必须为可哈希类型。

#### 创建

与字典语法相同，使用花括号，但初始化数据非键值对。

```
>>> type({})   # 没有初始化值，表示创建空字典
<class 'dict'>
>>> type({"a":1}) #字典：键值对
<class 'dict'>
>>> type({1})     #集合
<class 'set'>

>>> set({1, "a", 1.0})
{1, 'a'}
>>> frozenset(range(3))
frozenset({0, 1, 2})
>>> {x+1 for x in range(6) if x%2 == 0 }  //推导式创建
{1, 3, 5}
>>> s = {1}
>>> f = frozenset(s)  # set与frozenset转换
>>> set(f)
{1}
```

#### 操作

支持大小、相等运算符。

```
>>> {1, 2} > {2, 1}
False
>>> {1, 2} == {2, 1}
True
```

子集判断不可用in、not in语句，它仅用来检查是否包含某个元素。

```
>>> {1, 2} <= {1, 2, 3}   # 子集：issubset
True
>>> {1, 2, 3} >= {1, 2}   # 超集：issuperset
True
>>> {1, 2} in {1, 2, 3}   # 判断是否包含{1， 2}单一元素
False 
```

集合作为初等数学概念，重点是并差集运算。

> 交集： 同时属于A、B两个集合。
>
> 并集：A、B的所有元素。
>
> 差集：仅属于A，不属于B。
>
> 对称差集：仅属于A，加上仅属于B，相当“并集 - 交集”。

```
>>> {1, 2, 3} & {2, 3, 4}  #交集：intersection
{2, 3}
>>> {1, 2, 3} | {2, 3, 4}  #并集：union
{1, 2, 3, 4}
>>> {1, 2, 3} - {2, 3, 4}  #差集: difference
{1}
>>> {1, 2, 3} ^ {2, 3, 4}  #对称差集：symmetric_difference
{1, 4}
```

集合运算还可与更新操作一起使用。

```
>>> x = {1, 2}
>>> x |= {2, 3}
>>> x
{1, 2, 3}
>>> x = {1, 2}
>>> x &= {2, 3}
>>> x
{2}
```

删除操作不能用remove，可能引发异常，可改用discard。

```
>>> x = {2, 1}
>>> x.remove(2)
>>> x.remove(2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 2
>>> x.discard(2)
```

#### 自定义类型

自定义类型虽可是哈希类型，但默认实现并不足以完成集合去重操作。

```
>>> class User:
...     def __init__(self,uid,name):
...             self.uid=uid
...             self.name=name

>>> issubclass(User, collections.Hashable)
True
>>> u1 = User(1, "user1")
>>> u2 = User(1, "user1")
>>> s=set()
>>> s.add(u1)
>>> s.add(u2)
>>> s
{<__main__.User object at 0x00000171FD7BC780>, <__main__.User object at 0x00000171FD7BCB00>}
```

根本原因是默认实现的\_\_hash\_\_方法返回随机值，而\_\_eq\_\_仅比较自身。为符合逻辑需求，需重载这2个方法。

```
>>> class User:
...     def __init__(self, uid, name):
...             self.uid =uid
...             self.name=name
...     def __hash__(self):            #针对uid去重，忽略其他字段
...             return hash(self.uid)
...     def __eq__(self, other):
...             return self.uid == other.uid
...
>>> u1 = User(1,"user1")
>>> u2 = User(1,"user1")
>>> s=set()
>>> s.add(u1)
>>> s.add(u2)
>>> s
{<__main__.User object at 0x00000171FD7BCB38>}
>>> u1 in s
True
>>> u2 in s
True
```

