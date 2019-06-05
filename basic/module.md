# 模块

### 定义

模块是顶层代码组织单元，其提供大粒度封装和复用。

通常每个模块对应一个源码文件。其中定义的变量、函数、类型等，都属于其私有成员。

模块在首次导入时，被编译成字节码。随后解释器开始创建模块实例，执行初始化语句，构建内部成员。模块不仅是代码组织形式，还是运行期对象，其为成员提供全局名字空间。

无论被导入多少次，每个模块在整个解释器进程内部仅有一个实例存在。重复导入只是引用已存在的实例，并不会再次执行初始化过程。

demo.py

```
x = 1234


def hello(): pass


class User:
    pass
```

```
>>> import demo
>>> import types
>>> isinstance(demo, types.ModuleType)
True
```

##### 初始化

初始化过程就是将模块里的代码按序执行一面。初始化过程很简单：普通语句直接执行，类似于def、class等则创建函数和类型对象。最终这些成员都保存在模块的全局名字空间内。

```
>>> import demo
>>> vars(demo)
{'__name__': 'demo',origin='D:\\workspace\\Python\\study_notes\\demo.py'), '__file__': 
'D:\\workspace\\Python\\study_notes\\demo.py', '__cached__': 'D:\\workspace\\Python\\study_notes\\__pycache__\\demo.cpython-37.pyc', ... 
, 'x': 1234, 'hello': <function hello at 0x000001D5FC04F510>, 'User': <class 'demo.User'>}
```

> 示例中全局变量x，初始化过程为其赋予一个初始值。以后执行中，它可能改变。如每次都初始化，则可能会将改变的状态丢失。所以，重复导入只是引用，不会再次执行。

##### 名字空间

模块的全局名字空间对应\_\_dict\_\_属性，其中内部不能被直接访问，且不会别dir输出。

```
>>> vars(demo) is demo.__dict__
True
```

> 函数vars返回目标__dict__属性。参数为空时，类似于locals返回当前名字空间。而dir则返回目标，或当前名字空间可访问的名字列表。

当def创建函数对象时，会将所有模块的名字空间作为构造参数。这意味着，无论后续将该函数传递到何处，还是"绑定"给其他模块，都不能改变函数内部globals总是返回出生地的名字空间。

demo.py

```
>>> import demo
>>> demo.test() is vars(demo)  # 任何地方调用，函数依然返回出生地名字空间
True
>>> import types
>>> m = types.ModuleType("abc")
>>> m.test = demo.test
>>> m.test() is vars(demo)     # 即使绑定给其他模块，依然无法改变这点
True
>>> m.test.__module__
'demo'
>>> dir(m)   
['__doc__', '__loader__', '__name__', '__package__', '__spec__', 'test']
```

##### 名字

可通过\_\_name\_\_、\_\_file\_\_获取所在模块信息。或者，利用\_\_module\_\_属性返回类型、函数等对象的的定义模块名称。

```
>>> x = demo.test
>>> x.__module__
'demo'
```

> 建议使用inspect.getmodule获取目标对象定义模块引用。

正常情况下，模块名字对应原文件名。但有个例外，就是当模块作为程序入口时，会被赋予"_\_\main\_\_"名字。

demo.py

```
print(__name__)
```

```
$ python demo.py
__main__
$ python -c "import demo"
demo
```

如此，只需要编写如下语句，则可让其自动适应身份变化。

```
def main():
    test()


if __name__ == "__main":
    main()

```

> 创建语句def并不会执行main函数，只有作为入口时才会进入流程。如果作为普通模块导入，则因名字不匹配而被忽略。另外，不建议将main函数内容直接放在if语句内，因为这回直接操作全局名字空间，可能引发不必要的错误。

### 导入

按LEGB规则，除包含内置函数的\_\_builtins\_\_模块外，所有名字搜索都不能超出当前模块。所有任何外部目标，都必须提前导入当前名字空间，否则无法访问。

导入步骤：

1. 搜索目标模块文件；
2. 按需编译目标模块；
3. 创建模块实例，执行初始化；
4. 将模块实例保存到全局列表；
5. 在当前名字空间建立引用。

