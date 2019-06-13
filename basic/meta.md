# 元编程

元编程将程序当做数据，或在运行期完成编译器的工作。

### 1、装饰器

代码可利用装饰器，在不浸入内部实现，甚至在不知情的情况下，插入扩展逻辑。

```
>>> @log
... def add(x, y):
...     return x + y
```

编译器会将装饰器语法处理成如下模式。

```
>>> def add(x, y):
...     return x + y
...
>>> add = log(add)
```

对按名字搜索执行的调用函数，根本察觉不到。而被调用函数，只要能收到正确参数即可。这样，就可在任何时候添加额外的处理过程了。从装饰器语法和实现效果来看，类似AOP编程范式。

#### 1.1、实现

基于装饰器的实际执行方式，可直接以函数实现。

```
>>> def log(fn):
...     def wrap(*args, **kwargs):   # 通过包装函数间接调用原函数
...             print(f"log: {args}, {kwargs}")
...             return fn(*args, **kwargs)
...     return wrap                  # 返回包装函数替代原函数与名字关联
...
>>> @log
... def add(x, y):
...     return x +y
...
>>> add(1, 2)
log: (1, 2), {}
3
```

其返回包装代理，或仅添加一些额外属性后，原样返回。

```
>>> def log(fn):
...     fn.log_func = lambda *args, **kwargs: print(f"log: {args}, {kwargs}")
...     return fn
```

同样，任何可调用对象(callable)都可用来实现装饰器模式。

```
>>> class log:
...     def __init__(self, fn):
...             self.fn = fn
...     def __call__(self, *args, **kwargs):
...             print(f"log: {args}, {kwargs}")
...             return self.fn(*args, **kwargs)
...
>>> @log
... def add(x, y):
...     return x + y
...
>>> add(1, 4)
log: (1, 4), {}
5
```

> 因每条“@log”都会被处理成“log(add)”，也就是每次都会新建一个实例。所以，其应用于多个目标函数完全没问题。

相比于函数，使用类实现似乎可构建更复杂的装饰器，但这存在一些麻烦。类实现的装饰器应用于实例方法时，会导致方法绑定丢失。

```
>>> class log:
...     def __init__(self, fn):
...             self.fn = fn
...     def __call__(self, *args, **kwargs):
...             print(f"log: {args}, {kwargs}")
...             return self.fn(*args, **kwargs)
...
>>> class X:
...     @log
...     def test(self): pass
...
>>> x = X()
>>> x.test  # 方法被装饰器实例所替代
<__main__.log object at 0x0000018F23BA6128>
>>> x.test()   
log: (), {}
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in __call__
TypeError: test() missing 1 required positional argument: 'self'
```

因装饰器实例替代了方法，结果导致实现绑定的描述符方法被隐藏，无法自动调用。要么让装饰器也实现描述符协议，要么显式传递参数。这样反而不如用函数来的简单，因为函数对象默认实现了描述符协议和绑定规则。至于状态维持，函数一样可以添加属性。

```
>>> def log(fn):
...     def wrap(*args, **kwargs):
...             print(f"log: {args}, {kwargs}")
...             return fn(*args, **kwargs)
...     return wrap
...
>>> class X:
...     @log
...     def test(self): pass
...
>>> x = X()
>>> x.test
<bound method log.<locals>.wrap of <__main__.X object at 0x0000018F23BA6390>>
>>> log.__get__   # 函数默认实现了描述符协议
<method-wrapper '__get__' of function object at 0x0000018F23B90F28>
```

##### 嵌套

嵌套对一个目标使用多个装饰器。

```
@a
@b
def test(): pass
```

最终效果就是多次嵌套调用。

```
test = a(b(test))
```

装饰器接收前一装饰器的返回值，该返回值可能是包装对象或是原函数。如此，就需要注意排列顺序，因为每个装饰器的返回值并不相同。比如，须确保类型方法的装饰器是最外面一个，因为无法确定内层装饰器如何实现。这可能也会因描述符问题，导致方法绑定失效。

```
class X:
	@classmethod   # 注意放在最外层
	@log
	def test(cls): pass
```

##### 参数

