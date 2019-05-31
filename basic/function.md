# 函数

函数就是带名字的代码块，用来完成一些具体的事情。要执行函数定义的特定任务时，就要调用该函数。定义函数是为了在程序中能多次调用，而无需反复编写代码。使用函数，会让程序编写、阅读、测试、DEBUG都变得容易。

函数应该减少依赖关系，具备良好的可测试性和可维护性。

#### 定义函数

函数由两部分构成：代码对象持有字节码和指令元数据，负责执行；函数对象则为上下文提供调用实例，并管理所需的状态数据。下面定义一个简单函数，如：

```
>>> def greet(user="Python"):
...     s = "Hello "
...     print(s+user.title())
...
>>> greet   # 函数对象
<function greet at 0x000001D7243EC1E0>
>>> greet.__code__  #代码对象
<code object greet at 0x000001D72443D780, file "<stdin>", line 1>
>>> greet.__code__.co_varnames  #参数及变量名列表
('user', 's')
>>> greet.__code__.co_consts    #指令常数
(None, 'Hello ')
>>> greet.__defaults__          #参数默认值  
('Python',)
>>> greet('Python')			    #调用函数
Hello Python
>>> greet.__defaults__ =('world',)  #修改默认值
>>> greet()
Hello World
>>> greet.abc="hello,world"   #为函数实例添加属性
>>> greet.__dict__            #函数字典
{'abc': 'hello,world'}
```

上面函数传递了一个参数，那么在函数greet中，user就是参数，也可以称为形参。而调用greet('Python')中的’Python‘就是实参。greet('Python')就是将实参'Python'传递给函数greet中的形参user中。

代码对象相关属性有编译器生成，为只读模式。存储指令运行所需相关信息，如源码行、指令操作数，以及参数和变量名等。而函数对象作为外部实例存在，负责管理运行期状态，比如参数默认值及动态添加属性等。注意：def是运行期指令。以代码对象为参数，创建函数实例，并在当前上下文中与指定名字关联。故可用def以单个代码对象为模板创建多个函数实例，如下：

```
>>> def make(n):
...     ret = []
...     for i in range(n):
...             def test(): print("hello")   # test = make_function(code)
...             print(id(test), id(test.__code__))
...             ret.append(test)
...     return ret
...
>>> make(3)                    
2023541025920 2023538776224     #不同实例，相同代码
2023541026192 2023538776224
2023541026600 2023538776224
[<function make.<locals>.test at 0x000001D72471AC80>, <function make.<locals>.test at 0x000001D72471AD90>, <function make.<locals>.test at 0x000001D72471AF28>]
```

> 多个实例，和多个名字引用同一个实例，不是一回事。用列表持有多个实例，阻止临时变量test被回收，避免因内存复用而出现相同id值。
>

在名字空间里，名字仅能与单个目标相关联。因此，就无法实现函数重载。另外，作为第一类对象，函数可作为参数和返回值传递。

```
>>> def test(op, x, y):  
...     return op(x, y)
...
>>> def add(x, y):
...     return x+y
...
>>> test(add, 1, 2)   #将函数作为参数
3

>>> def test():
...     def hello():
...             print("hello world")
...     return hello   #将函数作为返回值    
...
>>> test()
<function test.<locals>.hello at 0x000001D72471AD90>
>>> test()()       
hello world

>>> def test():     #支持函数嵌套，甚至可与外层函数同名。内外函数虽同名，但属于不同名字空间
...     print("outer test")
...     def test():
...             print("inner test")
...     return test
...
>>> x = test()
outer test
>>> x
<function test.<locals>.test at 0x000001D72471AC80>
>>> x()
inner test
```

##### 匿名函数

匿名函数的正式名称为lambda表达式。

相比普通函数，匿名函数内容只能是单个表达式，而不能使用语句，也不能提供默认函数名。

```
>>> add = lambda x, y: x+y
>>> add
<function <lambda> at 0x000001D72471AD90>
>>> add(1,2)
3
```

普通函数有个默认名字(\_\_name\_\_)，用来标识真实身份。该名字是编译期静态绑定的，与运行期的变量名引用无关。

```
>>> def test(): pass
...
>>> a = test
>>> a.__name__
'test'
>>> a
<function test at 0x000001D724732158>
>>> test = lambda: None
>>> a = test
>>> a.__name__
'<lambda>'
>>> a
<function <lambda> at 0x000001D72471AF28>
```

在某些合适场合，lambda比普通函数更灵活与自由。

