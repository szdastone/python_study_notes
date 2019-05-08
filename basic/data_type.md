# 数据类型

## 1、字符串

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



### 1.1 字符串操作

| 操作   | 说明                 | 示例                                     |
| ------ | -------------------- | ---------------------------------------- |
| title  | 将字符串首字符大写   | "hello world".title()<br />=>Hello World |
| lower  | 将字符串改为小写     | "Hello World".lower()<br />=>hello world |
| upper  | 将字符串改为大写     | "hello world".upper()<br />=>HELLO WORLD |
| rstrip | 删除字符串右侧的空格 | "  Python  ".rstrip()<br />=>' Python'   |
| lstrip | 删除字符串左侧的空格 | "  Python  ".lstrip()<br />=>'Python '   |
| strip  | 删除字符串2侧的空格  | "  Python  ".strip()<br />=>'Python'     |



### 1.2 字符串合并

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

### 1.3 其他

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

## 2、数字

### 2.1 整数

可处理任意大小整数，并可使用加(+)减(-)乘(*)除-(/)来运算。可以用python当计算器做各种运算。

```
>>> 3*3
9
>>>3**3
27
>>>(2+10)*2-(10+3)
11
```

### 2.2 浮点数

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