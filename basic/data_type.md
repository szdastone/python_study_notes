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

本部分有关列表、元组、字典等另开章节来讨论，本部分仅介绍字符串、数字、字节数组等。



## 字符串

字符串就是一系列字符，可用引号括起来的都是字符串，存储Unicode文本，是不可变序列类型。

```
>>>s = "中国"
>>>len(s)
2
>>>hex(ord("中"))
'0x4e2d'
>>>chr(0x4e2d)
'中'
>>>ascii(“中国”)
"'\\u4e2d\\u56fd'"
>>>"h\x69, \u6c49\U00005B57"     #\u表示16位unicode，\U为32位
'hi, 汉字'
```



引号可以为单引号，双引号。可灵活的使用单引号与双引号。如果字符串包含有单双引号，可使用转义符(\\\)。当然，转义符也可以用来转义其他字符，比如制表符\t,换行符\n，但如果要使用\\，则需要使用\\\来表示！

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

多个动态字符串拼接，优先选择join或format方式。

```
>>>tmpl = "/data/{user}/message/{time}.txt"
>>>tmpl.format(user="qyuhen",time="2017010")
'/data/qyuhen/message/2017010.txt'

>>>import string
>>>x = list(string.ascii_uppercase)
>>>"".join(x)                      #join比以+来拼接的性能要好的多
'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
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
>>>word[0:2]   #切片
'Py'
>>>word[-2:]
'on'
>>>J+word[1:]
'Jython'
>>>s="-"*1024
>>>s1=s[10:100] #片段
>>>s2=s[:]      #复制，内容同
>>>s3=s.split(",")[0]   #分割，内容同
```

使用in、not in 判断是否包含子串

```
>>>"py" not in "Python" 
True
```

内置函数len()表示字符串的长度

```
>>>S='abdde'
>>>len(S)
5
```

##### 转换

除了与数值、unicode转换外，还有不同编码之间转换。Python3使用bytes、bytearray存储字节序列。如要处理BOM信息，需导入codecs模块。

```
>>>str(1.23e10)
'12300000000.0'
>>>s="汉字"
>>>b=s.encode("utf-8")
>>>print(b)
b'\xe6\xb1\x89\xe5\xad\x97'
>>>b.decode("utf-8")
'汉字'
>>>import codecs
>>>codecs.BOM_UTF16_LE.hex()
'fffe'
>>>codecs.encode(s,"utf-16be").hex()  #按指定BOM转换
'6c495b57'
>>>codecs.encode(s,"utf-16le").hex()
'496c575b'
>>>import sys
>>>sys.getdefaultencoding()
'utf-8'
```

##### 格式化

f-strings的支持，使用f前缀标志，解释器解析大括号内的字段或表达式，在上下文名字空间查找同名对象进行替换。格式化遵循format规范。

```
>>>x = 10
>>>y = 20
>>>f"{x}+{y} = {x=y}"   #f-strings
'10+20=30'
>>>"{}+{}={}".format(x,y,x+y)  #str.format
'10+20=30'
>>>f"{type(x)}"      #函数调用
"<class 'int'>"
```

完整的format格式化以位置序号或字段名匹配参数进行替换，可添加对齐、填充、精度等控制。从某种角度看，f-strings类似format的增强语法糖。

* 手工序号和自动序号

  ```
  >>>"{0} {1} {0}".format("a", 10)
  'a 10 a'
  >>>"{} {}".format(1, 2)   #自动序号
  '1 2'
  ```

* 主键

  ```
  >>>"{x} {y}".format(x =100, y= [1, 2, 3])
  '100 [1, 2, 3]'
  ```

* 属性和索引

  ```
  >>>x.name = "jack"
  >>>"{0.name}".format(x)       #对象属性
  'jack'
  >>>"{0[2]}".format([1,2,3,4])  #索引
  '3'
  ```

* 宽度、补位

  ```
  >>>"{0:#08b}".format(5)
  '0b000101'
  ```