> 模块实例一旦创建，就保存在sys.modules，后续导入直接引用。
>
> 内置模块\_\_builtins\_\_由解释器自动导入。

```
>>> for i in range(3):
...     import sys       #重复导入，不会新建实例。
...     print(id(sys))
... 
2709757489048
2709757489048
2709757489048
```

#### 搜索

导入所使用的模块名不能包含路径信息，这需要系统提供搜索方式和匹配规则。其中搜索路径，由解释器在启动时，按优先级整理到sys.path列表中。

搜索路径列表：

1. 程序根目录；
2. 环境变量(PYTHONPATH)设定的路径列表；
3. 标准库目录；
4. 第三方扩展库等附加路径(site-specific)。

> 附加路径由site模块添加，在解释器启动时自动执行(可用参数-S禁用)。除系统site-packages外，还包括用户相关目录。

```
D:\>python -c "import sys, pprint; pprint.pprint(sys.path)"
['',  # 根目录
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\python37.zip',  # 标准库
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\DLLs',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37',
 'C:\\Users\\user\\AppData\\Roaming\\Python\\Python37\\site-packages',  
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages', # 系统扩展库目录
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\win32',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\win32\\lib',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\Pythonwin']

D:\>python -S -c "import sys, pprint; pprint.pprint(sys.path)" # 禁用site, 缺少site-packages
['',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\python37.zip',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\DLLs',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib',
 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37']
```

另可设置环境变量PYTHONPATH，然后请注意优先级。

虽可在运行期向sys.path添加搜索路径。但对扩展库而言，更简便方式就是路径配置文件。将所有要添加的路径按行保存在.pth文本文件内，并放在site-packages等目录中，剩下的就由site完成。

site-packages/demo.pth

```
# comment              # 注释

D://workspace//Python  # 绝对路径
mypkg                  # 相对路径 
```

```
D:\>python -m site
sys.path = [
    'D:\\',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\python37.zip',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\DLLs',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37',
    'C:\\Users\\user\\AppData\\Roaming\\Python\\Python37\\site-packages',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages',               # 系统扩展库目录 
    'D:\\workspace\\Python',  # 路径配置添加目录
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\mypkg',        # 路径配置添加目录 
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\win32',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\win32\\lib',
    'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\Pythonwin',
]
USER_BASE: 'C:\\Users\\user\\AppData\\Roaming\\Python' (exists)
USER_SITE: 'C:\\Users\\user\\AppData\\Roaming\\Python\\Python37\\site-packages' (exists)
ENABLE_USER_SITE: True
```

> 注意：site会过滤掉配置重复的目录或文件，以及不存在的路径。

整理好搜索路径列表后，就需要进一步匹配模块文件名。问题是，语言规范并未定义不同扩展名的同名模块匹配次序，所以安全起见，不要在同一路径放同名模块。

* 源码或字节码文件(.py、.pyc)；
* 包目录名；
* 其他语言编写的扩展模块(.dll、.so等)。

#### 编译

确定模块文件路径后，编译器优先选择已编译的字节码文件，这有助提升载入性能。

```
D:\workspace\Python\study_notes>python -c "import demo"

D:\workspace\Python\study_notes>dir __pycache__
 Volume in drive D is DATA
 Volume Serial Number is 94EF-A3E1

 Directory of D:\workspace\Python\study_notes\__pycache__

2019/06/04  17:10    <DIR>          .
2019/06/04  17:10    <DIR>          ..
2019/06/04  17:10               261 demo.cpython-37.pyc
               1 File(s)            261 bytes
               2 Dir(s)  139,498,631,168 bytes free

D:\workspace\Python\study_notes>python -OO -c "import demo"

D:\workspace\Python\study_notes>dir __pycache__
 Volume in drive D is DATA
 Volume Serial Number is 94EF-A3E1

 Directory of D:\workspace\Python\study_notes\__pycache__

2019/06/04  17:11    <DIR>          .
2019/06/04  17:11    <DIR>          ..
2019/06/04  17:11               261 demo.cpython-37.opt-2.pyc
2019/06/04  17:10               261 demo.cpython-37.pyc
               2 File(s)            522 bytes
               2 Dir(s)  139,498,631,168 bytes free
```