除被装饰目标外，还可向装饰器传递其他参数。以实现更多定特性。

```
@log("demo")
def test(): pass
```

这样，就多了一个处理过程。

```
>>> def log(name = "default"):  # 外层函数接收参数
...     print(f"args: {name}")
...     def decorator(fn):      # 装饰器 
...             print(f"decorator: {fn}")
...             return fn       # 返回包装函数或原函数
...     return decorator
...
>>> @log("demo")
... def test(): pass
...
args: demo
decorator: <function test at 0x0000018F23BAA400>
>>> @log()                # 对于有参数的装饰器，如有默认值须括号。
... def test(): pass
...
args: default
decorator: <function test at 0x0000018F23BAA510>
```

##### 属性

可让包装函数更像原函数一些，比如拥有某些相同属性。

```
>>> import functools
>>> def log(fn):
...     @functools.wraps(fn)
...     def wrap(*args, **kwargs):
...             return fn(*args, **kwargs)
...     print(f"wrap: {id(wrap)}, func: {id(fn)}")
...     return wrap
...
>>> @log
... def add(x: int, y: int) -> int:
...     return x + y
...
wrap: 1714291384800, func: 1714291385208
>>> add.__name__
'add'
>>> add.__annotations__
{'x': <class 'int'>, 'y': <class 'int'>, 'return': <class 'int'>}
>>> id(add), id(add.__wrapped__)
(1714291384800, 1714291385208)
```

> 装饰器functools.wrap将原函数\_\_module\_\_、\_\_name\_\_、\_\_doc\_\_、\_\_annotations\_\_等属性复制到包装函数，还用\_\_wrapped\_\_存储原始函数或上一装饰器返回值。可由此判断并绕开装饰器对单元测试的干扰。

##### 类型装饰器

装饰器同样可用于类型，这里的区别无非是接收参数为类型对象而已。

```
>>> def log(cls):
...     class wrapper:
...             def __init__(self, *args, **kwargs):
...                     self.__dict__["inst"] = cls(*args, **kwargs)
...             def __getattr__(self, name):
...                     value = getattr(self.inst, name)
...                     print(f"get: {name} = {value}")
...                     return value
...             def __setattr__(self, name, value):
...                     print(f"set: {name} = {value}")
...                     return setattr(self.inst, name, value)
...     return wrapper
...
>>> @log
... class X: pass
...
>>> x = X()
>>> x.a = 1
set: a = 1
>>> x.a
get: a = 1
1
```

> 因每次都新建wrapper类型，故可应用于不同类型目标。

当然，未必要返回包装类，也可用函数代替，间接调用目标构造方法创建实例。

```
>>> def log(cls):
...     def wrap(*args, **kwargs):
...             o = cls(*args, **kwargs)
...             print(f"log: {o}")
...             return o
...     return wrap
...
>>> @log
... class X: pass
...
>>> X()
log: <__main__.X object at 0x0000018F23BA6780>
<__main__.X object at 0x0000018F23BA6780>
```

#### 1.2、应用

利用装饰器功能，可编写各种辅助开发工具。如调试跟踪、性能测试、内存检测等。也可用于模式设计，改善代码结构。

##### 调用跟踪

记录目标调用参数、返回值，以及执行次数和执行时间等信息。

```
>>> def call_count(fn):
...     def counter(*args, **kwargs):
...             counter.__count__ += 1
...             return fn(*args, **kwargs)
...     counter.__count__ = 0
...     return counter
...
>>> @call_count
... def a(): pass
...
>>> @call_count
... def b(): pass
...
>>> a(); a(); a.__count__
2
>>> b(); b(); b(); b.__count__
3
```

在标准库中有类似的应用，通过缓存结果减少目标执行次数。

```
In [1]: import functools

In [2]: @functools.lru_cache(10)
   ...: def test(x):
   ...:     time.sleep(x)
   ...:

In [3]: import time

In [4]: %%time
   ...: for i in range(1000): test(1)
   ...:
   ...:
Wall time: 1 s
```

##### 属性管理

为目标添加额外属性，在原有设计上以装配方式混入(mixin)其他功能组。

