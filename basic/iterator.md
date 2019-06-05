# 迭代器

#### 概述

迭代就是重复从对象中获取数据，直到结束。在Python中就是用\_\_iter\_\_方法返回一个实现了\_\_next\_\_方法的迭代器对象。

> 实现_\_iter\_\_方法就表示目标为可迭代类型，允许执行手动或自动迭代操作。该方法新建并返回一个迭代器实例。随后，通过调用\_\_next\_\_依次返回结果，直到抛出StopIteration异常来表示结束。

可迭代类型可以是序列数据，也可按前后顺序操作的栈、队列，以及随机返回键值的哈希表等。

```
>>> from collections import abc
>>> issubclass(list, abc.Iterable)
True
>>> isinstance(range(3), abc.Iterable)
True
>>> isinstance(zip([1, 2]), abc.Iterable)
True
```

##### 自定义类型

按迭代器流程及操作方式，来逐一实现，以便更好理解。

```
>>> class Data:
...     def __init__(self, n):
...             self.data = list(range(n))
...     def __iter__(self):
...             return DataIter(self.data)
>>> class DataIter:
...     def __init__(self, data):
...             self.data = data
...             self.index = 0
...     def __next__(self):
...             if not self.data or self.index >= len(self.data):
...                     raise StopIteration
...             d = self.data[self.index]
...             self.index += 1
...             return d
...
>>> d = Data(2)   # 手工迭代
>>> x = d.__iter__()
>>> x.__next__()
0
>>> x.__next__()
1
>>> x.__next__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 7, in __next__
StopIteration

>>> for i in Data(2): print(i)   #自动迭代
...
0
1
>>> x = Data(2).__iter__()    # x是可迭代类型。
>>> x.__iter__()              # x缺少__iter__方法实现，无法使用。当然可添加__iter__方法，如返回自身 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'DataIter' object has no attribute '__iter__'
```

##### 辅助函数

上例的列表容器已经实现迭代接口，直接返回即可。也就是按名字规定，不应直接调用\_\_iter方法，而改用iter函数。

```
>>> class Data:
...     def __init__(self, n):
...             self.data = list(range(n))
...     def __iter__(self):
...             return iter(self.data)
...
>>> Data(2).__iter__()
<list_iterator object at 0x00000265BEC73908>
```

作为辅助函数，iter还可为序列对象(\_\_getitem\_\_)自动创建迭代器包装。

```
>>> class Data:
...     def __init__(self, n):
...             self.n = n
...     def __getitem__(self, index):
...             if index<0 or index>=self.n: raise IndexError
...             return index + 100
...
>>> iter(Data(2))
<iterator object at 0x00000265BEC739B0>
```

iter还可对函数、方法等可调用类型进行包装。其用在网络和文件等I/O数据接收方面，比循环语句更优雅些。

```
>>> x = lambda: input("n:")   # 被__next__调用，无参数
>>> for i in iter(x, "end"): print(i) # 函数x的返回值等于end时结束
...
n:1
1
n:2
2
n:end
```

与\_\_next\_\_方法对应的是next函数，用于手动迭代。

```
>>> x = iter([1, 2])
>>> while True:
...     try:
...             print(next(x))
...     except StopIteration:
...             break
...
1
2
```

##### 自动迭代

使用for循环语句，编译器会生成迭代相关指令，以实现对协议方法的调用。

```
>>> def test():
...     for i in [1, 2]: print(i)
...
>>> import dis
>>> dis.dis(test)
  2           0 SETUP_LOOP              20 (to 22)
              2 LOAD_CONST               1 ((1, 2))
              4 GET_ITER                              #调用__iter__返回迭代器对象(或包装)
        >>    6 FOR_ITER                12 (to 20)  #调用__next__返回数据，结束则跳转
              8 STORE_FAST               0 (i)
             10 LOAD_GLOBAL              0 (print)
             12 LOAD_FAST                0 (i)
             14 CALL_FUNCTION            1
             16 POP_TOP
             18 JUMP_ABSOLUTE            6   # 继续迭代
        >>   20 POP_BLOCK  				     # 迭代结束
        >>   22 LOAD_CONST               0 (None)
             24 RETURN_VALUE
```

