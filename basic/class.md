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



#### 统一类型

Python3不再支持Classic Class，所有类型都是New-Style Class。也就是默认继承自object。

```
>>> class A: pass
...
>>> class B(A): pass
...
>>> class C(B): pass
...
>>> issubclass(A, object)
True
>>> type(A) is A.__class__
True
>>> B.__base__   # 子类以__base__引为基类
<class '__main__.A'>
>>> A.__subclasses__()   # 基类通过__subclasses__获知所有直接子类。仅返回直接子类
[<class '__main__.B'>]
```

> \_\_init_subclass\_\_是一个隐式类型方法，在所有层次的子类型创建时被调用，其甚至可接收键值参数。

```
>>> class A:
...     def __init_subclass__(cls, **kwargs):
...             print("init_subclass:", cls, kwargs)
...
>>> class B(A): pass
...
init_subclass: <class '__main__.B'> {}
>>> class C(B, data = "hello"): pass   # 向基类传递参数
...
init_subclass: <class '__main__.C'> {'data': 'hello'}
```

#### 初始化

初始化方法\_\_init\_\_是可选的。

如果子类没有新的构造参数或新的初始化逻辑，则没必要创建该方法。按搜索顺序，解释器会找到基类初始化方法并执行。同样的缘故，可在子类的初始化方法中显式调用基类方法。

```
>>> class A:
...     def __init__(self):
...             print("A.init")
...
>>> class B(A): pass
...
>>> B()
A.init
<__main__.B object at 0x0000016F308C6A90>
```

可使用名字引用基类方法或调用super返回基类代理，以解除对类型名字的直接依赖。

```
>>> class A:
...     def __init__(self, x):
...             self.x = x
...
>>> class B(A):
...     def __init__(self, x, y):
...             super().__init__(x)
...             self.y = y
...
>>> o = B(1,2)
>>> vars(o)
{'x': 1, 'y': 2}
```

> 无论是A.\_\_init\_\_，还是B.\_\_init\_\_，其self参数都引用同一实例对象，所以在不同继承层次里创建的实例字段都存储在同一名字空间中，而不会出现多个同名成员。

#### 覆盖

覆盖(override)是指子类重新定义基类方法，从而实现功能变更。Python只需在搜索优先级更高的名字空间中定义同名方法即可实现覆盖。

覆盖不应改变方法参数列表和返回值类型，须确保不影响原有调用代码和相关文档，这与名字遮蔽不同。无论是多态机制，还是名字搜索顺序，都能确保新定义方法被执行。

```
>>> class A:
...     def m(self): print("A.m")
...     def do(self): self.m()     # 如果self是B类型，那么搜索首先找到自然是B.m
...
>>> class B(A):
...     def m(self): print("B.m")  # override
...
>>> A().do()
A.m
>>> B().do()
B.m
```

方法搜索发生在运行期，其不会静态绑定具体类型方法。注意的是方法没有属性(property)那样的优先级策略，应避免被实例同名成员遮蔽。

#### 多继承

多重继承允许类型有多个继承体系。从优点来说，提供了一种混入机制，让既有体系的类型可扩展出其他体系功能，但却会导致严重混乱，且让设计和代码的复杂度增大。故须慎重，尽量少用。

```
>>> class A:
...     def a(self): pass
...
>>> class B:
...     def b(self): pass
...
>>> class X(A, B): pass
...
>>> dir(X)   # 可访问所有基类成员
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'a', 'b']
>>> X.__base__     # 仅返回单个基类的只读属性  
<class '__main__.A'>
>>> X.__bases__    # 获取全部基类，按继承顺序返回所有全部基类
(<class '__main__.A'>, <class '__main__.B'>)
>>> class C:
...     def c(self): pass
...
>>> X.__bases__= (B, A, C)  # 调整继承顺序，并添加新基类，实现功能混入
>>> dir(X)                  # 可访问新基类成员
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'a', 'b', 'c']
```

除横向动态扩展外，还可纵向注入继承体系中。

```
>>> class A:
...     def test(self): print("A.test")
...
>>> class B(A):
...     pass
...
>>> class Proxy(A):
...     def test(self):              # 拦截A.test调用
...             print("proxy")
...             return super().test()
...
>>> B.__bases__ = (Proxy,)           # 修改B基类，形成B => Proxy => A方式
>>> B().test()
proxy
A.test
```