```
>>> def pet(cls):
...     cls.dosomething = lambda self: None
...     return cls
...
>>> @pet   # 添加类似宠物功能
... class Parrot: pass
...
```

> 比起前面章节提及的直接添加基类(\_\_bases\_\_)方式，装饰器更好点。
>
> 还可用代理类拦截(\_\_getattr\_\_、\_\_setattr\_\_)对目标进行访问，比如实现只读功能。

##### 实例管理

替代目标构造方法，拦截实例的创建。用于实现对象缓存，或单例模式。

```
>>> def singleton(cls):
...     inst = None
...     def wrap(*args, **kwargs):
...             nonlocal inst
...             if not inst: inst = cls(*args, **kwargs)
...             return inst
...     return wrap
...
>>> @singleton
... class X: pass
...
>>> X() is X()
True
```

##### 部件注册

在很多Web框架里，用装饰器替代配置文件实现路由注册。

```
>>> class App:
...     def __init__(self):
...             self.routers = {}
...     def route(self, url):
...             def register(fn):
...                     self.routers[url] = fn
...                     return fn
...             return register
...
>>> app = App()
>>> @app.route("/")
... def index(): pass
...
>>> @app.route("/help")
... def help(): pass
...
>>> app.routers
{'/': <function index at 0x0000018F23BAAD08>, '/help': <function help at 0x0000018F23BAAD90>}
```

### 2、描述符

描述符一致被解释器当秘密武器使用。前面提及的属性、方法绑定等内部机制都是描述符在起作用。

不同于实例通过拦截方法（\_\_getattr\_\_等），描述符以单个属性出现，并针对该属性的不同访问行为自动做出响应。最重要的是，描述符能"感知"通过什么引用该属性，从而和目标建立绑定关联。

```
>>> class descriptor:
...     def __set_name__(self, owner, name):
...             print(f"name: {owner.__name__}.{name}")
...             self.name = f"__{name}__"
...     def __get__(self, instance, owner):
...             print(f"get: {instance}, {owner}")
...             return getattr(instance, self.name, None)
...     def __set__(self, instance, value):
...             print(f"set: {instance}, {value}")
...             setattr(instance, self.name, value)
...     def __delete__(self, instance):
...             print(f"del: {instance}")
...             raise AttributeError("delete is disabled")
...
>>> class X:
...     data = descriptor()
...
name: X.data
```

描述符属性必须定义为类型成员，所以其自身不适合存储实例相关的状态。在创建属性时，\_\_set\_name\_\_方法被调用，并可通过参数获知目标类型（owner）,以及属性名称。

以类型或实例访问描述符属性时，\_\_get\_\_被自动调用，且会接收到类型和实例引用。

```
>>> x = X()
>>> x.data = 100
set: <__main__.X object at 0x0000014E7E576320>, 100
>>> x.data
get: <__main__.X object at 0x0000014E7E576320>, <class '__main__.X'>
100
```

方法\_\_set\_\_、\_\_delete\_\_仅在实例引用时被调用。以类型引用进行赋值或删除操作，会导致描述符属性被替换或删除。

```
>>> X.data
get: None, <class '__main__.X'>
>>> X.data = 100   # 以类型引用赋值，导致描述符属性被替换
>>> X.data
100
```

还有，将描述符属性赋值给变量或传参时，实际结果是\_\_get\_\_方法的返回值。

```
>>> x = X()
>>> x.data = 100
set: <__main__.X object at 0x0000014E7E576320>, 100
>>> o = x.data
get: <__main__.X object at 0x0000014E7E576320>, <class '__main__.X'>
>>> o
100
```

##### 数据描述符

如果定义了\_\_set\_\_或\_\_delete\_\_方法，那么我们便称其为数据描述符。而仅有\_\_get\_\_d的则是非数据描述符。这两者的区别在于，数据描述符属性的优先级高于实例名字空间中的同名成员。

```
>>> class descriptor:
...     def __get__(self, instance, owner):
...             print("__get__")
...     def __set__(self, instance, value):
...             print("__set__")
...
>>> class X:
...     data = descriptor()
...
>>> x = X()
>>> x.__dict__["data"] = 0
>>> x.data = 100
__set__
>>> x.data
__get__
```