#### 生成器

生成器是迭代器进化版本，用函数和表达式替代接口方法。生成器函数的特殊之处在其内部以yield返回迭代数据。无论内部逻辑怎样，其函数调用总是返回生成器对象。随后，以普通迭代器方式继续操作。

```
>>> def test():
...     yield 1
...     yield 2
...
>>> test()   # 函数调用返回生成器对象
<generator object test at 0x00000265BEC7D138>
>>> for i in test(): print(i)  # 以普通方式迭代数据
...
1
2
```

每句yield语句对应一次\_\_next\_\_调用。可分列多条，或出现在循环语句中。只要结束函数流程，就相当于抛出迭代终止异常。

```
>>> def test():
...     for i in range(10):
...             yield i+100
...             if i>=1: return
...
>>> x = test()
>>> next(x)
100
>>> next(x)
101
>>> next(x)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>> for i in test(): print(i)
...
100
101
```

##### 子迭代器

如果数据源本身就是可迭代对象，那么可使用yield from子迭代器语句。和在for循环中用yield并无不同，只是语法更简练。

```
>>> def test():
...     yield from "ab"
...     yield from range(3)
...
>>> for i in test(): print(i)
...
a
b
0
1
2
```

##### 生成器表达式

除使用小括号外，与推导式完全相同。

```
>>> x = (i+100 for i in range(8) if i % 2 ==0)
>>> x
<generator object <genexpr> at 0x00000265BEC7D138>
```

生成器表达式可被当做函数调用参数。

```
>>> def test(x):
...     print(x)
...     for i in x: print(i)
...
>>> test(i for i in range(3))
<generator object <genexpr> at 0x00000265BEC7D048>
0
1
2
```

##### 执行

相比普通迭代器，生成器的执行过程会复杂点。

首先，编译器会为生成器函数添加标记。对此类函数，解释器并不直接执行。而是将栈帧和代码作为参数，创建生成器实例。

```
>>> def test(n):
...     print("gen.start")
...     for i in range(n):
...             print(f"gen.yield {i}")
...             yield i
...             print("gen.resume")
...
>>> test.__code__.co_flags   # generator
99
>>> import inspect
>>> inspect.isgeneratorfunction(test)
True
>>> x = test(2)
>>> x.gi_frame.f_locals  #栈帧内存储的函数调用参数
{'n': 2}
>>> x.gi_code            # 关联用户函数
<code object test at 0x00000265BEC7E6F0, file "<stdin>", line 1>
>>> x.__next__           # 实现迭代器协议方法
<method-wrapper '__next__' of generator object at 0x00000265BEC7D048>
>>> next(x) # 第一次执行，执行到yield指令时，设置返回值，解释器保存线程状态
gen.start   # 并挂起当前函数流程，只有再次调用__next__方法，才恢复状态，继续
gen.yield 0 # 执行。如此，以yield切换分界线，往复交替，知道函数结束 。
0
>>> next(x)
gen.resume
gen.yield 1
1
>>> next(x)
gen.resume
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

##### 方法

生成器另一进化特征，就是提供双向通信能力。生成器不单是简单的数据提供方，还可作为接收方存在。生成器甚至能在外部停止迭代，或发送信号实现重置等自定义行为。

方法send除可向yield传送数据外，其他与next完全一致。不过发送前，生成器必须已经启动。只有这样，才可进入函数执行流程，才会对外提供数据和交互。

```
>>> def test():
...     while True:
...             v = yield 200
...             print(f"resume {v}")
...
>>> x = test()
>>> x.send(None)  # 必须用next或send(None)启动生成器
200
>>> x.send(100)   # 可发送任何数据，包括None,这与启动参数无关
resume 100
200
```

对生成器函数而言，挂起点是一个安全位置，相关状态被临时冻结。

调用close方法，解释器将终止生成器迭代。

```
>>> def test():
...     for i in range(10):
...             try:
...                     yield i
...             finally:
...                     print("finally")
...
>>> x = test()
>>> next(x)   # 启动生成器
0
>>> x.close() # 终止生成器
finally
>>> next(x)   # 已经终止
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