* 数字

  ```
  >>>"{:06.2f}".format(1.234)
  '001.23'
  ```

* 对齐

  ```
  >>>"[{:^10}]".format("abc")              #居中
  '[   abc    ]'
  >>>"[{:.<10}]".format("abc")           #左对齐，以点填充
  '[abc.......]'
  ```

##### 池化

字符串是进程实例数量对多的类型之一，为节约内存，且可省去创建新实例开销，可对内容相同，且不可变的变量名进行池化。在Python中就是实现一个字符串池（intern)，这样好处就是池负责管理实例，只需引用即可使用；从池中返回字符串，只需比较指针就可知内容是否相同，以提高性能。但一旦失去外部引用，池内字符串对象可能会被回收

```
>>>a="abc"
>>>b="abc"
>>>a is b
False
>>>sys.intern(a) is sys.intern("abc")  
True
>>>a = sys.intern("hello,world!")   
>>>id(a)
2811852698032
>>>id(sys.intern("hello,world!"))
2811852698032
>>>del a                   #删除外部引用后被回收
>>>id(sys.intern("hello,world!"))  #从id值不同可看到是新建后入池的
2811852698224
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

带小数的就是浮点数，可表达16到17个小数位。浮点数的小数点位置是可以变的。比如$1.23*10^9$和$12.3*10^8$是相等的。浮点数可使用数学写法，如:1.23，3.14，-9.01，但也可使用科学计数法，如$1.23*10^9$就是1.23e9，0.000012可写成1.2e-5，等等。

```
>>>tax=12.5/100
>>>price=100.50
>>>price*tax
12.5625
>>>price+_
113.0625
>>>round(_,2)
113.06
>>>0.1234567890123456789
0.12345678901234568
```

在交互模式下，变量(_)代表了最后一个表达式。

如对精度有严格要求，则应该固定精度类型。可通过float.hex方法输出实际存储值的十六进制格式字符串。也可通过该方法实现浮点值精度传递，避免精度丢失。

```
>>>0.1*3 == 0.3
False
>>>(0.1*3).hex()
'0x1.3333333333334p-2'
>>>(0.3).hex()
'0x1.3333333333333p-2'
>>>s=(1/3).hex()
>>>float.fromhex(s)
0.3333333333333333
```

可固定精度情况下进行比较操作，比如：

```
>>>round(0.1*3, 2)==round(0.3, 2) #避免不确定性，左右都使用固定精度
True
>>>round(0.1, 2)*3 == round(0.3, 2) #round返回值作为操作数会导致精度丢失
False
```

##### 转换

将整数或字符串转换成浮点数很简单，使用内置float函数即可。如超出了有效精度，结果会有差异。

```
>>>float("\t 100.123\n ")  #会自动处理空白符
100.123
>>>float("1.234E2")  #科学计数
123.4
>>>float("0.1234567890123456789")
0.12345678901234568
```

如浮点数转为整数，可使用内置函数int或标准库函数trunc,floor,ceil等

```
>>>int(2.6),int(-2.6)
(2,-2)
>>>from math import trunc,floor,ceil
>>>trunc(2.6),trunc(-2.6)  #截取小数部分
(2,-2)
>>>floor(2.6),floor(-2.6)   #往小数字方向去最近整数
(2, -3)
>>>ceil(2.6),ceil(-2.6)  #往大数字方向取最近整数
(3, -2)
>>>round(2.6),round(-2.6) #四舍五入
（3， -3）
```

decimal.Decimal可提供最高28位有效精度，而且不存在近似值问题。

```
>>>1.1+2.2
3.3000000000000003
>>>from decimal import Decimal
>>>Decimal("1.1")+Decimal("2.2")
Decimal('3.3')
>>>Decimal(0.1)  #必须传入准确数值，比如整数或字符，如传入float则精度会丢失
Decimal('0.1000000000000000055511151231257827021181583404541015625')
```

除非有需求，不建议用Decimal替代float，这样的性能会差很多。可使用getcontext或localcontext来修改Decimal默认的28位精度。

```
>>>from decimal import Decimal,getcontext,localcontext
>>>getcontext()
Context(prec=28, rounding=ROUND_HALF_EVEN, Emin=-999999, Emax=999999, capitals=1, clamp=0, flags=[FloatOperation], traps=[InvalidOperation, DivisionByZero, Overflow])
>>>getcontext().prec = 2    #根据上下文修改全局的精度
>>>Decimal(1) / Decimal(3)
Decimal('0.33')
>>> with localcontext() as ctx:    #localcontext限制某个区域精度
		ctx.prec =3
		print(getcontext().prec)
		print(Decimal(1)/ Decimal(3))
		