但如果注释\_\_set\_\_，使其变成非数据描述符，在执行时，结果不一样了。

```
>>> class descriptor:
...     def __get__(self, instance, owner):
...             print("__get__")
...
>>> class X:
...     data = descriptor()
...
>>> x = X()
>>> x.data = 10  # 同名实例成员的优先级高于非数据描述符
>>> x.data
10
>>> vars(x)
{'data': 10}
```

属性（property）就是数据描述符。就算没有提供setter方法，但\_\_set\_\_依然存在，所以其优先级高于同名实例成员。

```
>>> p  = property()
>>> p.__get__
<method-wrapper '__get__' of property object at 0x0000014E7E55F0E8>
>>> p.__set__
<method-wrapper '__set__' of property object at 0x0000014E7E55F0E8>
>>> p.__delete__
<method-wrapper '__delete__' of property object at 0x0000014E7E55F0E8>
```

##### 方法绑定

因为函数默认实现了描述符协议，所以当以实例或类型访问方法时，\_\_get\_\_首先被调用。

类型和实例作为参数被传入\_\_get\_\_，从而拦截绑定目标（\_\_self\_\_），如此就将函数包装成绑定方法对象返回。实际被执行的，就是这个会隐式传入第一参数的包装品。

```
>>> class X:
...     def test(self, o): print(o)
...
>>> x = X()
>>> x.test
<bound method X.test of <__main__.X object at 0x0000014E7E576668>>
>>> x.test(123)
123
>>> m = x.test.__get__(x, type(x))
>>> X.test(m.__self__, 123)
123
>>> m
<bound method X.test of <__main__.X object at 0x0000014E7E576668>>
>>> m.__self__, m.__func__
(<__main__.X object at 0x0000014E7E576668>, <function X.test at 0x0000014E7E577598>)
```

> x.test(123)实际分成了2个步骤来执行：
>
>* x.test(123) ---> m = x.test.\_\_get\_\_(x, type(x)) #将函数包装为绑定方法
>
>* m(123) ---> X.test(m.\_\_self\_\_, 123) # 执行时，隐式将self/cls参数传给目标函数。
>
> 在绑定方法对象内，\_\_self\_\_和\_\_func\_\_存储了执行所需的信息。

### 3、元类

元类（metaclass）制造了所有类型对象，并将其与逻辑上的父类关联起来。所以，存在两条线：创建和逻辑。

系统默认的元类是type，所以可描述如下过程。

```
base     = type("base", object, ...)
class    = type("class", base, ...)
instance = class(...)
instance.__class__ is class and isinstance(instance, class)
class.__class__    is type  and isinstance(class, type)
base.__class__     is type  and isinstance(base, type)
```

> 属性\_\_class\_\_表明该对象由何种类型创建，可用type(o)返回。

实际上，可用type直接创建类型对象。

```
>>> User = type("User",(object,),{
... "__init__": lambda self, name: setattr(self, "name", name),
... "test"    : lambda self: print(self.name),
... "table"   : "user",
... })
>>> User.__dict__
mappingproxy({'__init__': <function <lambda> at 0x0000014E7E5607B8>, 'test': <function <lambda> at 0x0000014E7E577620>, 'table': 'user', '__module__': '__main__', '__dict__': <attribute '__dict__' of 'User' objects>, '__weakref__': <attribute '__weakref__' of 'User' objects>, '__doc__': None})
>>> u = User("yuhen")
>>> u.test()
yuhen
>>> u.__dict__
{'name': 'yuhen'}
```

#### 3.1、 自定义

可自定义元类，以控制类型对象的生成过程。通常自type继承，以Meta为后缀名。

```
>>> class DemoMeta(type): pass
...
>>> class X(metaclass = DemoMeta): pass
...
>>> X.__class__
<class '__main__.DemoMeta'>
```

与普通类型构造过程类似，可覆盖构造和初始化方法的定制创建过程。