```
>>> map(lambda x: x**2, range(3))  #直接作为参数
>>> ops = {                        #构建方法表
...     "add": lambda x, y: x+y,
...     "sub": lambda x, y: x-y,
... }
>>> ops["add"](2,3)
5
>>> def make(n):                               # 作为推导式输出结果
...     return [lambda: print("hello") for i in range(n)]

#lambda支持嵌套，还可直接调用。
>>> test = lambda x: (lambda y: x+y)      #将另一个lambda作为返回值，支持闭包
>>> add= test(2)
>>> add(3)
5
>>> (lambda x: print(x))("hello")        #使用括号避免语法错误
hello
```




#### 参数

上面简单解释了实参与形参。那么如果有多个参数怎么传递呢？调用函数时，每个实参都需要关联到函数定义的形参中，最简单的关联方式就是基于实参的位置。比如：

```
def desc_pet(type,name):
	print("宠物类型为："+type)
	print("宠物名称为："+name)
	
desc_pet('hamster','harry')
desc_pet('cat','jerry')
```

注意：函数调用实参顺序和函数定义的形参顺序要一致，不然就会错误。

如果不关注参数位置，则可通过形参关键字来调用。比如：

```
desc_pet(name='micky',type='mouse')
```

使用关键字实参时，必须要准确的指定函数定义中的形参名。

定义函数时，还可以指定默认值，如不传入实参，则按缺省值来提供。比如：

```
def desc_pet(name,type='dog'):
	print("宠物类型为："+type)
	print("宠物名称为："+name)
	
desc_pet('harry')	
```

有默认值的形参最好放在最后，这样进行位置传送时就不宜出错，不然要就需要指定形参名。

> 形参出现在函数定义的参数列表中，可视作函数局部变量，仅能在函数内部使用。而实参由调用方提供，通常以复制方式将值传递给形参。形参在函数调用后销毁，而实参则受调用方作用域影响。不同形参以变量形式存在，实参可是变量、常量、表达式等，总之有确定值可供复制传递。不管实参是名字、引用，还是指针，其都以值复制方式传递，随后的形参变化不会影响实参。当然，对该指针或引用目标的修改，与此无关。

```
>>> def test(a, b, c =3):
...     print(locals())
...
>>> test(1,2)   # 忽略有默认值参数
{'a': 1, 'b': 2, 'c': 3}
>>> test(1,2,30)  #为默认值参数提供实参
{'a': 1, 'b': 2, 'c': 30}
>>> test(*(1,2,30))  #星号展开
{'a': 1, 'b': 2, 'c': 30}
>>> test(b =2, a=1)  #命名方式传递，可不理会参数顺序
{'a': 1, 'b': 2, 'c': 3}
>>> test(**{"b":2, "a":1})   #键值展开后，等同命名传递
{'a': 1, 'b': 2, 'c': 3}
>>> test(1, c=30, b =2)    #命名与顺序混用时，必须顺序在命名参数前
{'a': 1, 'b': 2, 'c': 30}
```

##### 传递列表参数

形参可接收任何类型，如传入列表，字典等。如果传入列表后，希望不能修改列表怎么处理呢？那么传入实参时，可传入实参的副本。比如:

```
def love_pets(pets):
	for pet in pets:
		print(pet)

pets = ['dog','cat','goldfish']
love_pets(pets) #传入的实参可在函数中修改
love_pets(pets[:]) #传入实参列表的副本到函数，不影响实参。
```

向函数传递列表副本可保留原始列表的内容，但除非有充分理由需要传递副本，否则还是传递原始列表给函数，因为函数使用现成列表可避免花时间和内存来创建副本，从而提供效率。

##### 任意数量参数

有些函数定义时并不知道有多少实参，那么函数定义形参时加入*，表示可接收任意数量参数，也可称为收集参数。比如：

```
def make_pizza(*toppings):
	print(toppings)

make_pizza('pepperoni')
make_pizza('mushrooms','green peppers','extra cheese')
```

如函数接收不同类型实参，那么可接纳任意数量实参的形参必须放在最后，比如:

```
def make_pizza(size,*toppings):
	print(size)
	print(toppings)

make_pizza(16,'pepperoni')
make_pizza(12,'mushrooms','green peppers','extra cheese')
```

单星号收集参数，只能一个。但普通位置参数以及默认值位置参数可0到多个。收集参数也不可命名传递。

有时，需要接收任意数量的实参，而且不知道会传递给函数什么参数。这时，可将函数编写成接收任意数量键值对，使用**来标识。也称为键值收集参数。比如:

```
def build_profile(first,last,**user_info):
	profile ={}
	profile['first_name']=first
	profile['last_name']=last
	for key, value in user_info.items():
		profile[key]=value
	return profile
    
user_profile = build_profile('albert','lee',location='hubei',field='physics')
print(build_profile)
```