##### 无源码部署

运行时，源码文件不是必需的。解释器需要的字节码及相关元数据，编译后存储在内存或保存在字节码文件中。部署时，可移除源码，改为直接部署字节码文件。

> 不能直接使用\_\_pycache\_\_下的文件，它的文件命名方式不适合作为直接部署使用。缓存文件很容易被dis反编译，并不能有效保护代码安全。

main.py

```
import demo

demo.hello()
```

demo.py

```
def hello():
    print("hello world!")
```

使用compileall编译所有源文件。

```
$ python -m compileall -b -l *.py  # 编译所有文件（以源文件名保存，不含目录)
$ mkdir release
$ mv *.pyc release/   # 将生成的.pyc文件转移到独立目录
$ cd release/
$ ls
demo.pyc  main.pyc
$ python main.pyc    # 使用解释器执行
hello world!
```

#### 引用

可同时引入多个模块，或模块中的多个成员，当名字发生冲突时，可用as命名别名。

```
In [1]: import sys, inspect as reflect

In [2]: from string import ascii_letters as letters, ascii_lowercase as lowercase

In [3]: %whos    # Ipython命令，查看环境变量信息
Variable    Type      Data/Info
-------------------------------
letters     str       abcdefghijklmnopqrstuvwxy<...>BCDEFGHIJKLMNOPQRSTUVWXYZ
lowercase   str       abcdefghijklmnopqrstuvwxyz
reflect     module    <module 'inspect' from 'c<...>thon37\\lib\\inspect.py'>
sys         module    <module 'sys' (built-in)>
```

> 别名仅在当前名字空间有效，并不影响被导入的模块或其成员。别名还可用来缩写处理。如导入成员过多，可用括号或续行符分成多行。

导入语句放在文件头部，以“标准库、扩展库、当前程序模块”分别排列。

```
>>> import sys     # 标准库
>>> import inspect
>>> import psutil  # 扩展库
>>> import numpy
>>> import db      # 应用程序模块
>>> import demo
```

成员导入时，其名字和其他模块或当前模块成员冲突可能性很大。且对模块重新载入有影响，故不建议在模块级别使用。以星号导入目标模块全部成员的行为，更是被规范所禁止。

```
>>> from sys import version
>>> print(version)
3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 22:22:05) [MSC v.1916 64 bit (AMD64)]
>>> version = "ver: 1.0"   # 在当前名字空间内被覆盖，同一名字仅一个关联
>>> print(version)
ver: 1.0
```

在函数内部，因其名字空间范围小，且生命周期短，故影响较小。这样，使用成员导入可减少代码，且能提升性能。Python3在函数内不允许用星号导入。

```
>>> def test():
...     from sys import version
...     print(version)
...
>>> test()
3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 22:22:05) [MSC v.1916 64 bit (AMD64)]
```

下面比较下模块导入和成员导入指令的差异性：

```
>>> def a():
...     import sys
...     print(sys.version)
...
>>> def b():
...     from sys import version
...     print(version)
...
>>> import dis
>>> dis.dis(a)
  2           0 LOAD_CONST               1 (0)
              2 LOAD_CONST               0 (None)
              4 IMPORT_NAME              0 (sys)
              6 STORE_FAST               0 (sys)

  3           8 LOAD_GLOBAL              1 (print)
             10 LOAD_FAST                0 (sys)   # 需要两条指令引用version
             12 LOAD_ATTR                2 (version)
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 LOAD_CONST               0 (None)
             20 RETURN_VALUE
>>> dis.dis(b)
  2           0 LOAD_CONST               1 (0)
              2 LOAD_CONST               2 (('version',))
              4 IMPORT_NAME              0 (sys)
              6 IMPORT_FROM              1 (version)
              8 STORE_FAST               0 (version)
             10 POP_TOP

  3          12 LOAD_GLOBAL              2 (print)
             14 LOAD_FAST                0 (version) # 单条执行，且是FAST
             16 CALL_FUNCTION            1
             18 POP_TOP
             20 LOAD_CONST               0 (None)
             22 RETURN_VALUE
```

