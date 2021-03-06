# 表达式

### 赋值

赋值操作就是为变量名与目标对象建立关联。

多个名字赋值：

```
>>>a = b = c = 1023
>>> a is b is c
True 
```

以逗号分割多个值会当做元组的初始化元素。

```
>>> x = 1, "abc", {10, 20}
>>> x
(1, 'abc', {10, 20})
```

#### 增量赋值

增量赋值就是直接修改原对象内容，实现累加效果。前提是目标必须存在，而且目标对象允许，否则为普通赋值。也可以可变列表，不可变元组进行增量赋值。

```
>>> s = 0
>>> s += 10
>>> a = b = []
>>> a += [1,2]
>>> a is b               # 同一对象
True
>>> c = d = ()
>>> c += (1, 2)
>>> c is d               # 新对象
False
```

可看出，列表直接修改原内容，而元组新建对象。按编译器处理是按INPLACE_ADD指令，但最终执行按时按目标类型定。这里的+=，对应\_\_iadd\_\_方法，如该方法不存在，则会执行\_\_add\_\_方法，这样就从增量操作变成了普通加法操作了。

```
>>> "__iadd__" in dir([1,2])
True
>>> "__iadd__" in dir((1,2))
False
>>> "__add__" in dir((1,2))
True
```

#### 序列解包

与多个名字关联到一个对象不同，序列解包是展开所有元素，再分别于多个名字关联、

```
>>> a, b, c = [1, 2, 3]
>>> a, b, c
(1, 2, 3)
>>> a, b, c = "xyz"
>>> a, b, c
('x', 'y', 'z')
>>> a, b, c = range(3)
>>> a, b, c
(0, 1, 2)
>>> a, b = [1,2], (3,4)
>>> a
[1, 2]
>>> b
(3, 4)
>>> a, b = b, a   #交换对象
>>> a
(3, 4)
>>> b
[1, 2]
>>> a, ((b, c), (d, e)) = 1, [(10, 20), "ab"] #深度嵌套，左右值表达式以相同方式嵌套。
>>> a, b, c, d, e
(1, 10, 20, 'a', 'b')
>>> a, _, b, _, c = "a0b0c"  #以_来忽略某些元素
>>> a, b, c
('a', 'b', 'c')
>>> a, b = 1, 2, 3    #序列元素与变量名不等时解包错误
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: too many values to unpack (expected 2)
>>> a, *b, c = range(5)  #名字前添加星号*，表示可接纳所有多余元素
>>> a, b, c
(0, [1, 2, 3], 4)
>>> a, *b, c = 1, 2   #收集不到数据，返回空列表
>>> a, b, c
(1, [], 2)
>>> a, *b, c, d = 1, 2  # 元素少于非收集变量名数目
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: not enough values to unpack (expected at least 3, got 2)
>>> a, *b, *c, d = range(10)  # 不能出现2个或以上的星号
  File "<stdin>", line 1
SyntaxError: two starred expressions in assignment
>>> *a = 1, 2                 # 星号收集不能单独出现
  File "<stdin>", line 1
SyntaxError: starred assignment target must be in a list or tuple
>>> [*a] = 1, 2               # 星号单独出现需放在列表/元组中
>>> a
[1, 2]
>>> (*a,) = 1, 2              # 1个元祖，不要忘了逗号
>>> a
[1, 2]
>>> for a, *b in ["abc", (1,2,3)]: print(a,b) #序列解包和星号收集用于控制表达式的场景
a ['b', 'c']
1 [2, 3]
```

星号还可展开可迭代对象。所有序列类型、字典、集合、文件等都是可迭代类型。

```
>>> a = (1, 2)
>>> b = "ab"
>>> c = range(10, 13)
>>> [*a, *b, *c]
[1, 2, 'a', 'b', 10, 11, 12]
>>> d= {"a": 1, "b": 2}
>>> [*d]        # 字典中，单星号展开主键
['a', 'b']
>>> {"c": 3, **d}  # 字典中，双星号展开键值
{'c': 3, 'a': 1, 'b': 2}
```

