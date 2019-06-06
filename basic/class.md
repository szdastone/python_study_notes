# 类

面向对象编程是最流行的编写软件方法之一。在面向对象编程中，基于类来创建对象就是很通用的方法。根据类来创建对象被称为实例化。

### 定义

类封装一组相关数据，使之成为一个整体，并使用方法持续展示和维护。与函数类似，类也是一种小粒度复用单位，其行为特征更复杂。函数具有单一入口和出口，可完成一次计算过程，而类从构造开始就面临多个方法。按不同次序调用会有不同结果。

类存在两种关系：继承 和 组合。

类与模块作为复合结构由相似之处，但不同之处在于：

* 类可生成多个实例

* 类可被继承和扩展

* 类实例的生命周期可控

* 类支持运算符，可按需重载

##### 创建和使用类

使用类可模拟任何东西。比如简单类Dog，如下：

```
class Dog():
	def __init__(self, name, age):
		self.name = name
		self.age = age
	
    def sit(self):
    	print(slef.name.title()+' is now sitting.')
  
  	def roll_over(self):
  		print(self.name.title()+"rolled over!")
```

在Python中，类一般定义时名称都是首字母大写的。类定义括号为空，表示从空白创建这个类。

方法\_\_init\_\_()时一个特殊方法，在类创建新实例时，会自动运行它，开头和末尾有两个下划线，这是一种约定，以便与自己定义的普通方法发生名称上的冲突。

在定义方法时，形参self必不可少，还必须在其他形参前面。这是因为Python在调用方法时会自动传入实参self。每个与类相关联的方法调用都自动传递实参self，它是一个指向实例本身的引用，让实例能访问类中的属性和方法。所以每个类所定义的方法第一个参数都是self。

以上是创建类，那么怎么调用呢？也就是需要根据类创建实例，再通过实例来调用相应的方法。比如:

```
my_dog = Dog('willie', 6)
print("My dog's name is "+my_dog.name.title()+"!")
print("My dog's is "+str(my_dog.age)+" years old.")
my_dog.sit()
my_dig.roll_over()
```

上面演示了怎么通过实例来调用对象的属性与方法。那怎么修改属性呢？第一是直接修改，第二是通过方法来修改。比如：

```
my_dog.age = 7
```

通过方法就需要在类中定义方法。接上面定义的类（部分代码省略）：

```
	def setAge(self, age):
		self.age = age

my_dog.setAge(7)
```

虽然可通过上面方法来控制类修改类中的属性，但为确保安全，还是要进行必要的基本检测，以保障类的安全。

关键字class同样是运行期指令，用以完成类型对象的创建。

```
>>> def test():
...     class X:
...             data = 100
...             def get(self): return self.data
...
>>> import dis
>>> dis.dis(test)
  2           0 LOAD_BUILD_CLASS
              2 LOAD_CONST               1 (<code object X ...>) # test.__code__.co_consts[1]
              4 LOAD_CONST               2 ('X') 
              6 MAKE_FUNCTION            0   # 创建X函数
              8 LOAD_CONST               2 ('X')
             10 CALL_FUNCTION            2  # 调用__build_class__哈数
             12 STORE_FAST               0 (X)
             14 LOAD_CONST               0 (None)
             16 RETURN_VALUE
             ...
```

> 从上反编译来看，先创建X函数，其内容是属性设置和方法创建。随后，该函数被当做参数传递给builtins.\_\_build_class\_\_调用，这里其实就是元类执行所在。
>
> 在_\_build_class\_\_调用X时，会提供一个字典作为其推栈帧的名字空间，用于保存类型成员。完成后，将该字典连同名字和基类一起传递给元类，最终生成目标类型对象。
##### 类型和实例

如果类型在模块中定义，那么其生命周期与模块等同。如果其放在函数内，那么每次都是新建，即使名字和内容形同，也属于不同类型。

```
>>> def test():
...     class X: pass
...     return X()
...
>>> a, b = test(), test()
>>> a.__class__ is b.__class__
False
```

> 函数内定义的类型对象，在所有实例死亡后，会被垃圾回收

类型对象除用来创建实例外，也为所有实例定义了基本操作接口，其负责管理整个家族的可共享收据和行为模板。而实例只保存私有特征，其在内部引用从所属类型或其他祖先类查找所需的方法，用来驱动和展现个体面貌。

类型所创建的多个实例之间，除家庭归属外，并无直接关系。无论从诞生到消亡，还是私有数据变更，都不影响类型或其他兄弟姐妹。