如多次调用，对比更明显。

```
In [1]: def a():
   ...:     import sys
   ...:     for i in range(100):
   ...:         s = sys.version
   ...:     return s
   ...:

In [2]: def b():
   ...:     from sys import version
   ...:     for i in range(100):
   ...:         s = version
   ...:     return s
   
In [4]: %timeit a()
4.08 µs ± 53.1 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)  

In [7]: %timeit b()
2.74 µs ± 80.1 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
```

##### 导出列表

星号导入是将目标模块名字空间内的所有成员全部导入，这肯定有用不到的东西。但无法阻止用户使用星号。结果就是导致调用方名字空间被污染。

demo.py

```
import sys

x = 1234


def hello():
    print("hello world!")

```

main.py

```
from demo import *

print(dir())

```

```
$ python main.py   # demo的成员 以及导入的sys，都进入调用者main的名字空间
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'hello', 'sys', 'x']
```

简单做法就是模块成员私有化，也就是名字添加下划线前缀。私有成员不会被星号导入。

demo.py

```
import sys as _sys  # 私有别名？

_x = 1234   # 私有成员


def hello():
    print("hello world!")
```

```
$ python main.py
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'hello']
```

这样不太方便也不很美观，正确做法就是模块添加\_\_all\_\_声明，明确指定那些可以被星号导入到成员名字列表。为空则表示不会导入任何成员。

demo.py

```
__all__ = ["hello"]
import sys
x = 1234
def hello(): pass
```

```
$ python main.py    # 仅 __all__指定成员被星号导入
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'hello']
```

无论私有成员，还是_\_all\_\_导出列表，都不影响显示成员导入。

demo.py

```
__all__ = []
import sys
_x = 1234
def hello(): pass
```

main.py

```
from demo import _x, hello
print(dir())
```

```
$ python main.py    # 仅 __all__指定成员被星号导入
['__annotations__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_x', 'hello']
```

##### 动态导入

使用import是明确知道目标模块名字。但有时，只有运行期才能动态获取，这需要以其他方式导入。

方法一：使用exec 动态执行，随后从sys.modules中获取模块实例。

```
>>> def test(name):
...     exec(f"import {name}")
...     m = sys.modules[name]
...     print(m)
>>> test("inspect")
<module 'inspect' from 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\inspect.py'>
```

方法二：使用importlib库。

```
>>> import importlib
>>> def test(name):
...     m = importlib.import_module(name)
...     print(m)
...
>>> test("inspect")
<module 'inspect' from 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\inspect.py'>
```

##### 重新载入

模块一旦导入，解释器不关心源文件是否被修改。但有时需要在不重启进程情况下，对某些模块实现热更新。

既然模块实例保存在sys.modules列表，可否移除实例引用，然后重新导入呢？

demo.py

```
x = 1234
```

```
In [1]: import demo

In [2]: demo.x
Out[2]: 999

In [3]: import sys

In [4]: del sys.modules["demo"]  # 从sys.modules移除，使其回收
								 # 修改源码，将999改为888，然后保存						
In [5]: import demo              # 重新导入

In [6]: demo.x                   # 更改生效
Out[6]: 888

In [7]: m = demo                  # 另一引用

In [8]: m.x
Out[8]: 888

In [9]: del sys.modules["demo"]  # 移除
                                 # 修改源码，将888改为777，然后保存
In [10]: import demo             # 再次导入 

In [11]: demo.x
Out[11]: 777

In [12]: m.x                     # 另一引用m没有影响。热更新失败
Out[12]: 888

In [13]: m is demo               # 不同模块实例
Out[13]: False
```

由于m的存在，导致旧demo模块实例不会回收。这样的话，更新后就各自引用不同模块实例。

标准库提供了importlib.reload。

```
In [1]: import importlib

In [2]: import demo

In [3]: m = demo   # 另一引用

In [4]: m.x
Out[4]: 777     #修改源码，将777改为666，然后保存 
                           
In [5]: importlib.reload(demo)  # 重新载入模块
Out[5]: <module 'demo' from 'D:\\workspace\\Python\\study_notes\\demo.py'>

In [6]: demo.x  # 热更新成功
Out[6]: 666

In [7]: m.x   # 其他引用也同步更新
Out[7]: 666

In [8]: demo is m  # 引用同一模块实例
Out[8]: True
```