编写函数时，可以各种方式混合使用位置实参，关键字实参和任意数量参数。

Python3中新增了名为keyword-only的键值参数类型 。

* 以星号与位置参数列表分割边界；

* 普通keyword-only参数，0到多个；

* 有默认值的keyword-only参数，0到多个；

* 双星号键值收集参数，仅1个。

无默认值的keyword-only必须显示命名传参，否则视为普通位置参数。

```
>>> def test(a,b,*,c):
...     print(locals())
...
>>> test(1,2,3)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: test() takes 2 positional arguments but 3 were given
>>> test(1,2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: test() missing 1 required keyword-only argument: 'c'
>>> test(1,2,c=3)
{'a': 1, 'b': 2, 'c': 3}
```

即使没有位置参数，keyword-only也必须按规则传递。

```
>>> def test(*,c):pass
...
>>> test(1)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: test() takes 0 positional arguments but 1 was given
>>> test(c=1)
```

除单星号外，位置收集参数(*args)也可作为边界。但只取其一，不能同时出现。

```
>>> def test(a, *args, c, d=99, **kwargs):
...     print(locals())
...
>>> test(1, 2, 3, c=88, x=10, y=20)
{'a': 1, 'c': 88, 'd': 99, 'args': (2, 3), 'kwargs': {'x': 10, 'y': 20}}
```

同样不能对键值收集参数命名传参。否则，会被当做普通参数收集。

```
>>> def test(**kwargs):
...     print(kwargs)
...
>>> test(kwargs={"a":1, "b":2})    #被当着普通键值参数收集，kwargs["kwargs"]
{'kwargs': {'a': 1, 'b': 2}}
>>> test(**{"a":1, "b":2})         #这样才正确
{'a': 1, 'b': 2}
```

> 静态局部变量可在不用外部变量的情况下维持函数状态。比如，用来存储调用计数等。但相比于参数默认值，正确做法是为函数创建一个状态属性。毕竟变量为函数内部使用，而参数属于对外接口。所创建的属性与函数对象的生命周期相同，不会调用结束就终结。
>
> ```
> >>> def test():
> ...     test.__count__ = hasattr(test,"__count__") and test.__count__ + 1 or 1
> ...     print(test.__count__)
> ...
> >>> test()
> 1
> >>> test()
> 2
> ```
>

##### 默认值

参数默认值允许省略实参传值。默认值在函数对象创建生成，保存在\_\_defaults\_\_中，为每次调用共享。

```
>>> def test(a, x =[1,2]):
...     x.append(a)
...     print(x)
...
>>> test.__defaults__
([1, 2],)
>>> test(3)
[1, 2, 3]

>>> def test(a, x = None):   如默认值为不可变类型，建议以None表示可忽略
...     x = x or []   # 忽略时，主动新建
...     x.append(a)
...     return x
...
>>> test(1)
[1]
>>> test(3,[1,4])  #提供非默认值实参
[1, 4, 3]
```

##### 形参赋值

解释器对形参赋值的过程如下。

1、按顺序对位置参数赋值；

2、按命名方式对指定参数赋值；

3、收集多余的位置参数；

4、收集多余的键值参数；

5、为没有赋值的参数设置默认值；

6、检查参数列表，确保非收集参数都已经赋值。

> 收集参数args、kwargs属于习惯性命名，非强制要求

而实参也有一定规则：

* 无默认值参数，必须有实参传入；
* 键值参数总是以命名方式传入；
* 不能对同一参数重复传值。

如设计显示函数如下：

```
def printx(*objects, sep =",", end="\n")
```

> 参数分2部分，待显示对象为主，可省略显示设置为次。待显示对象为外部资源，设置项用于控制函数自身行为。

用Python3的keyword-only参数则可解决此问题；

```
>>> def printx(*objects, sep=",", end ="\n"):
...     print(locals())
...
>>> printx(1,2,3)
{'sep': ',', 'end': '\n', 'objects': (1, 2, 3)}
>>> printx(1,2,3,sep="|")
{'sep': '|', 'end': '\n', 'objects': (1, 2, 3)}
```



#### 返回值

函数可以处理后返回一个或一组值。函数返回的值称为返回值。必须使用return来返回，一旦返回，则函数停止执行。

```
def format_name(first_name,last_name):
	return fist_name+' '+last_name

print(format_name('jack','ma'))
```

如果上面函数需要提供中间名，而中间名并非必须呢？也就是希望参入的实参是可选的，那上面函数可修改为：