##### \_\_mro\_\_

任何类型的最终基类都是object，所以多重继承会出现菱形布局。最大问题是多重继承线上出现相同名字的成员时，如何选择？

这设计成员搜索基本规则：MRO(method resolution order)。对单继承，有近到远向祖先类依次查找。而多继承的复杂性在于深度和广度间的排列。

MRO步骤如下：

1. 按"深度优先，从左到右"顺序获取类型列表；

2. 移除列表中重复类型，仅保留最后一个；

3. 确保子类从在基类前，并保留多继承定义顺序。

![](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1560218865799.png)

```
>>> class Y:
...     n = "Y.n"
...     def test(self): print("Y.test")
...
>>> class X(Y): pass
...
>>> class N: pass
...
>>> class M(N):
...     n = "M.n"
...     def test(self): print("M.test")
...
>>> class A(X, M): pass
...              # “左侧优先，子类优先”
>>> A.__mro__    # __mro__是只读属性，且不能通过实例访问。
(<class '__main__.A'>, <class '__main__.X'>, <class '__main__.Y'>, <class '__main__.M'>, <class '__main__.N'>, <class 'object'>)
>>> A().n
'Y.n'
>>> A().test()
Y.test
>>> A.__bases__ = (M, X)  # 调整继承顺序，M变为左侧了。
>>> A.__mro__
(<class '__main__.A'>, <class '__main__.M'>, <class '__main__.N'>, <class '__main__.X'>, <class '__main__.Y'>, <class 'object'>)
>>> A().test()
M.test
```

应注意避免多条继承线存在交叉的现象，并竭力控制多继承和继承深度。

##### super

该函数返回基类型代理，完成对基类成员的委托访问。

有2个参数，第二参数对象提供\_\_mro\_\_列表，第一参数则指定起点。函数总是返回起点位置后一类型。这要求第二参数必须是第一参数的实例或子类型，否则第一参数不会出现在列表中。同时，第二参数还是实例和类型方法所需的绑定对象。

看看super算法伪码：

```
def super(t, o):
	mro = getattr(o, "__mro__", type(o).__mro__) # 返回实例或类型mro列表  
	index = mro.index(t)+1                       # 返回列表中的下一类型 
	return mro[index]
```

对单继承，通常省略参数，默认使用当前类型和实例，返回直接基类。但对多继承，则须明确指定起点和列表提供者。

```
>>> class A:
...     @classmethod
...     def demo(cls): pass
...     def test(self): print("A")
...
>>> class B(A):
...     def test(self): print("B")
...
>>> class C(B):
...     def test(self): print("C")
...
>>> C.__mro__
(<class '__main__.C'>, <class '__main__.B'>, <class '__main__.A'>, <class 'object'>)
>>> super(C, C).test
<function B.test at 0x0000024CFC99A0D0>
>>> super(B, C).test
<function A.test at 0x0000024CFC99A048>
>>> o = C()
>>> super(C, o).__self__ is o  # 使用__self__作为绑定属性，存储第二参数引用
True
>>> super(C, o).test
<bound method B.test of <__main__.C object at 0x0000024CFC996470>>
>>> super(C, C).demo
<bound method A.demo of <class '__main__.C'>>
# 不建议直接使用__class__.__base__访问基类，可能会引发意外错误。如：
>>> class A:
...     def test(self):
...             print("A.test")
...
>>> class B(A):
...     def test(self):
...             print("B.test")                    # 在B的角度，self.__class__.__base__指向A
...             self.__class__.__base__.test(self) # 但实际，self可能是B子类实例，如C
...                                                # 这样self.__class__.__base__实际指向B   
>>> class C(B): pass                               # 结果导致B.test被递归               
...
>>> C().test()
RecursionError: maximum recursion depth exceeded while calling a Python object
```

> 正确方式采用显式类型名字访问，或用super函数。即使省略super参数，默认使用当前类型B为搜索起点。这样就可确保访问A.test方法，避免递归错误。即修改为super().test()即可

#### 抽象类

抽象类表示部分完成，且不能被实例化的类型。

作为设计方式，抽象类可分离主体框架和局部实现，或将共用和定制解耦。不同接口纯粹调用声明，抽象类属于继承树的组成部分，可有实现代码。从抽象类继承，必须实现所有层级未被实现的抽象方法，否则无法创建实例。