##### 名字空间

类型有自己的名字空间，存储当前类型定义的字段和方法。这其中并不包括所继承的祖先成员，其同样以引用关联祖先类型，无须复制到本地。

```
>>> class A:
...     a = 100                 # 类字段
...     def __init__(self, x):  # 实例初始化方法
...             self.x = x      # 实例字段 
...     def get_x(self):        # 实例方法
...             return self.x

>>> class B(A):                 # 继承自A
...     b = "hello"
...     def __init__(self, x, y):
...             super().__init__(x)  # 调用父类的初始化方法
...             self.y = y
...     def get_y(self):
...             return self.y

>>> o = B(1, 2)
```

实例会存储所有继承层次的实例字段，因为这些都属于其私有数据。方法算是函数辩题，其本身无状态，所以可共享。即便继承，也有其私有特征。也因为这些私有数据驱动，才能让方法展现不同结果来。

```
>>> A.__dict__  # 类型名字空间返回mappingproxy只读视图，不可直接修改。
mappingproxy({'__module__': '__main__', 'a': 100, '__init__': <function A.__init__ at 0x0000015740D507B8>, 'get_x': <function A.get_x at 0x0000015740D50F28>, '__dict__': <attribute '__dict__' of 'A' objects>, '__weakref__': <attribute '__weakref__' of 'A' objects>, '__doc__': None})
>>> B.__dict__
mappingproxy({'__module__': '__main__', 'b': 'hello', '__init__': <function B.__init__ at 0x0000015740E02840>, 'get_y': <function B.get_y at 0x0000015740E028C8>, '__doc__': None})
>>> o.__dict__   # 实例 名字空间是普通字段，可直接修改。
{'x': 1, 'y': 2}
>>> dir(o)       # 函数dir搜索所有可访问成员名字，vars直接返回__dict__属性
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'a', 'b', 'get_x', 'get_y', 'x', 'y']
>>> vars(o)
{'x': 1, 'y': 2}
```

当通过实例或类型访问某个成员时，会从当前对象开始，依次由近到远向祖先类查找。如此的好处就是祖先新增功能可直接遗传给所有后代。

> 在继承层次的不同名字空间中允许有同名成员，并按顺序优先命中。

成员查找规则和LEGB不同。前者基于继承体系，后者基于执行作用域。

```
>>> x = 100
>>> class A:
...     def __init__(self, x):
...             self.x =x
...     def test(self):
...             print(x)      # LEGB
...             print(self.x) # 明确以self指定搜索目标
...
>>> o = A("abc")
>>> o.test()
100
abc
```

> 类型创建时，class在内部以函数方式执行。接收类型名字空间字典作为堆栈帧执行的名字空间。这样，成员定义作用域内的locals实际指向了class.\_\_dict\_\_（可读写）。从语法上，class是类型定义，而非函数定义，这与内部执行方式无关，因此class不构成E/enclosing作用域

```
>>> def enclosing():
...     a = "enclosing.a"   
...     class A:
...             a = "A.a"                       # L ==> A.__dict__
...             def test(self):
...                     print("E.a=", a)        # E ==> enclosing.a
...             print("A.locals =", locals())   # L ==> A.__dict__
...             print("A.a =", a)               # L ==> A.__dict__
...     A().test()
...
>>> enclosing()
A.locals = {'__module__': '__main__', '__qualname__': 'enclosing.<locals>.A', 'a': 'A.a', 'test': <function enclosing.<locals>.A.test at 0x0000015740E02AE8>}
A.a = A.a
E.a= enclosing.a
```

### 字段

按所处名字空间不同，将字段分为类型字段和实例字段两类。官方文档将成员统称为Attribute。

类型字段在class语句块内直接定义，而实例字段必须通过实例引用(self)赋值定义。仅从执行方式来看，无论实例方法存在于哪级类型，其隐式参数self总指向当前调用实例。那么通过它创建的字段，也必然存在于该实例的名字空间内。

```
>>> class A: 
...     def test(self):       # self总是引用当前实例，这与继承层次无关
...             print(self)
...             self.x = 100  # 向当前实例字段赋值
...
>>> class B(A):
...     pass
...
>>> o = B()
>>> hex(id(o))
'0x15740dfccf8'
>>> o.test()                # 对比实例id
<__main__.B object at 0x0000015740DFCCF8>
```

站在语法和设计角度来看，实例字段是所有类型的组成部分，与该类型的相关方法关联。所以，依然称其为某某类型的实例字段。

##### 字段赋值