```
def format_name(first_name,last_name, mid_name=''):
	return fist_name+' '+mid_name+ ' '+last_name

print(format_name('john','lee','hooker'))
print(format_name('jack','ma'))
```

如果要将返回值返回为一个更复杂的类型，比如字典呢？那上面函数可修改为：

```
def format_name(first_name,last_name, mid_name=''):
	person = {'first':first_name,'last':last_name,'mid':mid_name,}
	return person
```

从实现角度看，只要返回数量多于一，编译器就将其打包成元组对象。

```
>>> def test(a, b):
...     return "hello", a +b
...
>>> x =test(1,2)
>>> type(x)
<class 'tuple'>
>>> x
('hello', 3)

>>> def test(): pass    # 默认返回None
...
>>> x = test()
>>> type(x)
<class 'NoneType'>
>>> x
>>> def test(): return  # 默认返回None
...
>>> x = test()
>>> type(x)
<class 'NoneType'>
>>> x
```

#### 作用域

在函数内访问变量，会以特定顺序依次查找不同层次的作用域。

```
>>> import builtins
>>> builtins.B="B"
>>> G="G"
>>> def enclosing():
...     E="E"
...     def test():
...             L="L"
...             print(L,E,G,B)
...     return test
...
>>> enclosing()()
L E G B
```

以上可称为LEGB规则。

> LEGB含义解释：
>  L-Local(function)；函数内的名字空间
>  E-Enclosing function locals；外部嵌套函数的名字空间(例如closure)
>  G-Global(module)；函数定义所在模块（文件）的名字空间
>  B-Builtin(Python)；Python内置模块的名字空间

名字空间也叫命名空间，简单点说，命名空间是对变量名的分组划分，在Python中一切皆对象，所以变量名就是字符串对象。如

```
>>> a = 10   # 建立字符串对象a与Number对象10之间对应映射关系。
>>> locals()   # 上面变量名以键值形式来表示！
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'a': 10}
```

从上可看出命名空间就是对许多键值对分组划分，及键值对的集合，总结下就是：Python的命名空间是一个字典，字典保存了变量名称与对象之间的映射关系。

那么查找变量名其实就是在命名空间字典中查找键值对。Python中有多个命名空间，因此，需要有规则来规定，按怎样顺序来查找命名空间。而LEGB规则就是来规定命名空间查找顺序的规则。

> ​	LEGB规定了查找一个名称的顺序为：local-->enclosing function locals-->global-->builtin

![1559182801747](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1559182801747.png)

例如：

```
>>> x = 1
>>> def foo():
...     x =2
...     def innerfoo():
...             x = 3  # 如注释后，结果会怎样呢？
...             print('locals', x)
...     innerfoo()
...     print('enclosing function locals', x)
...
>>> foo()
locals 3
enclosing function locals 2
>>> print('global', x)
global 1
```

从上例子，解释下，从内到外，依次形成四个命名空间：

* def innerfoo(): Local,即函数外部命名空间；
* def foo():Enclosing funciton locals;外部嵌套函数的名字空间;
* module: Global(module)；函数定义所在模块(文件)的名字空间;
* Python内置模块的名字空间: Builtin

如将x =3这句注释掉呢？x =3本属函数内部命名空间，注释掉后，函数innerfoo内部使用变量名x时，就会触发名称查找动作。在Local命名空间找不到，就会去Enclosing funciton locals命名查找，如找到就调用，如找不到就会继续向上查找。但总的都要遵循LEGB规则。

##### 内存结构

函数每次调用，都会新建栈帧，用于局部变量和执行过程存储。等执行结束，栈帧内存被回收，同时释放相关对象。

```
>>> def test():
...     print(id(locals()))
...
>>> test()
2082488697984
>>> test()
2082488697984
```

以字典实现的名字空间，虽灵活，但访问效率不高。为此，解释器有专门的内存空间，用效率最快的数组来替代字典。在函数指令执行前，先将包含参数在内的所有局部变量，以及要使用的外部变量复制（指针）到该数组。基于作用域不同，此内存可简单分为2部分：FAST和DEREF。这样，操作指令用索引就可立即读取或存储目标对象，这比哈希查找过程要高效。从反编译来看，有大量类似LOAD_FAST指令，其参数就是索引号。

