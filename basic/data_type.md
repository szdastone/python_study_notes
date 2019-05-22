# 数据类型

Python内置的基本数据类型可简单分为数字、序列、映射和集合基类。又根据其实例是否可修改，分可变和不可变。

| 名称      | 分类     | 可变类型 |
| --------- | -------- | -------- |
| int       | number   | N        |
| float     | number   | N        |
| str       | sequence | N        |
| bytes     | sequence | N        |
| bytearray | sequence | Y        |
| list      | sequence | Y        |
| tuple     | sequence | N        |
| dict      | mapping  | Y        |
| set       | set      | Y        |
| frozenset | set      | N        |

标准库collections.abc可列出相关类型的抽象基类，及是否可变。

```
>>>import collections.abc
>>>issubclass(str,collections.abc.Sequence)
True
>>>issubclass(str,collections.abc.MutableSequence)
False
```





## 字符串

字符串就是一系列字符，可用引号括起来的都是字符串。引号可以为单引号，双引号。可灵活的使用单引号与双引号。如果字符串包含有单双引号，可使用转义符(\\\)。当然，转义符也可以用来转义其他字符，比如制表符\t,换行符\n，但如果要使用\\，则需要使用\\\来表示！

比如：

```
"this is a string"
'this is also a string'
'hello, "python"！'
"i'm a good boy!"
'I\'m \"OK\"'
'\tI love python!\n\n \\but python don\'t easy!'
```

如果字符串有很多的\\ ,为了简化，可以使用r''来表示''内部字符串不转义，如

```
>>>print('\\\t\\')  #\	\
>>>print(r'\\\t\\') #\\\t\\
```

如果有太多的换行\n，或者每行的字符太长有多行时，可使用'''...'''来输入多行内容，比如：

```
>>>print('''line1
...line2
...line3''')
line1
line2
line3
```



###  字符串操作

| 操作   | 说明                 | 示例                                     |
| ------ | -------------------- | ---------------------------------------- |
| title  | 将字符串首字符大写   | "hello world".title()<br />=>Hello World |
| lower  | 将字符串改为小写     | "Hello World".lower()<br />=>hello world |
| upper  | 将字符串改为大写     | "hello world".upper()<br />=>HELLO WORLD |
| rstrip | 删除字符串右侧的空格 | "  Python  ".rstrip()<br />=>' Python'   |
| lstrip | 删除字符串左侧的空格 | "  Python  ".lstrip()<br />=>'Python '   |
| strip  | 删除字符串2侧的空格  | "  Python  ".strip()<br />=>'Python'     |



###  字符串合并

字符串可使用+来拼接，比如：

```
>>>s = "Hello"
>>>s = s + ' '+'World'
>>>print(s)
```

字符串合并也可混合运算符，比如+为合并，*为重复

```
>>>3*'un'+'ium'
'unununium'
```

2个或多个字符串会自动合并，如:

```
>>>'Hello' ' ' 'Py' 'thon'
Hello Python
```

###  其他

要输入多行怎么办呢？可使用（）来输入多行，如：

```
>>> text = ('Put several strings within parentheses 
...         'to have them joined together.')
>>> text
'Put several strings within parentheses to have them joined together.'
```

字符串其实就是一个元组，可使用元组的一些特性，如：

```
>>>word='Python'
>>>word[0]
'P'
>>>word[-1]
'n'
>>>word[0:2]
'Py'
>>>word[-2:]
'on'
>>>J+word[1:]
'Jython'
```

内置函数len()表示字符串的长度

```
>>>S='abdde'
>>>len(S)
5
```

## 数字

### 整数

在Python3中，将int、long这两种类型合并为int,采用变长结构。可处理任意大小整数，并可使用加(+)减(-)乘(*)除-(/)来运算。可以用python当计算器做各种运算。