实现抽象类，须继承abc，或使用ABCMeta元类。

```
>>> from abc import ABCMeta, abstractmethod
>>> class Store(metaclass = ABCMeta):
...     def __init__(self, name):
...             self.name = name
...     def save(self, data):
...             with open(self.name, "wb") as f:
...                     d = self.dump(data)
...                     f.write(d)
...     def read(self):
...             with open(self.name, "rb") as f:
...                     return self.load(f.read())
...     @abstractmethod
...     def dump(self, data): ...
...     @abstractmethod
...     def load(self, data): ...
...
>>> Store("test.dat")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class Store with abstract methods dump, load
```

该示例定义存储框架，并实现了通用部分的接口方法。至于具体的序列化过程，则交由子类完成。如此，可按需创建不同版本。

```
>>> import pickle
>>> class PickleStore(Store):
...     def dump(self, data):
...             return pickle.dumps(data)
...     def load(self, data):
...             return pickle.loads(data)
...
>>> s = PickleStore("test.dat")
>>> s.save({"x":1, "y":2})
>>> s.read()
{'x': 1, 'y': 2}
```

如从抽象类继承，未实现全部方法，或添加新的抽象定义，那么该类型依然是抽象类。另外，抽象装饰器还可用于属性、类型方法和静态方法。

```
>>> class A(metaclass = ABCMeta):
...     @classmethod                    # 注意两个装饰器顺序
...     @abstractmethod
...     def hello(cls): ...
...
>>> class B(A): pass
...
>>> B()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class B with abstract methods hello
>>> class B(A):    # 即便抽象类型方法，依然需要实现，否则无法创建实例
...     @classmethod
...     def hello(cls): pass
...
>>> B()
<__main__.B object at 0x000001D392D46780>
```

### 开放类

在运行期，可动态向实例或类型添加新成员，其中也包括方法。

有别于实例字段，方法需要添加到类型名字空间，因为要作用于所有实例和后续类型。另外，只有添加到类型名字空间才能自动构成绑定，这是有解释器的运行规则所决定的。

> 将已绑定的方法添加到实例名字空间，只能算是字段赋值，算不上添加方法。而将一个函数添加到实例，即便手动设定\_\_self\_\_也无法构成绑定

```
>>> class X: pass
...
>>> o = X()
>>> o.test = lambda self:None   # 向实例添加函数字段
>>> o.test.__self__ = o         # 尝试手工设定绑定 
>>> o.test                      # 依然无法构成绑定，除非重写描述符的相关方法 
<function <lambda> at 0x000001D3929FC1E0>
>>> o.test()                    
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: <lambda>() missing 1 required positional argument: 'self'
```

考虑类型\_\_dict\_\_是一个只读代理，其只能以字段赋值方式添加新方法。如是类型和静态方法，还须调用装饰器函数。

```
>>> class X:pass
...
>>> o = X()
>>> X.a = lambda self: print(f"instance method: {self}")
>>> X.b = classmethod(lambda cls: print(f"class method: {cls}"))
>>> X.c = staticmethod(lambda: print("static method"))
>>> o.a()
instance method: <__main__.X object at 0x000001D392D59C88>
>>> o.b()
class method: <class '__main__.X'>
>>> o.c()
static method
```

##### object

作为祖先根类object，有点特殊，无法向其类型和实例添加任何成员

```
>>> object.x = 1
TypeError: can't set attributes of built-in/extension type 'object'
>>> o = object()
>>> o.x = 1
AttributeError: 'object' object has no attribute 'x'
>>> object().__dict__  # 实例没有__dict__属性
AttributeError: 'object' object has no attribute '__dict__'
```

> SimpleNamespace简单继承自object，用于替代那些"class X: pass"语句。不同于namedtuple创建结构化类型，SimpleNamespace直接创建实例。

```
>>> import types
>>> o = types.SimpleNamespace(a = 1, b="abc")
>>> o.c = [1,2]
>>> o.__dict__
{'a': 1, 'b': 'abc', 'c': [1, 2]}
```

##### \_\_slots\_\_

名字空间字段带来便利同时，也造成内存和性能问题。