```
>>> def enclosing():
...     E = "E"
...     def test(a, b):
...             c= a+b
...             print(E, c)
...     return test
...
>>> t = enclosing()   # 返回test函数
>>> t.__code__.co_varnames  #局部变量列表(含参数)。与索引号对应
('a', 'b', 'c')
>>> t.__code__.co_freevars  #所引用的外部变量列表。与索引号对应
('E',)
>>> import dis
>>> dis.dis(t)
  4           0 LOAD_FAST                0 (a) #从FAST区域，以索引号访问并载入
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 STORE_FAST               2 (c) #结果存入FAST区域

  5           8 LOAD_GLOBAL              0 (print)
             10 LOAD_DEREF               0 (E) # 从DEREF区域，访问并载入外部变量
             12 LOAD_FAST                2 (c)
             14 CALL_FUNCTION            2
             16 POP_TOP
             18 LOAD_CONST               0 (None)
             20 RETURN_VALUE
```

> FAST和DEREF数组大小是统计参数和变量得来的，对应索引值在编译期确定。故，不能在运行期扩。global关键字可向全局名字空间新建名字，但nonlocal却不可以。原因就是nonlocal代表外层函数，无法动态向其FAST数组插入或追加新元素。另外LEGB中的E保存在DEREF数组，相应查找过程也被优化，无需去迭代调用堆栈。所以，LEGB是针对远吗，而不是内部实现。

##### 名字空间

除非调用函数，否则函数执行期间，不会创建名字空间字典。也就是说，函数返回字典是按需延迟创建的，并从FAST区域赋值相关信息得来的。

```
>>> def test():
...     locals()["x"] = 100  # 运行期通过名字空间新建变量
...     print(x)             # 编译此指定时，本地并没有x这个名字
...
>>> test()                   # 失败
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in test
NameError: name 'x' is not defined
>>> import dis
>>> dis.dis(test)
  2           0 LOAD_CONST               1 (100)
              2 LOAD_GLOBAL              0 (locals)
              4 CALL_FUNCTION            0
              6 LOAD_CONST               2 ('x')
              8 STORE_SUBSCR

  3          10 LOAD_GLOBAL              1 (print)
             12 LOAD_GLOBAL              2 (x) # 编译时确定，从全局而非FAST载入
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 LOAD_CONST               0 (None)
             20 RETURN_VALUE
```

显然，名字使用静态作用域。运行期行为，对此并无影响。而另一方面，locals名字空间不过是FAST复制品，对其变更不会同步到FAST区域。

```
>>> def test():
...     x = 100
...     locals()["x"] = 999   #新建字典，进行复制。对复制品修改不影响FAST
...     print("fast.x=" , x)
...     print("locals.x=", locals()["x"])
...
>>> test()
fast.x= 100
locals.x= 100
```

> globals能新建全局变量，并影响外部环境，是因为模块直接以字典实现名字空间，没有类似FAST机制

栈帧会缓存locals函数所返回的字典，以避免每次都新建。故，可用它存储额外的数据，比如向后续逻辑提供上下文状态等。注意：只有再次调用locals函数，才会刷新字典数据。

```
>>> def test():
...     x = 1
...     d = locals()    
...     print(d is locals())    # 每次返回同一字典对象
...     d["context"] = "hello"  # 存储额外数据
...     print(d)
...     x = 999                 # 修改FAST时，不会主动刷新locals字典
...     print(d)                # 依旧输出上次调用locals结果
...     print(locals())         # 刷新操作由locals调用触发
...     print(d)                # 返回刷新后结果 
...
>>> test()
True
{'x': 1, 'd': {...}, 'context': 'hello'}
{'x': 1, 'd': {...}, 'context': 'hello'}
{'x': 999, 'd': {...}, 'context': 'hello'}
{'x': 999, 'd': {...}, 'context': 'hello'}
```

##### 静态作用域

```
>>> def test():
...     if False: x = 100
...     print(x)
...
>>> test()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in test
UnboundLocalError: local variable 'x' referenced before assignment
>>> dis.dis(test)
  3           0 LOAD_GLOBAL              0 (print)
              2 LOAD_FAST                0 (x)
              4 CALL_FUNCTION            1
              6 POP_TOP
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

从上代码可看到编译器将死代码剔除了。但x作用域还是不变的，依然是本地变量。

```
>>> def test():
...     if False: global x   # 死代码
...     x = 100
...
>>> dis.dis(test)
  3           0 LOAD_CONST               1 (100)
              2 STORE_GLOBAL             0 (x) # x 作用域为GLOBAL
              4 LOAD_CONST               0 (None)
              6 RETURN_VALUE