```
>>> class DemoMeta(type):
...     @classmethod
...     def __prepare__(cls, name, bases):
...             print(f"__prepare__: {name}")
...             return {"__make__": "make in DemoMeta"} # 定制名字空间
...     def __new__(cls, name, bases, attrs):
...             print(f"__new__: {name}, {bases}, {attrs}") 
...             return type.__new__(cls, name, bases, attrs) # 创建并返回类型对象
...     def __init__(self, name, bases, attrs):
...             print(f"__init__: {self}")
...             return type.__init__(self, name, bases, attrs) # 初始化后，返回类型对象
...     def __call__(cls, *args, **kwargs):
...             print(f"__call__: {cls}, {args}, {kwargs}")
...             return type.__call__(cls, *args, **kwargs) #调用类型对象创建实例过程，返回实例
```

> 按需去实现相关方法。
>
> \_\_prepare\_\_用来创建类型对象名字空间（X.\_\_dict\_\_），可往里添加点或使用自定义字典类型。随后，该名字空间会填充其他属性成员，并继续传给\_\_new\_\_和\_\_init\_\_方法。
>
> ```
> class X(metaclass = DemoMeta) --->
> 		namespace = DemoMeta.__prepare__(name, ...)
> 			    X = DemoMeta.__new__(name, bases, namespace{attrs})
> 			    DemoMeta.__init(X, ...)
> ```
>
> \_\_call\_\_在类型对象(X)创建其所属的实例时被调用。实际上X.\_\_new\_\_、X.\_\_init\_\_方法就是由此调用的，可用于拦截实例的创建。
>
> ```
> o = X(1, 2)  ---> 
> 	o = DemoMeta.__call__(X, 1, 2) --->
> 		o = X.__new__(...)
> 			X.__init__(o, ...)
> ```

```
>>> class X(metaclass = DemoMeta):
...     data = 100
...     def __init__(self, x, y): pass
...     def test(self): pass
...
__prepare__: X
__new__: X, (), {'__make__': 'make in DemoMeta', '__module__': '__main__', '__qualname__': 'X', 'data': 100, '__init__': <function X.__init__ at 0x0000014E7E577950>, 'test': <function X.test at 0x0000014E7E5779D8>}
__init__: <class '__main__.X'>
```

```
>>> o = X(1, 2)
__call__: <class '__main__.X'>, (1, 2), {}
>>> o
<__main__.X object at 0x0000014E7E576B00>
```

当然，这里也可用函数或其他可调用对象代替。

```
>>> def demo_meta(name, bases, attrs):
...     print(f"{name}, {bases}, {attrs}")
...     return type(name, bases, attrs)
...
>>> class X(metaclass = demo_meta):
...     data = 100
...     def __init__(self, x, y): pass
...     def test(self): pass
...
X, (), {'__module__': '__main__', '__qualname__': 'X', 'data': 100, '__init__': <function X.__init__ at 0x0000014E7E577AE8>, 'test': <function X.test at 0x0000014E7E577B70>}
```

> 函数只是拦截调用，类型对象的创建依然使用type完成。

##### 参数

还可向元类传递参数，实现功能定制。

```
>>> class DemoMeta(type):
...     def __new__(meta, name, bases, attrs, **kwargs):
...             print(kwargs)
...             return type.__new__(meta, name, bases, attrs)
...
>>> class X(metaclass = DemoMeta, a = 1, b = "abc"):
...     def test(self): pass
...
{'a': 1, 'b': 'abc'}
```

##### 继承

类型对象的元类设置顺序如下:

1、从metaclass显式指定；

2、从基类继承；

3、默认元类type。

```
>>> class DemoMeta(type): pass
...
>>> class X(metaclass = DemoMeta): pass
...
>>> class Y(X): pass   # 从基类继承元类
>>> type(Y)
<class '__main__.DemoMeta'>
```

如是多继承，则必须保证能继承所有的祖先元类。

```
>>> class AMeta(type): pass
...
>>> class BMeta(type): pass
...
>>> class A(metaclass = AMeta): pass
...
>>> class B(metaclass = BMeta): pass
...
>>> class C(A, B): pass
...
TypeError: metaclass conflict: the metaclass of a derived class must be a (non-strict) subclass of the metaclasses of all its bases
```