还可像send发送数据一样，向生成器throw指定异常作为信号。

```
>>> class ExitException(Exception): pass
>>> class ResetException(Exception): pass
>>> def test():
...     while True:
...             try:
...                     v = yield
...                     print(f"recv: {v}")
...             except ResetException:
...                     print("reset.")
...             except ExitException:
...                     print("exit.")
...                     return
...
>>> x = test()
>>> x.send(None)   # 启动生成器
>>> x.throw(ResetException) # 发送重置信号
reset.
>>> x.send(1)  # 可继续发送数据
recv: 1
>>> x.throw(ExitException)  # 发送终止信号
exit.
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>> x.send(2)  # 生成器已经终止
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

#### 模式

##### 生产消费模型

在不借助并发框架情况下，实现生产、消费协作。

> 消费者启动后，使用yield将执行权限交给生产者，等待其发送数据后激活处理。
>
> 如有多个消费者，或数据处理时间较长，依然建议使用专业并发方案

```
>>> def consumer():
...     while True:
...             v = yield
...             print(f"consume: {v}")
...
>>> def producer(c):
...     for i in range(10, 13):
...             c.send(i)
...
>>> c = consumer()   # 创建消费者
>>> c.send(None)     # 启动消费者
>>> producer(c)      # 生产者发送数据
consume: 10
consume: 11
consume: 12
>>> c.close()        # 关闭生产者  
```

##### 消除回调

回调函数是常见异步接口设计方式。调用者在发起请求后，不再阻塞等待结果返回，而改由异步服务调用预先注册的函数来完成后续处理。

设计一个简单异步服务示例，然后尝试用生成器消除回调函数。

```
>>> import threading
>>> import time

>>> def target(request, callback):
...     s = time.time()
...     request()       # 调用请求函数
...     time.sleep(2)   # 模拟阻塞
...     callback(f"done: {time.time() - s}")  # 调用回调函数，传入结果
...
>>> def service(request, callback):
...     threading.Thread(target= target, args=(request, callback)).start()
...
>>> def request():    # 任务请求模拟
...     print("start")
...
>>> def callback(x):  # 任务结束通知
...     print(x)