```

##### 总结

函数最好设计为纯函数，仅依赖参数、内部变量和自身属性；如依赖外部状态，会给重构和测试带来麻烦。也可将外部依赖变成keyword-only参数，这样测试就可自动以依赖环境，以确保最终结果一致。如必须依赖外部变量，则尽量不修改，以返回值交给调用方决策。

> 纯函数输出与输入以外的状态无关，没有任何隐式依赖。相同输入总是输出相同结果，且不对外部环境产生影响。

### 闭包
闭包是指函数离开生成环境后，依然可记住，并持续引用语法作用域里的外部变量。

```
>>> def make():
...     x = [1, 2]
...     return lambda: print(x)
...
>>> a = make()
>>> a()
[1, 2]
```

上面代码x作用域是make栈帧，调用结束后就应该销毁。但所返回匿名函数还可访问x变量，这就是闭包效应。

闭包不等于函数，只是形式上返回函数而已。因引用外部状态，闭包函数也不是纯函数。应闭包会延长环境变量生命周期，应慎重使用。

##### 创建

闭包由2部分组成，其创建过程为：

* 打包环境变量；

* 将环境变量作为参数，新建要返回的函数对象。

> 因生命周期改变，环境变量存储区从FAST转移到DEREF。闭包函数可以是匿名函数，也可是普通函数。

```
>>> def make():
...     x = 100
...     def test(): print(x)
...     return test
...
>>> dis.dis(make)
  2           0 LOAD_CONST               1 (100)
              2 STORE_DEREF              0 (x)    # 保存到DEREF

  3           4 LOAD_CLOSURE             0 (x)    # 闭包环境变量
              6 BUILD_TUPLE              1
              8 LOAD_CONST               2 (<code object test at 0x000001FF1D9358A0, file "<stdin>", line 3>)
             10 LOAD_CONST               3 ('make.<locals>.test')
             12 MAKE_FUNCTION            8        # 创建函数时包含闭包参数
             14 STORE_FAST               0 (test)

  4          16 LOAD_FAST                0 (test)
             18 RETURN_VALUE

Disassembly of <code object test at 0x000001FF1D9358A0, file "<stdin>", line 3>:
  3           0 LOAD_GLOBAL              0 (print)
              2 LOAD_DEREF               0 (x)
              4 CALL_FUNCTION            1
              6 POP_TOP
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
>>> f = make()     # 闭包函数得从DEREF读取环境变量
>>> dis.dis(f)
  3           0 LOAD_GLOBAL              0 (print)
              2 LOAD_DEREF               0 (x)     # 从DEREF载入闭包环境变量         
              4 CALL_FUNCTION            1
              6 POP_TOP
              8 LOAD_CONST               0 (None)
             10 RETURN_VALUE