对于需要创建海量实例的类型，须做“固化”处理。通过设置\_\_slots\_\_，阻止实例创建\_\_dict\_\_等成员。解释器仅为指定成员分配空间，添加任何非预置属性都会引发异常。

> 对于有\_\_slots\_\_设置的类型，解释器在创建类型对象时，直接将指定成员包装成描述符后静态分配到类型对象尾部。实例字段也不再通过\_\_dict\_\_存储，而是改为直接分配引用内存，这样就只能替换引用内容，无法改变成员名字，更无法分配新的成员空间。
>
> 成员后续操作均由描述符完成，可间接修改或删除字段内容，但却无法改变描述符本身。

```
>>> class A:
...     __slots__ = ("x","y")
...
>>> o = A()
>>> o.z = 100   # 不能新增属性，因为无法为其分配引用内存
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'A' object has no attribute 'z'
>>> o.x = 1    # 只能为预定字段赋值 
>>> o.y = 2
>>> o.x = "abc"  # 通过描述符修改字段内容
>>> o.x
'abc'
>>> del o.x      # 删除字段内容，但描述符依然存在
>>> o.x = 100    # 引用内存也在，可重新赋值
```

对\_\_slots\_\_的修改并不会影响类型创建时设定的内存分配策略，加上预定成员以描述符实现，所以也无法更换其名称。

```
>>> A.__slots__ = ("x","y","z")
>>> o = A(1,2)
TypeError: A() takes no arguments
>>> o.z = 3
AttributeError: 'A' object has no attribute 'z'
>>> A.__slots__ = ("a", "b")
>>> dir(A)
[..., 'x', 'y']
```

> 注意，\_\_slots\_\_限制的是实例，而非类型。因此，还是可为类型添加新成员。如果在\_\_slots\_\_中添加\_\_dict\_\_，可回归其原来样子，不过没啥意义了。

继承有\_\_slots\_\_设置的类型，同样需要添加该设置。其可为空，或是新增字段，以指示解释器继续使用特定分配策略。

```
>>> class A:
...     __slots__ = ("x", "y")
...     def __init__(self, x, y):
...             self.x =x
...             self.y =y
...
>>> class X(A):                            # 没有__slots__,会创建__dict__
...     def __init__(self,x, y, z):
...             super().__init__(x,y)
...             self.z = z
...
>>> class Y(A):
...     __slots__ = ("z",)                # 新增字段列表，或为空
...     def __init__(self, x, y, z):
...             super().__init__(x, y)
...             self.z = z
...
>>> x = X(1,2,3)
>>> x.__dict__          # 包含__dict__，用于存储非__slots__新增成员
{'z': 3}
>>> y = Y(1,2,3)
>>> y.__dict__
AttributeError: 'Y' object has no attribute '__dict__'
>>> dir(y)
[..., 'x', 'y', 'z']
```

### 运算符重载

每种运算符都有对应一个特殊名称的方法。解释器会将运算符指令转换成方法调用，方法名可参考标准库的operator文档。运算符重载就是定义目标方法。但不限于运算符，还可包括内置函数和某些语句。

```
>>> class X:
...     def __init__(self, data):
...             self.data = data
...     def __repr__(self):
...             return f"{self.__class__} {self.data!r}"
...     def __add__(self, other):     # 加法 +
...             return X(self.data + other.data)
...     def __iadd__(self, other):    # 增量赋值 +=
...             self.data.extend(other.data)
...             return self
...
>>> a, b = X([1, 2]), X([3, 4])
>>> a+b
<class '__main__.X'> [1, 2, 3, 4]
>>> a +=b
>>> a
<class '__main__.X'> [1, 2, 3, 4]
```

##### \_\_repr\_\_

函数repr、str都输出对象字符创格式信息。

方法\_\_repr\_\_倾向输出运行期状态，比如类型、id，以及关键性内容，其适应调试和观察。而\_\_str\_\_通常返回内容数据，其面向用户，易阅读，可输出到终端或日志。

```
>>> class X:
...     def __init__(self, x):
...             self._x = x
...     def __repr__(self):
...             return f"<X:{hex(id(self))}>"
...     def __str__(self):
...             return str(self._x)
...
>>> repr(X(1))
'<X:0x1d392dbb160>'
>>> str(X(1))
'1'
```

##### \_\_item\_\_