```
>>> 3*3
9
>>>3**3
27
>>>3/2    #单斜线为True Division，无论是否整除，总返回浮点数
1.5
>>>3 // 2   #双斜线为False Division，会截掉小数部分，仅返回整数结果
1
>>>(2+10)*2-(10+3)
11
>>>78_654_321      #由于逗号表示tuple语法，可使用下划线便是分割千位位数
78654321
>>>5%2   #取模运算符(mod)
1
>>>divmod(99,5)
(19,4)
```

除了十进制，还可以二进制、八进制、十六进制表示。下划线分隔符号也适用这些进制的字面量。

```
>>>0b110011   #bin 
51
>>>0o12       #oct
10
>>>0x64      #hex
100
>>>0b_11001_1  
51
```

可用内置函数将整数转换为指定进制字符串，或使用int还原，int函数默认为十进制，会忽略空格、制表符等，如指定进制，则可省略相关进制前缀。

```
>>>int("0b1100100",2)
100
>>>int("0o144",8)
100
>>>int("0x64",16)
100
>>>int("64",16)
100
>>>int("  100\t  ")
100
```

如使用eval也可完成转换，但性能要差很多。

将整数转换为字节数组时需要指定目标字节数组大小，而整数类型是变长结构，故通过二进制位长度来计算，另可通过sys.byteorder来获取当前系统字节序。

```
>>>x=0x1234
>>>n=(x.bit_length()+8-1)//8  #计算按8位对齐所需的字节数
>>>b=x.to_bytes(n,sys.byteorder)
>>>b.hex()
'3412'
>>>hex(int.from_bytes(b,sys.byteorder))
'0x1234'
```

##### 布尔

布尔为整数子类型。True，False可当做整数直接使用。

```
>>>True == 1
True
>>> False == 0
True
>>>issubclass(bool,int)
True
>>>isinstance(True,int)
True
>>>True+1
2
```

在进行布尔转换时，数字零、空值(None)、空序列和空字典都为False，反之为True。

```
>>>data=(0,0.0,None,"",list(),tuple(),dict(),set(),frozenset())
>>>any(map(bool,data))
False
```

##### 枚举

Python中没有枚举类型的定义，但可通过标准库来实现。首先定义枚举类型，随后由内部代码生成枚举值实例。

```
>>>import enum
>>>Color = enum.Enum("Color","BLACK YELLOW BLUE RED")
>>>isinstance(Color.BLACK, Color)
True
>>>list(Color)
[<Color.BLACK: 1>, <Color.YELLOW: 2>, <Color.BLUE: 3>, <Color.RED: 4>]
```

枚举值不一定为整数，可通过继承，将其指定为任一类型。

```
>>>class X(enum.Enum):
	A="a"
	B=100
	C=[1,2,3]
	
>>>X.C
<X.C: [1, 2, 3]>
```

枚举类型的内部以字典方式实现，每个枚举值都有name和value属性。可通过名字或值查找对应枚举实例。按字典规则，值可相同，但名字不可重复。如要避免相同枚举定义，可用enum.unique装饰器。

```
>>>X.B.name
'B'
>>>X.B.value
100
>>>X["B"]
<X.B: 100>
>>>X([1,2,3])
<X.C: [1, 2, 3]>
```

##### 内存

在Python中，小数字都会预先缓存，一般缓存范围为[-5,256]。如超出这个范围，则要新建对象，并内存分配等。

```
>>>a=-5
>>>b=-5
>>>a is b
True
>>>a=257
>>>b=257
>>>a is b
False
```



### 浮点数

带小数的就是浮点数。浮点数的小数点位置是可以变的。比如$1.23*10^9$和$12.3*10^8$是相等的。浮点数可使用数学写法，如:1.23，3.14，-9.01，但也可使用科学计数法，如$1.23*10^9$就是1.23e9，0.000012可写成1.2e-5，等等。

```
>>>tax=12.5/100
>>>price=100.50
>>>price*tax
12.5625
>>>price+_
113.0625
>>>round(_,2)
113.06
```

在交互模式下，变量(_)代表了最后一个表达式。