因为\_\_class\_\_只能指向一个元类，所以除非祖先元类之间存在继承关系，否则必然会由于无法访问而导致错误发生。为此，须新建一个元类，让其继承所有的祖先元类。

```
>>> class CMeta(AMeta, BMeta): pass
...
>>> class C(A, B, metaclass = CMeta): pass
```

#### 3.2、应用

基于元类，可实现很多魔法，让对象拥有很高的隐式“智能”，但会提升代码复杂度。

另，元类虽能像普通类型那样为自己的实例提供共享成员，但依旧避免使用，能让元类专注于类型创建和管理，不要涉及逻辑最好。

##### 静态类

阻止类型创建实例对象。

这里直接用\_\_call\_\_拦截实例的创建即可。

```
>>> class StaticClassMeta(type):
...     def __call__(cls, *args, **kwargs):
...             raise RuntimeError("can't create object for static class")
...
>>> class X(metaclass = StaticClassMeta): pass
...
>>> x = X()
RuntimeError: can't create object for static class
```

##### 密封类

阻止类被继承。可使用该元类，也就是不能被继承类添加到集合里，作为后续基类判断条件即可。

```
>>> class SealedClassMeta(type):
...     types = set()
...     def __init__(cls, name, bases, attrs):
...             if cls.types & set(bases): raise RuntimeError("can't inherit from sealed class")
...             cls.types.add(cls)
...
>>> class A(metaclass = SealedClassMeta): pass
...
>>> class B(A): pass
...
RuntimeError: can't inherit from sealed class
>>>
```

### 注解

注解（annotation）为函数参数、返回值，以及模块和类型属性添加额外的元数据。

```
>>> def add(x: int, y: int) -> int:
...     return x + y
...
>>> add.__annotations__
{'x': <class 'int'>, 'y': <class 'int'>, 'return': <class 'int'>}
```

其本质仅是一种可编程和可执行的注释，在编译期被提取，并与对象相关联。在运行期，其对解释器指令的执行没有任何影响和约束。

```
>>> add("abc", "d")   # 并不受注解int类型约束
'abcd'
>>> import dis
>>> dis.dis(add)      # 字节码指令中也没有任何注解相关内容
  2           0 LOAD_FAST                0 (x)
              2 LOAD_FAST                1 (y)
              4 BINARY_ADD
              6 RETURN_VALUE
>>> dis.dis(compile("add('abc', 'd')","","exec")) # 和普通函数调用一致
  1           0 LOAD_NAME                0 (add)
              2 LOAD_CONST               0 ('abc')
              4 LOAD_CONST               1 ('d')
              6 CALL_FUNCTION            2
              8 POP_TOP
             10 LOAD_CONST               2 (None)
             12 RETURN_VALUE
```

注解内容可是任何对象和表达式。其可应用于变量，但不可用于lambda函数。

```
>>> x: int  = 123   # 注意和等号后的初始化值分开
>>> __annotations__
{'x': <class 'int'>}
>>> def test(x: {'type': int, 'range': (0, 10)} =5): # 等号后为默认值
...     pass
...
>>> test.__annotations__
{'x': {'type': <class 'int'>, 'range': (0, 10)}}
>>> test.__defaults__
(5,)
>>> class X:
...     data: int = 10
...     def test(self, o: str) -> str: pass
...
>>> X.__annotations__
{'data': <class 'int'>}
>>> X.test.__annotations__
{'o': <class 'str'>, 'return': <class 'str'>}
```

##### 用途

注解最常见的用途就是作为类型和取值范围的检查条件，在ORM框架中常见。

```
>>> def test(x: (int, 0, 10)):
...     if __debug__:
...             ann_x = test.__annotations__["x"]
...             assert isinstance(x, ann_x[0])
...             assert ann_x[1] <=x <= ann_x[2]
...     return
```

> 更好的做法就是使用装饰器完成，成为一个普通检查器。

其还可为代码编辑器(PyCharm)和帮助生成工具(pydoc)提供更详细的分类信息。另有MyPy等实验型解释器，借助注解实现编译器静态类型检查。

> 标准库typing提供了基于注解的编程支持。