在函数调用时，可将单个对象分解成多个实参。

```
>>> def test(a, b, c):
...     print(locals())
>>> test(*range(3))
{'a': 0, 'b': 1, 'c': 2}
>>> test(*[1,2], 3)
{'a': 1, 'b': 2, 'c': 3}
>>> a = {"a": 1, "c":3}
>>> b = {"b": 2}
>>> test(**b, **a)
{'a': 1, 'b': 2, 'c': 3}
```

#### 作用域

赋值操作默认是针对当前名字空间的。

```
>>> g = 1
>>> def outer():
...     e = 2
...     def inner():
...             global g     #声明全局变量
...             nonlocal e	 #声明外部嵌套函数变量
...             g = 10
...             e = 20
...     inner()
...     return e
...
>>> outer()
20
>>> g
10
```

可用global在函数内创建全局变量，用nonlocal则自内向外一次检索嵌套函数，但不包括全局名字空间。如多层嵌套函数存在同名变量，则就近原则处理，另外，nonlocal不能为外层嵌套函数新建变量。不同global运行期行为，nonlocal要求编译期绑定，故目标变量提前存在。

```
>>> def enclosing():
...     x = 1
...     def outer():
...             def inner():
...                     nonlocal x
...                     x = 999
...             inner()
...     outer()
...     print(x)
...
>>> enclosing()
999

>>> def inner():
...     nonlocal x
...
  File "<stdin>", line 2
SyntaxError: no binding for nonlocal 'x' found
```



### 运算符

Python运算符接近自然表达方式。但，优先级顺序和隐式转换会导致某些隐蔽错误。

```
>>> not "a" in ["a", 1]   # 应该先in后not
False
>>> (not "a") in ["a", 1]
False
>>> not ("a" in ["a", 1])  #使用括号，避免误解
False
```

![1559027907842](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1559027907842.png)

运算符优先级如上图。

标准库operator提供了特定函数或方法来实现运算符，可像普通对象那样作为参数传递。

```
>>> def calc(x, y, op):
...     return op(x, y)
...
>>> import operator
>>> calc(1, 2, operator.add)    # operator提供了很多运算方法，可通过help(operator)查看
3
```

#### 链式比较

链式比较就是将多个比较表达式组合在一起。该方法可有效缩短代码，并可提供性能以及更符合阅读习惯。

```
>>> a, b = 2, 3
>>> a > 0 and b > a and b <= 5
True
>>> 0<a<b<=5
True
```

#### 切片

切片可用于表达序列片段或整体。切片操作由3个参数构成，[start: stop: step]。以起始和结束构成半开半闭区间，不含结束位置。默认起始位置为0，结束为止为len(x)，默认步长为1。

```
>>> x = [0, 1, 2, 3, 4, 5, 6]
>>> s = x[2:5]    # 从列表复制指定范围的引用
>>> s
[2, 3, 4]
>>> x.insert(3, 100)  #修改原列表，不影响切片
>>> x
[0, 1, 2, 100, 3, 4, 5, 6]
>>> s
[2, 3, 4]
>>> x[:5]    #省略起始，以0开始
[0, 1, 2, 100, 3]  
>>> x[2:]   #省略结束，到最尾。
[2, 100, 3, 4, 5, 6]
>>> x[:]    #完成复制
[0, 1, 2, 100, 3, 4, 5, 6]
>>> x[2:6:2]  #指定步长为2
[2, 3]
>>> x[::2]  #从头到尾，步长为2
[0, 2, 3, 5]
>>> x[::-1] #从头到尾，反向复制
[6, 5, 4, 3, 100, 2, 1, 0]
>>> x[5:2:-1] #反向步进
[4, 3, 100]
>>> x[-2:-5:-1]  #用负索引表达起始，结束，反向步长。
[5, 4, 3]
>>> x[-2]      #使用负索引来直接访问序列元素
5

>>> x[3:7]
[100, 3, 4, 5]
>>> del x[3:7]   #使用切片来删除指定范围序列
>>> x
[0, 1, 2, 6]
>>> del x[::2]   #步进删除
>>> x
[1, 6]

```