但reload只对模块引用有效，对成员引用无法更新，因为reload不会递归修改成员。

因此，应避免直接引用其他模块成员，而通过模块直接访问。当然，在函数内部，成员引用生命周期短，对热更新影响有限。

另外，模块重载会重新初始化，会导致状态丢失，并引发其他错误，请慎重对待。



### 包

模块用来组织代码，包就是用来组织模块。

将多个源文件放在同一目录，就构成了包。包能隐藏内部文件的组织结构，而仅暴露必要的用户接口，毕竟不是所有模块都对外提供服务。

```
│   demo.py
│   main.py
│
└───lib
        demo.py
```

```
>>> import lib
>>> lib
<module 'lib' (namespace)>
>>> vars(lib)
{'__name__': 'lib', '__doc__': None, '__package__': 'lib', '__file__': None, '__path__': _NamespacePath(['D:\\workspace\\Python\\study_notes\\lib', 'C:\\Users\\user\\AppData\\Local\\Programs\\Python\\Python37\\lib\\site-packages\\win32\\lib']),...}
```

从包名字空间看，仅导入包并不能直接访问其内部模块，须显示导入。

```
>>> import lib.demo
>>> lib.demo.__package__
'lib'
>>> lib.demo.__name__
'lib.demo'
```

#### 初始化

包是另类的模块实例，其对应源文件\_\_init\_\_.py。虽在python3不再必须，但可用来执行初始化操作，比如提供对外接口，解除用户对内部模块的直接依赖。

```
│   demo.py
│   main.py
│
└───lib
    │   demo.py
    │   __init__.py
```

```
>>> import lib
>>> lib.__file__
'D:\\workspace\\Python\\study_notes\\lib\\__init__.py'
```

初始化文件在包或其内部模块首次导入时自动执行，且仅执行一次。

\_\_init\_\_.py

```
print("init")
```

demo.py

```
print("demo")
```

```
>>> import lib.demo
init
demo
>>> import lib
```

> 注意：重载包内部模块，不会再次执行初始化文件，但重载包则会。

还可在包内创建\_\_main\_\_.py文件，作为直接执行时的入口。

_\_main\_\_.py

```
print("main")
```

```
$ python -m lib  # 以包方式运行，自动执行初始化文件
init             # 先导入包，后执行__main__.py
main

$ python lib  # 以普通方式运行，对应__main__入口模块
main          # 不会自动执行初始化文件，除非显示导入__init__

$ python -c "import lib"  #__main__.py对普通导入没影响
init             
```

##### 隐藏结构

既然初始化文件构成了包名字空间，那么只要将要公开给用户的模块或成员导入过来，即可解除用户对具体模块的依赖。

demo.py

```
def hello():
    print("Hello, World!")
```

_\_init\_\_.py

```
from .demo import hello
```

```
>>> import lib
>>> lib.hello()
Hello, World!
```

这样，用户只依赖包和初始化文件中导入的特定成员。无论怎么重构代码和重新组织模块文件，都不影响用户调用接口。即使将公开成员升级到新版本，也可用别名方式映射到原名字，实现多版本的单一名称。

demo.py

```
def hello():
    print("Hello, World!")


def hello2():
    from sys import version
    print(version)
    hello()
```

_\_init\_\_.py

```
from .demo import hello2 as hello
```

还可将整个包压缩成ZIP文件分发。

```
lib.v1.zip                 # 任意文件名，可添加版本等信息，然后压缩，并可将源目录命名
│
└───lib                    # 必须包含包目录
    │   demo.py
    │   __init__.py
```

使用前，将.zip文件路径加入到sys.path，这可动态添加或使用路径配置文件。

```
>>> import sys
>>> sys.path.append("./lib.v1.zip") 
>>> import lib
>>> lib.__file__
'.\\lib.v1.zip\\lib\\__init__.py'
```

因解释器不会向.zip中写入缓存文件，故建议用.pyc打包，加快导入速度。