>>> service(request, callback)
start
done: 2.001331090927124
```

接下来，用生成器函数替代上面的2个函数，用yield分隔请求和返回代码，以便服务可接入其中，同时，修改服务框架，以生成器方式来调用任务。

```
>>> import threading
>>> import time
>>> def request():
...     print("start")
...     x = yield
...     print(x)
...
>>> def target(fn):
...     try:
...             s = time.time()
...             g = fn()        # 调用目标生成器函数
...             g.send(None)    # 启动，开始执行请求代码
...             time.sleep(2)
...             g.send(f"done: {time.time() - s}") # 阻塞结束后，结果返回任务
...     except StopIteration:   # 目标结束，拦截异常
...             pass
...
>>> def service(fn):
...     threading.Thread(target=target, args=(fn,)).start()
...
>>> service(request)
start
>>> print("do something")  # 发请求后，并未阻塞
do something
done: 2.001655340194702
```

##### 协程

协程以协作调度方式，在单个线程上切换执行并发任务。配合异步接口，可将I/O阻塞时间来执行更多任务。由于在用户空间实现，因此被称为用户线程。

```
>>> def sched(*tasks):
...     tasks = list(map(lambda t: t(), tasks)) #调用所有任务函数，生成器列表
...     while tasks:     #循环调用任务
...             try:
...                     t = tasks.pop(0)  # 从列表头部弹出任务
...                     t.send(None)      # 开始执行该任务
...                     tasks.append(t)   # 如该任务没有结束，则放回列表尾部
...             except StopIteration:     # 该任务结束，丢弃
...                     pass
>>> def task(id, n, m):                  # 模拟任务模板
...     for i in range(n, m):
...             print(f"{id}: {i}")
...             yield                    # 主动调度
...
>>> t1 = partial(task, 1, 10, 13)
>>> t2 = partial(task, 2, 30, 33)
>>> sched(t1, t2)
1: 10
2: 30
1: 11
2: 31
1: 12
2: 32
```

#### 函数式编程

函数式编程以计算表达式来替代命令式语句，强调逐级结果推导，而非执行过程。

函数式编程要求函数为第一类型，可作为参数和返回值传递。需要时，可用闭包构成带有上下文状态的逻辑返回。其通常不使用独立变量，所有状态以参数传递，用嵌套或链式调用代替过程语句。最好没有外部依赖的纯函数，且参数不可变，仅将结果带入下一次运算。

命令式：

```
x = a + b
y = x * 2
```

函数式:

```
mul(add(a, b), 3)
add(a, b).mul(2)
```

> 函数式编程三大特征：第一类型函数，不可变数据，尾递归优化。
>
> 函数式编程只是一种编程范式

##### 迭代

以迭代器方式，用指定函数处理数据源。相比推导式，它能从多个数据源中平行接收参数，直到最短数据源迭代完成。

```
>>> x = map(lambda a, b: (a, b), [1 ,2 ,3], "abcd")
>>> list(x)   # 将迭代器对象转换为列表
[(1, 'a'), (2, 'b'), (3, 'c')]
```

##### 聚合

平行从多个数据源接收参数，聚合成元组，直到最短数据源迭代结束。

```
>>> x = zip([1,2,3], "abcd", (1.1,1.2))
>>> list(x)
[(1, 'a', 1.1), (2, 'b', 1.2)]
```

如构造字典。都以最短数据源，如以最长数据源，则用itertools.zip_longest。

```
>>> kv = zip("abcd", range(100, 200))
>>> dict(kv)
{'a': 100, 'b': 101, 'c': 102, 'd': 103}
```

##### 累积

迭代数据源，将结果带入下次计算，适合完成统计或过滤操作。

```
>>> import functools
>>> def calc(ret, x):
...     print(f"ret = {ret}, x = {x}")
...     return ret + x
...
>>> functools.reduce(calc, [1, 2, 3]) #没有初始化值，直接将第一数据当结果
ret = 1, x = 2
ret = 3, x = 3
6
>>> functools.reduce(calc, [1, 2, 3], 100) # 要对第一数据和初始化值进行计算
ret = 100, x = 1
ret = 101, x = 2
ret = 103, x = 3
106
```

##### 过滤

使用指定函数对数据进行迭代过滤。

> 函数返回的布尔值决定数据的去留。参数为None时相当于bool函数，移除所有False元素。

```
>>> x = filter(lambda n: n % 2 ==0, range(10))
>>> list(x)
[0, 2, 4, 6, 8]
>>> list(filter(None, [0, 1, "", "a", [], (1,)]))
[1, 'a', (1,)]
```

##### 判断

判断在一系列数据中，全部或某个元素为真值。

```
>>> all([1, "a", (1,2)])
True
>>> any([0,"",(1,2)])
True
>>>
```

> 迭代会短路，比如all遇到第一个False，就立即终止。

```
>>> def a():
...     print('a')
...     return True
...
>>> def b():
...     print('b')
...     return False
...
>>> def c():
...     print('c')
...     return True
...
>>> all(map(lambda m: m(), (a, b, c)))
a
b
False
>>> any(map(lambda m: m(), (a, b, c)))
a
True
```

> 标准库itertools拥有大量针对迭代器的操作函数，推荐使用。