除表达式为也可用itertools.islice函数来执行切片操作。

以切片方式进行序列局部赋值，相当于先删除，后插入。

```
>>> x = [0, 1, 2, 3, 4, 5, 6]
>>> x[2:6] = [10, 20]
>>> x
[0, 1, 10, 20, 6]

>>> x = [0, 1, 2, 3, 4, 5, 6]
>>> x[::2]
[0, 2, 4, 6]
>>> x[::2] = [10,20,40,60]   #如用步进插入，则删除与插入元素必须相等。
>>> x
[10, 1, 20, 3, 40, 5, 60]
>>> x[::2] = [10,20,40,60,80]   # 步进插入，删除与插入元素不等。
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: attempt to assign sequence of size 5 to extended slice of size 4

```

#### 逻辑运算

逻辑运算用于判断多条件的布尔结果，或返回有效的操作数。

分别以and 、or、not运算符表示逻辑与、或、非三种关系。

and返回最后或导致短路的操作数；or返回第一真值，或最后操作数。

注意：相同逻辑运算符一旦短路，后续计算被终止。

```
>>> 1 and 2  # 最后操作数
2
>>> 1 and 0 and 2 #导致短路的操作数
0
>>> 1 or 0   #第一真值
1
>>> 0 or 1 or 2  #第一真值
1
>>> 0 or []  # 最后操作数
[]

>>> def x(o):
...     print("op:", o)
...     return o
...
>>> x(0) and x(1)  # 0 导致短路，后续不计算
op: 0
0
>>> x(1) or x(2)  # 返回1真值，后续不计算
op: 1
1
>>> x(0) and x(1) or x(2)  #不同运算符须多次计算。
op: 0
op: 2
2
```

##### 条件表达式

这里介绍三元运算符。T if X else F：当条件X为真时，返回T，否者返回F。等同X?T:F。

```
>>> "T" if 2>1 else "F"
'T'
>>> 2 > 1 and "T" or "F"
'T'
>>> 2 < 1 and "T" or "F"
'F'
>>> 2 < 1 and "T"   #a.分解上面运算符。2<1导致短路
False
>>> False or "F"    #b.分解上面运算符。返回真值"F"
'F'
>>> 2 > 1 and "" or "F"   #T为假时，or必须返回最后操作数，这与预期不符。
'F'
>>> "" if 2 > 1 else "F"    #当T和F时动态数据时，使用表达式会更安全点。
''

>>> x = None 
>>> y = x or 100    #可简化设定默认值
>>> y = x and x*2 or 100  #可简化设定默认值
```



### 推导式

推导式就是整合for、if语句来构造列表、字典和集合对象。

```
>>> [x for x in range(5)]  #列表
[0, 1, 2, 3, 4]
>>> [x+10 for x in range(10) if x % 2 == 0]   #条件过滤后的列表
[10, 12, 14, 16, 18]

>>> {k:v for k, v in zip("abc", range(10,13))}  #字典，使用大括号
{'a': 10, 'b': 11, 'c': 12}
>>> {x for x in "abc"}   #集合，使用大括号
{'c', 'a', 'b'}

>>> [f"{x} {y}" for x in "abc" if x!="c"   #多嵌套使用推导式，可用多个for子句，每个子句可选一个if条件表达式。
...             for y in range(3) if y !=0 ]
['a 1', 'a 2', 'b 1', 'b 2']
```

和普通循环语句不同，推导式临时变量不影响上下文名字空间。推导式在编译器中会生成函数，其变量在自动生成函数内使用，故不会影响上下文。

```
>>> def test():
...     a="abc"
...     data = {a:b for a,b in zip("xyz",range(10,13))}  #这里a,b是临时变量
...     print(locals())
...
>>> test()
{'a': 'abc', 'data': {'x': 10, 'y': 11, 'z': 12}}
```

注意：推导式中使用小括号，并非创建元组，而是创建生成器对象。区别于推导式，称为生成器表达式。

```
>>> (x for x in range(3))
<generator object <genexpr> at 0x00000213A1770A20>
```