也可将应用程序进行打包，不过须在根目录中放置_\_main\_\_.py作为执行入口

```
app.zip
│
└───__main__.py
│
└───lib                    # 必须包含包目录
    │   demo.py
    │   __init__.py
```

_\_main\_\_.py

```
print("main")

import lib
lib.hello()
```

```
$ zip -r -o app.zip __main__.py lib
$ python app.zip
main
Hello, World!
```

##### 导出列表

同样可在初始化文件中添加\_\_all\_\_星号导出成员列表。除当前文件成员外，还可指定要导出的模块名字。

_\_init\_\_.py

```
__all__ = ["x", "demo"]   # 可被星号导出的模块和本地成员

x = 100
y = 200

>>> from lib import *
>>> dir()
["demo", "x", ...]
```

#### 相对导入

通常将包放置在应用程序根目录，或其他系统目录。但是，搜索路径列表并不包括包目录自身，就导致在包内访问同级模块时，发生找不到文件的状况。

_\_init\_\_.py

```
import demo
demo.hello()
```

demo.py

```
def hello():
    print("Hello, World!")
```

```
>>> import lib
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "D:\workspace\Python\study_notes\lib\__init__.py", line 1, in <module>
    import demo
ModuleNotFoundError: No module named 'demo'
```

就算在包内导入，import语句也严格按照搜索路径列表查找。它会搜索应用程序根目录，但不检查包目录。

修改_\_init\_\_.py

```
import lib.demo
lib.demo.hello()
```

在包内使用全名绝对路径不够友好，为此，Python3引入了相对路径概念，以点前缀表达当前或上级包。

```
/
|
+---lib
|   |   demo.py
|   |   __init__.py
|   |   __main__.py
|   |
|   +---sub
|   |       test.py
```

lib/sub/test.py

```
from ..demo import hello   # 从上级包的demo模块导入hello
```

lib/\_\_init\_\_.py

```
from . import demo        # 从当前包导入demo模块
from .demo import hello   # 从当前包的demo模块导入hello
from .sub import test     # 从当前包的sub子包导入test模块
```

> 相对导入只能用于from子句，可导入模块或成员
>
> 单个.表示当前包，..表示上级包，...表示更上以及的包，依次类推。

相对路径解除了对报名的依赖，其还可用来解除名字冲突。

> 比如导入string时，如使用相对导入则明确表示选择包内(或上级)模块，否则就是选择标准库的同名模块。

##### 优先级

同名除让解释器困扰外，也给代码维护带来了潜在风险。当相同名字的包或模块出现在同一路径下，那么解释器按如下顺序匹配。

1. 如包有初始化文件(包括空文件)，则导入包；
2. 没有初始化文件，则.py或.pyc模块优先

![1559724201028](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1559724201028.png)

#### 拆分

当包内模块文件过多时，可建立子包分组维护，但这需要修改内部的相对导入路径。还有一个方法，同样是将文件分散到多个子目录下，但它们依然属于同一级别的包成员。

> 先将原本放在同一目录下的模块文件分别转移到各自的分组子目录中。
>
> 为让它们继续以包成员的形式存在，须在包\_\_init\_\_.py中修改\_\_path\_\_属性。
>
> 包_\_path\_\_的作用类似sys.path，实现包内搜索路径列表，并进行相同匹配规则。在该搜索列表中默认为包的全路径，只需将子目录的全路径添加进去即可。因子目录中的各模块依然属于原包级别，故模块间的相对导入无须修改。

![1559726056262](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1559726056262.png)

\_\_init\_\_.py

```
from . import hello
from . import demo
import os.path

__path__.extend(os.path.join(__path__[0], d) for d in ("a", "b"))


print(__path__)
print(demo.__file__)
print(hello.__file__)
```

```
>>> import lib
['D:\\workspace\\Python\\study_notes\\lib', 'D:\\workspace\\Python\\study_notes\\lib\\a', 'D:\\workspace\\Python\\study_notes\\lib\\b']
D:\workspace\Python\study_notes\lib\demo.py
D:\workspace\Python\study_notes\lib\hello.py
```