3
0.333
```

##### 四舍五入

因为近似值和精度问题，使用float进行四舍五入会出现操作的不确定性。

```
>>>round(0.5)
0
>>>round(1.5)
2
```

由于不确定，可改用Decimal，按需求选取可控的进位。

```
>>>from decimal import Decimal, ROUND_HALF_UP
>>>def roundx(x,n):
		return Decimal(x).quantize(Decimal(n),ROUND_HALF_UP)

>>>roundx("0.5",1)
Decimal('1')
>>>roundx("2.675",".01")
Decimal('2.68')
```

## 字节数组

从底层实现来看，所有数据都是二进制字节序列，在Python2引入了bytearray字节数组，在Python3新增了只读版本的bytes。字节数组为不可变序列类型。bytes与str有类似的操作。和bytes一次性内存分配相比，bytearray可按需扩张，同样也支持加法、乘法等运算符。

```
>>>b"abc"
b'abc'
>>>bytes("汉字","utf-8")
b'\xe6\xb1\x89\xe5\xad\x97'
>>>a=b"abc"+b"def"
>>>a
b'abcdef'
>>>a.startswith(b"abc")
True
>>>a.upper()
b'ABCDEF'
>>>b=bytearray(b"ab")
>>>len(b)
2
>>>b.append(ord("c"))
>>>b.extend(b"de")
>>>b
bytearray(b'abcde')
>>>b=b+b"123"+b"78"*3
>>>b
bytearray(b'abcde123787878')
```

#### 内存视图

Python没有指针的概念，另有内存安全模型的限制。要借助名为内存视图的方式来访问底层内存数据。内存视图要求目标对象支持缓冲协议。它直接引用目标内存，没有额外复制行为。因此，可读取最新数据。在目标对象允许下，还可执行写操作。常见支持视图操作的有bytes、bytearray、array.array及NumPy的某些类型。

```
>>>a = bytearray([0x10,0x11,0x12,0x13,0x14,0x15,0x16])
>>>v = memoryview(a)   #完整视图
>>>x = v[2:5]          #视图片段
>>>x.hex()
'121314'
>>>a[3]=0xee          #修改原数据，可通过视图观察到
>>>x.hex()
'12ee14'
>>>x[1]=0x13         #因引用相同内存区域，可通过视图修改原数据
>>>a
bytearray(b'\x10\x11\x12\x13\x14\x15\x16')
```

视图片段有自己索引范围，注意不要超出限制。是否可通过视图修改数据，需要看原对象是否允许。不如bytes就不可修改。

```
>>>a = b"\x10\x11"
>>>v = memoryview(a)
>>>v[1] =0xEE
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: cannot modify read-only memory
```

如果复制视图数据，可用tobytes、tolist等方法。复制后与原对象无关，不会影响视图本身。

```
>>>a = bytearray([0x10,0x11,0x12,0x13,0x14,0x15,0x16])
>>>v = memoryview(a)   #完整视图
>>>x = v[2:5]          #视图片段
>>>b = x.tobytes()     #复制并返回视图数据
>>>b
b'\x12\x13\x14'
>>> a[3]=0xff         #不影响复制数据
>>>b
b'\x12\x13\x14'
```