可使用赋值语句为类型或实例添加新的字段。

```
>>> class X: pass
...
>>> X.a = 100   # 新增类型字段
>>> vars(X)
mappingproxy({'__module__': '__main__', '__dict__': <attribute '__dict__' of 'X' objects>, '__weakref__': <attribute '__weakref__' of 'X' objects>, '__doc__': None, 'a': 100})
>>> o = X()
>>> o.b = 200  # 新增实例字段
>>> vars(o)
{'b': 200}
```

可一旦以子类或实例重新赋值，就将会在其名字空间建立同名字段，并会遮蔽原字段。这与“赋值总是在当前名字空间建立关联”规则一致。

```
>>> class X: pass
...
>>> o = X()
>>> X.data = 100
>>> o.data         # 按搜索规则，命中X.data
100
>>> o.data = 200   # 在o名字空间新建data实例字段
>>> o.data         # 按搜索顺序，命中o.data  
200
>>> vars(o)
{'data': 200}
>>> X.data        # 不影响原类型字段
100
>>> vars(X)
mappingproxy({'__module__': '__main__', '__dict__': <attribute '__dict__' of 'X' objects>, '__weakref__': <attribute '__weakref__' of 'X' objects>, '__doc__': None, 'data': 100})
```

删除操作仅针对当前名字空间，而不会按搜索顺序查找类型或基类。

```
>>> class X: pass
...
>>> X.data = 100
>>> o = X()
>>> del o.data    # 当前实例名字空间并没有data成员
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: data
```

成员访问总是按搜索规则进行，这与普通变量的静态作用域不同。

```
>>> class X:
...     data = "X.data"
...
>>> def test():
...     o = X()
...     print(o.data)      # 找到 X.data
...     o.data = "o.data"  # 新增o.data
...     print(o.data)      # 找到o.data 
...     del o.data         # 删除o.data   
...     print(o.data)      # 找到X.data
...
>>> test()
X.data
o.data
X.data
```

##### 私有字段

将私有字段暴露给用户是很危险的。因为无论是修改还是删除都无法被截获，因此可能引发意外错误。因为没有严格意义上访问权限设置，所以最好将他们隐藏起来。

如成员名字以上下划线开头，但没有以上下划线结尾，那么编译器会自动将其重命名。

```
>>> class X:
...     __table = "user"                 # 类型成员
...     def __init__(self, name):
...             self.__name = name       # 实例成员
...     def get_name(self):
...             return self.__name
...
>>> vars(X)
mappingproxy({'__module__': '__main__', '_X__table': 'user', '__init__': <function X.__init__ at 0x0000015740E02A60>, 'get_name': <function X.get_name at 0x0000015740E02BF8>, '__dict__': <attribute '__dict__' of 'X' objects>, '__weakref__': <attribute '__weakref__' of 'X' objects>, '__doc__': None})
>>> vars(X("user1"))
{'_X__name': 'user1'}
```

所谓重命名，就是被编译器附加了类型名称前缀。而且这种方式让继承类也无法访问。

重命名机制总是针对当前类型，继承类型无法访问重命名后的基类成员。可将双下划线前缀改为单下划线，虽不能自动重命名，但提示作用依旧。

```
>>> class A:
...     __name = "user"
...
>>> class B(A):
...     def test(self):
...             print(self.__name)   # 被当做当前类型的私有字段重命名：_B__name
...
>>> B().test()
AttributeError: 'B' object has no attribute '_B__name'
```

##### 相关函数

以下为几个与类成员相关的内置函数。

```
>>> class A:
...     data = "A.data"
...
>>> class B(A):
...     pass
...
>>> hasattr(B(), "data")   # 按搜索规则查找整个继承层次
True
>>> getattr(B(), "data")   # 相当于B().data,同样是按搜索规则进行
'A.data'
>>> getattr(B(), "name", "abc") # 返回默认值
'abc'
>>> o = B()
>>> o.data
'A.data'
>>> setattr(o, "data", "o.data")  # 等同于o.data ='o.data',针对当前名字空间
>>> o.data
'o.data'
>>> o = B()
>>> delattr(o, "data")    # 等同于del o.data,针对当前名字空间
AttributeError: data
```

它们是相关语句的函数版本，可接收动态成员名称，还能用于lambda函数。

```
>>> (lambda: getattr(B(), "data"))()
'A.data'
```

### 属性

对私有字段进行重命名保护，那公开字段该如何处理？