为对象添加索引(index)或主键（key）访问。

```
>>> class X:
...     def __init__(self, data = None):
...             self.data = data or []
...     def __getitem__(self, index):
...             return self.data[index]
...     def __setitem__(self, index, value):
...             if index >= len(self.data):
...                     self.data.append(value)
...                     return
...             self.data[index] = value
...     def __delitem__(self, index):
...             return self.data.pop(index)
...
>>> o = X([1, 2, 3, 4])
>>> o[2] = 300
>>> del o[2]
>>> o.data
[1, 2, 4]
```

##### \_\_call\_\_

让对象可像函数一样调用。实现该方法的对象被称为callable，函数也是其中一种。用实例对象替代函数，让逻辑自带内部状态(其封装性比闭包更好)。

```
>>> class X:
...     def __init__(self):
...             self.count = 0
...     def __call__(self, s):
...             self.count += 1
...             print(s)
...
>>> def test(f):
...     f("abc")
...
>>> test(lambda s:print(s))
abc
>>> test(X())
abc
>>> o= X()
>>> o("abc")
abc
```

##### \_\_dir\_\_

定制dir函数返回值，隐藏部分成员。

```
>>> class X:
...     def __dir__(self):
...             return (m for m in vars(self).keys() if not m.startswith("__"))
...
>>> o = X()
>>> o.a =1
>>> o.b = 2
>>> dir(o)
['a', 'b']
>>> o.__c = "abc"
>>> dir(o)
['a', 'b']
```

##### \_\_getattr\_\_

几个与实例属性访问相关方法：

* \_\_getattr\_\_： 当整个搜索路径都找不到目标属性时触发。
* \_\_setattr\_\_： 拦截对任何属性的赋值操作。
* \_\_delattr\_\_： 拦截对任何属性的删除操作。

> 拦截到赋值和删除操作后，由拦截方法负责处理赋值和删除行为，忽略则被视为放弃该操作。
>
> 在拦截方法内部，通过属性或setattr等函数调用都可能再次被拦截，甚至引发递归调用错误。应直接操作\_\_dict\_\_，或使用基类object.\_\_setattr\_\_方法。

```
>>> class A:
...     a = 1234
...
>>> class B(A):
...     def __init__(self, x):
...             self.x = x
...     def __getattr__(self, name):
...             print(f"get: {name}")
...             return self.__dict__.get(name)
...     def __setattr__(self, name, value):
...             print(f"set: {name} = {value}")
...             self.__dict__[name] = value
...     def __delattr__(self, name):
...             print(f"del: {name}")
...             self.__dict__.pop(name, None)
...
>>> o = B(1)
set: x = 1    # 拦截B.__init__里对self.x的赋值操作
>>> o.a		  # 搜索路径里能找到的成员，不会触发__getattr__	
1234
>>> o.xxx     # 找不到，才会触发
get: xxx
>>> o.x = 100 # 任何赋值操作都会被__setattr__拦截(无论其是否存在)
set: x = 100
>>> del o.x   # 拦截任何删除操作，(无论其是否存在)
del: x
```

方法\_\_getattribute\_\_拦截任何实例属性的访问(无论其是否存在)。

> 拦截目标包括\_\_dict\_\_，这意味着只能通过基类方法进行操作。
>
> 访问不存在的成员时，\_\_getattribute\_\_拦截后不再触发\_\_getattr\_\_，除非显式调用或触发异常。
>
> 在下面实例中，因object.\_\_getattribute\_\_找不到目标属性，所以会触发A.\_\_getattr\_\_调用，继而还会拦截到其内部对\_\_dict\_\_的访问。

```
>>> class A:
...     def __init__(self, x):
...             self.x = x
...     def __getattribute__(self, name):   # 返回结果，或引发AttributeError
...             print(f"getattribute: {name}")
...             return object.__getattribute__(self, name)
...     def __getattr__(self, name):
...             print(f"getattr: {name}")
...             return self.__dict__.get(name)
...
>>> o = A(1)
>>> o.x    # 无论其是否存在，均被拦截
getattribute: x
1
>>> o.s
getattribute: s          # __getattribute__拦截
getattr: s               # object.__getattribute__找不到s,触发__getattr__
getattribute: __dict__	 # __getattr__访问__dict__被再次拦截
```