```

##### 自由变量

闭包所引用的环境变量也被称为自由变量，它保存在函数对象的\_\_closure\_\_属性中。

```
>>> def make():
...     x = [1, 2]
...     print(hex(id(x)))
...     return lambda: print(x)
...
>>> f = make()
0x1ff1db77ac8
>>> f.__closure__
(<cell at 0x000001FF1DA2CFD8: list object at 0x000001FF1DB77AC8>,)
>>> f()
[1, 2]
>>> f.__code__.co_freevars   #当前函数引用外部自由变量列表
('x',)
>>> make.__code__.co_cellvars # 被内部闭包函数引用的变量列表
('x',)
```

> 自由变量保存在函数对象里，那多次调用是否被覆盖？不会，因为所返回的函数对象也是每次新建的。创建闭包等于“新建函数对象，附加自由变量”

```
>>> def make(x):
...     return lambda: print(x)
...
>>> a = make([1, 2])
>>> b = make(100)
>>> a is b    # 每次返回新的函数对象实例
False
>>> a.__closure__  # 自由变量保存在各自独立的函数实例中
(<cell at 0x000001FF1DA2CFA8: list object at 0x000001FF1DB77B88>,)
>>> b.__closure__
(<cell at 0x000001FF1DA2CE88: int object at 0x00007FFC35A96EF0>,)
```

多个闭包函数可共享同一自由变量。

```
>>> def queue():
...     data = []
...     push = lambda x: data.append(x)
...     pop = lambda: data.pop(0) if data else None
...     return push ,pop
...
>>> push, pop = queue()
>>> push.__closure__   #共享自由变量
(<cell at 0x000001FF1DA2C738: list object at 0x000001FF1DB77A88>,)
>>> pop.__closure__
(<cell at 0x000001FF1DA2C738: list object at 0x000001FF1DB77A88>,)
>>> for i in range(10, 13):
...     push(i)
...
>>> while True:
...     x = pop()
...     if not x: break
...     print(x)
...
10
11
12
```

##### 自引用

在函数内引用函数自己，也可构成闭包。当def创建函数对象后，会在当前名字空间将其与函数名字关联。因此，该函数实例自然也可作为自由变量。

```
>>> def make(x):
...     def test():
...             test.x = x
...             print(test.x)
...     return test
...
>>> a, b = make(1234), make([1, 2])
>>> a.__closure__    # 自由变量列表中包含自己
(<cell at 0x000001FF1DA2C648: function object at 0x000001FF1DB7F598>, <cell at 0x000001FF1DB7E138: int object at 0x000001FF1D9D1F50>)
>>> b.__closure__
(<cell at 0x000001FF1DB7E0A8: function object at 0x000001FF1DB7F950>, <cell at 0x000001FF1DB7E228: list object at 0x000001FF1DB77C08>)
>>> a()
1234
>>> b()
[1, 2]
```

##### 延迟绑定

闭包只是绑定自由变量，并不会立即计算引用内容。只有当闭包函数执行时，才访问所引用的目标对象。这样就有所谓的延迟绑定现象。

```
>>> def make(n):
...     x = []
...     for i in range(n):
...             x.append(lambda: print(i))
...     return x
...
>>> a, b, c =make(3)
>>> a(), b(), c()
2
2
2
(None, None, None)
>>> a.__closure__
(<cell at 0x000001FF1DA2CE88: int object at 0x00007FFC35A962B0>,)
>>> b.__closure__
(<cell at 0x000001FF1DA2CE88: int object at 0x00007FFC35A962B0>,)
>>> c.__closure__
(<cell at 0x000001FF1DA2CE88: int object at 0x00007FFC35A962B0>,)
```

为啥输出是2,2,2，而不是0,1,2呢？分析下执行次序：

* make创建并返回3个闭包函数，引用同一个自由变量i；
* make执行结束，i等于2；
* 执行闭包函数，引用并输出i的值，自然都是2。

从\_\_closure\_\_来看，函数并不直接存储自由变量，而是cell包装对象，以此间接引用目标。

每个自由变量都被打包成一个cell。循环期间虽然cell也和i一样引用不同整数对象，但这对尚未执行的闭包函数没有影响。循环结束，cell引用目标确定下来，这才是闭包函数执行时的输出结果。

那将上面函数修改下，复制自由变量后输出呢？

```
>>> def make(n):
...     x = []
...     for i in range(n):
...             c = i             # 复制对象，然后输出，但这是一个坑。。
...             x.append(lambda: print(c))
...     return x
...
>>> a, b, c =make(3)
>>> a(), b(), c()
2
2
2
(None, None, None)    
>>> a.__closure__           # 自由变量列表还是同一个对象
(<cell at 0x000001FF1DA2CFA8: int object at 0x00007FFC35A962B0>,)
>>> b.__closure__
(<cell at 0x000001FF1DA2CFA8: int object at 0x00007FFC35A962B0>,)
>>> c.__closure__
(<cell at 0x000001FF1DA2CFA8: int object at 0x00007FFC35A962B0>,)

>>> def make(n):
...     x = []
...     for i in range(n):
...             x.append(lambda o=i: print(o))   # 函数创建就计算的参数默认值，这样可不共享变量
...     return x
...
>>> a, b, c =make(3)
>>> a(), b(), c()
0
1
2
(None, None, None)
>>> a.__closure__   # 不是闭包了。
>>> b.__closure__
>>> c.__closure__
>>> make.__code__.co_cellvars
()
```

> 为啥呢？匿名函数参数o是私有变量，设置默认值时，其复制当前i引用。虽然只有一个变量i，但在循环工程中，它指向不同的整数对象。所以，复制的引用也就不同。最重要就是没有闭包了。

闭包具备封账特征，可实现隐式上下文状态，并减少参数。在设计上，其可部分替代全局变量，或将执行环境与调用接口分离。但其对自由变量隐式依赖，会提升代码复杂度，这直接影响测试和维护，自由变量生命周期的提升，也会提高内存占用。

### 调用

函数调用时，会专门为其分配用户栈内存。用户栈内存除用来存储变量外，还包括字节码参数和返回值所需空间。对系统指令来说，只是存放用户指令数据。

```
>>> def add(a, b):
...     c = a+b
...     return c
...
>>> dis.dis(add)
  2           0 LOAD_FAST                0 (a)  # 从FAST读取参数a,压入用户栈
              2 LOAD_FAST                1 (b)  # 从FAST读取参数b,压入用户栈
              4 BINARY_ADD                      # 系统指令从用户栈读取操作数，执行加法操作
              6 STORE_FAST               2 (c)  # 将结果返回FAST

  3           8 LOAD_FAST                2 (c)
             10 RETURN_VALUE