问题的核心是访问拦截，必须由内部逻辑决定如何返回结果。而属性(property)机制就是将读、写和删除操作映射到指定的方法调用上，从而实现操作控制。

```
>>> class X:
...     def __init__(self, name):
...             self.__name = name
...     @property                      # 读
...     def name(self):
...             return self.__name
...     @name.setter                   # 写(注意语法格式)
...     def name(self, value):
...             self.__name = value
...     @name.deleter                  # 删除
...     def name(self):
...             raise AttributeError("can't delete attribute")
...
>>> o = X("user1")
>>> o.name
'user1'
>>> o.name = "abc"
>>> o.name
'abc'
>>> del o.name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 12, in name
AttributeError: can't delete attribute
>>> vars(X)
mappingproxy({'__module__': '__main__', '__init__': <function X.__init__ at 0x0000015740E02D90>, 'name': <property object at 0x0000015740E004F8>, '__dict__': <attribute '__dict__' of 'X' objects>, '__weakref__': <attribute '__weakref__' of 'X' objects>, '__doc__': None})
```

> @语法被称为修饰器。多个方法名必须相同。默认从读方法开始定义属性，随后以属性名定义写和删除操作。如果实现只读，或禁止删除，则只需去掉对应方法即可。

可用lambda简化代码。

```
>>> class X:
...     name = property(
...             fget = lambda self:getattr(self,"_name", None),
...             fset = lambda self, value: setattr(self,"_name", value),
...     )
>>> o.name = "abc"
>>> o.name
'abc'
```

以方法应对操作，可主动判断是否继续执行。例如，判断赋值数据是否合法，或阻止创建同名实例字段。

```
>>> class X:
...     __data = "X.data"
...     @property
...     def data(self):
...             return self.__data
...     @data.setter
...     def data(self, value):
...             if not isinstance(value, str):   # 检查数据类型
...                     raise ValueError("value type must be str")
...             X.__data = value                 # 直接修改类型字段
...
>>> o = X()
>>> o.data = 1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 9, in data
ValueError: value type must be str
>>> o.data = "abc"
>>> vars(o)
{}
>>> o.data
'abc'
```

属性可实现延迟初始化，而无须提前准备数据。且属性方法调用也未必会绑定字段，可按需要从任意数据源获取。

##### 优先级

尽管属性保存于类型名字空间，其优先级高于同名实例字段。

```
>>> class X:
...     data = property(lambda self: "X.data")
...
>>> o = X()
>>> o.data = 123        # 属性优先，data没有指定赋值方法
AttributeError: can't set attribute
>>> o.__dict__["data"] = 123  # 绕开限制，直接对__dict__操作。
>>> vars(o)
{'data': 123}
>>> o.data
'X.data'
```

### 方法

方法是一种特殊函数，其与特定对象绑定，用来获取或修改对象状态。

实际上，无论是对象构造、初始化、析构，还是运算符，都以方法实现。根据绑定目标和调用方式不同，方法可分为实例方法、类型方法以及静态方法。

实例方法与实例对象绑定。其参数列表中，将绑定对象作为第一参数，以便在方法中读取或修改数据状态。在以实例引用调用方法时，无须显式传入第一实参，而由解释器自动完成。建议参数名为self，同样cls作为类型方法的第一参数名。

```
>>> class X:
...     def __init__(self, name):
...             self.__name = name
...     def get(self):            # 以实例引用调用，自动传入self参数
...             return self.__name
...     def set(self, value):
...             self.__name = value
...
>>> o = X("Q.yuhen")
>>> o.get()      # 忽略第一参数
'Q.yuhen'
>>> o.set("qyuhen")
```

类型方法用来维护类型状态，面向族群提供服务接口。除绑定的第一参数名称不同外，还需添加专门的装饰器，以便解释器将其与实例方法区分开来。

```
>>> class X:
...     __data = "X.data"
...     @classmethod   # 定义为类型方法
...     def get(cls):  # 解释器自动将类型对象X作为cls参数传入
...             return cls.__data
...     @classmethod
...     def set(cls, value):
...             cls.__data = value
...
>>> X.get()            # 同样忽略cls参数
'X.data'
>>> X.set("hello")
>>> o = X()
>>> o.get()
'hello'
>>> o.set("H")
>>> o.get()
'H'
```

静态方法则更像普通函数，即不接受实例引用，也不参与类型处理，所以也就没有自动传入的第一参数。使用静态方法，更多原因就是将类型作为一个作用域，或为当前类型添加便捷接口。

```
>>> class DES:
...     def __init__(self, key):
...             self.__key = key
...     def encrypt_bytes(self, value):  # 实例方法
...             pass
...     @staticmethod                    # 定义为静态方法    
...     def encrypt(key, s):             # 所有参数必须显式传入
...             des = DES(key)
...             return DES(key).encrypt_bytes(s.encode("utf-8"))
...
>>> DES.encrypt("key", "abc")
```

不论是哪种方法，都保存在类型名字空间中，故不能重名。装饰器和参数的差异，并不能改变在同一名字空间字典中对同一主键重复赋值会覆盖前一次赋值的现实。

```
>>> class X:
...     def test(self):
...             pass
...     @classmethod
...     def test(cls, a):
...             pass
...     @staticmethod
...     def test(x):
...             pass
...
>>> vars(X)
mappingproxy({'__module__': '__main__', 'test': <staticmethod object at 0x0000015740E0C048>, '__dict__': <attribute '__dict__' of 'X' objects>, '__weakref__': <attribute '__weakref__' of 'X' objects>, '__doc__': None})
```

##### 绑定

```
>>> class X:
...     def test(self): pass
...
>>> o = X()
>>> o.test
<bound method X.test of <__main__.X object at 0x0000015740E0AF98>>
>>> X.test
<function X.test at 0x0000015740E087B8>
```

同一方法，仅是引用方式不同，结果就有绑定方法和函数区别。这是为啥呢？这是描述符机制在起作用。它会将实例附加到方法的\_\_self\_\_属性，当解释器执行时就可隐式自动传入self参数。

```
>>> o.test.__self__ is o
True
>>> X.test.__self__
AttributeError: 'function' object has no attribute '__self__'
```

当然，就算不是绑定方法，依然可像函数那样调用，无非要显式传入self而已。

```
>>> X.test(o)  # 显式传入self参数
```

> 可用method.\_\_func\_\_获取\_\_code\_\_的相关信息。

自带\_\_self\_\_属性后，无论作为参数传递，还是赋值给变量，总能正确引用原实例。

```
>>> class X:
...     def __init__(self, name):
...             self.__name = name
...     def test(self):
...             return self.__name
...
>>> a, b = X("a").test, X("b").test  #将方法赋值给变量
>>> a()
'a'
>>> b()
'b'
```

与之类似的还有类型方法。其实现方式与实例方法完全相同，甚至连存储类型应用的字段名都还是\_\_self\_\_。至于剩下的静态方法，其不傍他人，总以函数的身份出现。

```
>>> class X:
...     @classmethod
...     def class_method(cls): pass
...     @staticmethod
...     def static_method(): pass
...
>>> X.class_method.__self__ is X
True
>>> X.static_method
<function X.static_method at 0x0000015740E089D8>
```

##### 特殊方法

下面是几个由解释器自动调用，与对象生命周期相关的方法。

* \_\_new\_\_：构造方法，创建对象实例。
* \_\_init\_\_：初始化方法，设置实例的相关属性。
* \_\_del\_\_：析构方法，实例被回收时调用。

创建实例时，会先后调用构造和初始化方法。

```
>>> class X:
...     def __new__(cls, *args):   # 与__init__接收相同的调用参数
...             print("__new__", args)
...             return super().__new__(cls)
...     def __init__(self, *args):  # self由__new__创建并返回
...             print("__init__", args)
...
>>> X(1,2)
__new__ (1, 2)
__init__ (1, 2)
```

如果\_\_new\_\_方法返回的实例与cls类型不符，将导致\_\_init\_\_无法执行。

```
>>> class X:
...     def __new__(cls):
...             print("__new__", cls)
...             return [1, 2]
...     def __init__(self):
...             print("__init__")
...
>>> X()
__new__ <class '__main__.X'>
[1, 2]
```



### 继承

类最大的好处就是可以继承。一个类继承另一个类时，它自动获得另一个类的所有属性和方法，原有类为父类，新类为子类。子类继承父类所有属性和方法，还可以定义自己属性和方法。

```
class Car():
	def __init__(self, make, model, year):
		self.make = make
		self.model = model
		self.year = year
		
class ElectricCar(Car):
	def __init__(self, make, model, year):
		super.__init__(make, model, year)
		self.battery_size = 70
```

对于父类方法，只要不符合子类模拟的行动，都可以重写。可在子类定义一个很父类同名的方法，就可重写父类方法。

使用继承，可让子类保留父类继承而来的精华，并剔除不需要的东西。

面向对象有3个基本特征：封装、继承和多态。