```

进程内存分为堆(heap)和栈(stack)两类：堆可自由申请，通过指针存储自由数据；而栈则用于指令执行与线程绑定。函数调用和执行都依赖线程栈存储上下文和执行状态。

比如，在函数A内调用函数B，需确保B结束后能回到A，并继续执行后续指令。这要求将A的后续指令地址预先存储起来。调用堆栈(call stack)的基本用途便是如此。

除返回地址外，还需为函数提供参数、局部变量存储空间。按不同调用约定，甚至为被调用函数提供参数和返回值内存。在线程栈内存里，每个被调用函数都有一块栈帧(stack frame)。

在栈帧内，变量内存固定不变，执行内存视具体指令重复使用。

```
>>> def add(x, y):
...     return x+y
...
>>> def test():
...     x =10
...     y = 20
...     z = add(x,y)
...     print(z)
...
>>> dis.dis(test)
  2           0 LOAD_CONST               1 (10)
              2 STORE_FAST               0 (x)

  3           4 LOAD_CONST               2 (20)
              6 STORE_FAST               1 (y)

  4           8 LOAD_GLOBAL              0 (add)   # 将待调用函数add入栈
             10 LOAD_FAST                0 (x)	   # 将变量x入栈	
             12 LOAD_FAST                1 (y)     # 将变量y入栈
             14 CALL_FUNCTION            2         # 调用函数(这里2位参数数量)   
             16 STORE_FAST               2 (z)	   # 将返回值从栈保存到变量区	

  5          18 LOAD_GLOBAL              1 (print)
             20 LOAD_FAST                2 (z)
             22 CALL_FUNCTION            1
             24 POP_TOP                             # 清除print返回值，确保栈平衡
             26 LOAD_CONST               0 (None)
             28 RETURN_VALUE
```

调用堆栈一般用在调试工具中，用于检视调用过程，以及各级环境变量取值。

```
>>> def A():
...     x = "func A"
...     B()
...
>>> def B():
...     C()
...
>>> def C():
...     f = sys._getframe(2)# 访问堆栈内不同层级栈帧对象，0位当前函数，1位上级，2为上2级，获取A栈帧
...     print(f.f_code)     # A代码对象
...     print(f.f_locals)   # A名字空间(此时运行期)
...     print(f.f_lasti)    # A最后执行指令偏移量(以确定继续执行位置)
...
>>> import sys
>>> A()
<code object A at 0x000001FF1D9480C0, file "<stdin>", line 1>
{'x': 'func A'}
6
>>> dis.dis(A)
  2           0 LOAD_CONST               1 ('func A')
              2 STORE_FAST               0 (x)

  3           4 LOAD_GLOBAL              0 (B)
              6 CALL_FUNCTION            0
              8 POP_TOP
             10 LOAD_CONST               0 (None)
             12 RETURN_VALUE
             
>>> import inspect   #标准库inspect，提供很多函数可查看线程的栈帧等
>>> def A(): B()
...
>>> def B(): C()
...
>>> def C():
...     for f in inspect.stack():
...             print(f.function, f.lineno)
...
>>> A()
C 2
B 1
A 1
<module> 1    

>>> import sys
>>> sys.setrecursionlimit(50) # 设置递归次数。可通过getrecursionlimit查看
>>> def test():
...     test()    #递归调用
...
>>> test()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in test
  File "<stdin>", line 2, in test
  File "<stdin>", line 2, in test
  [Previous line repeated 46 more times]
RecursionError: maximum recursion depth exceeded


```

对已有函数，可通过包装形式改变其参数列表。使其符合特定调用接口。

```
>>> def test(a, b, c):
...     print(locals())
>>> import functools
>>> t = functools.partial(test, b = 2, c = 3)
>>> t(1)
{'a': 1, 'b': 2, 'c': 3}
```

包装函数会将相关参数合并后，再调用原目标。

基本合并规则：

1、包装位置参数优先；

2、调用键值参数覆盖包装键值参数；

3、合并后不能对单个目标参数多次赋值。

```
>>> functools.partial(test, 1, 2)(3)   # 包装位置优先
{'a': 1, 'b': 2, 'c': 3}
>>> functools.partial(test, 1, c=3)(2, c=100) #调用键值参数覆盖包装参数
{'a': 1, 'b': 2, 'c': 100}
>>> t = functools.partial(test, 1, c = 3)
>>> t.func      # 原目标函数
<function test at 0x00000265BEC77EA0>
>>> t.args      # 包装位置参数
(1,)
>>> t.keywords  # 包装键值参数
{'c': 3}
```

