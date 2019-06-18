# 进阶

### 1、解释器

#### 1.1、字节码

字节码被存储在\_\_code\_\_.co_code里。

```
>>> def add(x, y):
...     z = x + y
...     return z
>>> " ".join(f"{b:02X}" for b in add.__code__.co_code) #字节码为二进制数据
'7C 00 7C 01 17 00 7D 02 7C 02 53 00'
>>> dis.dis(add)
  2           0 LOAD_FAST                0 (x)
              2 LOAD_FAST                1 (y)
              4 BINARY_ADD
              6 STORE_FAST               2 (z)

  3           8 LOAD_FAST                2 (z)
             10 RETURN_VALUE
# 行号     偏移量 指令                     参数（目标对象）            
>>> add.__code__.co_firstlineno   # 起始行号。
1
>>> add.__code__.co_lnotab       # 两个一组，分别表示(字节码偏移位置，相对前一组行号的增量)
b'\x00\x01\x08\x01'
>>> add.__code__.co_varnames     # dis指令参数所值对象列表
('x', 'y', 'z')

>>> def test():                 # 代码中存在循环、跳转。可看dis标记了目标位置
...     for i in range(3): pass
...
>>> dis.dis(test) 
  2           0 SETUP_LOOP              16 (to 18)   # 参数为相对偏移量，绝对位置要加上自身长度2
              2 LOAD_GLOBAL              0 (range)
              4 LOAD_CONST               1 (3)
              6 CALL_FUNCTION            1
              8 GET_ITER
        >>   10 FOR_ITER                 4 (to 16)
             12 STORE_FAST               0 (i)
             14 JUMP_ABSOLUTE           10
        >>   16 POP_BLOCK
        >>   18 LOAD_CONST               0 (None)
             20 RETURN_VALUE             
```

> dis - Disassembler for Python bytecode
>
> ​	**LOAD_FAST(var_num)**
>
> ​    Pushes a reference to the local co_varnames(var_num) onto the stack。
>
> ​    **STORE_FAST(var_num)**
>
>    Stores TOS into the local co_varnames(var_num)

初步看来，字节码解读并不麻烦，这比真正的汇编指令要简单。这是因为，它基于栈式虚拟机设计，没有寄存器等概念。而且，指令转译比较直白，没有另类的优化方式。字节码并不能被CPU执行，实际每条字节码对应一大堆C实现的机器指令。但依然可用栈式虚拟机模型，模拟其执行过程。

> CPython解释器有两个栈：执行栈存储指令操作数；而块栈存储循环、异常等信息，以便在不同代码块间快速跳转。

#### 1.2、全局锁

全局锁(GIL)机制让解释器在同一时刻仅有一个线程可被调度执行。对单核环境，该实现简单、高效。其易实现对象安全访问，便于进行内存管理和编写扩展。但导致多核环境下无法实现并行，这对多线程为基础的并发应用就是一个“灾难”。常见解决方案是多线程加协程来充分发挥多核环境计算能力。

当GIL被其他线程占用时，等待线程会阻塞一段时间。如果超时后，依然无法获取锁，则发出请求。这种请求设计的很轻巧，就是一个全局条件变量设置。正在执行的线程会解释循环内会检查该标记，然后释放锁，切换线程执行，其自身进入等待状态。这是很典型的协作机制。

Cpython使用系统线程，且没有实现线程调度。所以，具体哪个等待线程被切换执行，有操作系统决定。甚至，发出请求和被切换执行的也未必是同一线程。当然，算法保证不会在一个执行周期内多次发送请求。

```
import sys
import threading


def countdown():
    n = 10000000
    while n > 0:
        n -= 1


if len(sys.argv) > 1:
    t1 = threading.Thread(target=countdown)
    t2 = threading.Thread(target=countdown)

    t1.start()
    t2.start()

    t1.join()
    t2.join()
else:
    countdown()
    countdown()

```

```
$ time python test.py  # 顺序执行
 
real	0m18.752s
user	0m4.121s
sys	0m0.011s
$ time python test.py 1  # 多线程执行

real	0m13.678s
user	0m4.218s
sys	0m0.022s
```

上面测试时基于CPU-bound来测试的，不同环境下，测试结果会有差异。虽然测试环境不太严谨，但依然能看到GIL在顺序执行和多线程执行之间的差距缩小，这表明在锁请求和线程切换处理上的改进效果还是很明显的。

下面就I/O-bound在进行测试。

```
import sys
import threading
import requests

def test():
    return requests.get("http://cn.bing.com").status_code

if len(sys.argv) > 1:
    ts = [threading.Thread(target=test) for i in range(10)]
    for t in ts: t.start()
    for t in ts: t.join()
else:
  [test() for i in range(10)] 
```

```
$ time python test.py    # 顺序执行

real    0m1.418s
user    0m0.301s
sys     0m0.062s
$ time python test.py 1  # 多线程执行

real    0m0.511s
user    0m0.301s
sys     0m0.079s
```

从测试结果来看，还是体现了多线程的优势。所以要在对的场合使用多线程，而不能应为GIL就认为多线程绝不可触及。

#### 1.3、执行过程

简单介绍下执行过程，由于要对执行过程的源码进行解读，故忽略源码，只简要记录解读过程。

##### 入口

执行过程根据命令行参数来选择执行模式。

首先初始化，然后选择执行模式，缺省为命令行模式：-c，可选择模块模式:-m，以及交互模式，交互模式需要提供入口文件；最后退出清理。

##### 初始化

除内置类型外，最重要是创建builtins、sys模块，并执行sys.modules、sys.path等运行所需的环境配置。

* 创建解释器和主线程状态实例

* 创建并初始化内置类型

* 初始化带有对象缓存的内置类型

* 创建sys.modules，存储运行期并导入的模块

* 初始化\_\_builtins\_\_模块

* 初始化内置异常类型

* 初始化sys模块

* 设置sys.path搜索路径列表(不包含site-packages)

* 设置sys.modules

* 初始化导入机制

* 创建\_\_main\_\_模块，注意，此时\_\_main\_\_模块和入口文件并没有关系

* 初始化site模块，添加site-specific路径到sys.path列表

主模块使用\_\_main\_\_这个特殊名称，但并未就是入口源文件。在交互和命令行模式下，它也作为默认作用域存在，此处的重点是建立名字空间，并导入内置成员。

主模块\_\_main\_\_：

* 创建模块(包括创建\_\_dict\_\_，设置\_\_name\_\_)
* 将\_\_builtins\_\_添加到名字空间。

##### 执行

在完成初始化后，转入到选定执行模式。

使用先前创建的\_\_main\_\_.\_\_dict\_\_作为入口源文件的模块名字空间。

* 获取\_\_main\_\_.\_\_dict\_\_，添加\_\_file\_\_信息
* 运行入口文件（检查pyc缓存），将\_\_main\_\_.\_\_dict\_\_作为名字空间传入

编译源文件，随后进入字节码执行流程。

创建执行所需的栈帧对象，并准备好参数、自由变量(闭包)等执行数据。

最终，字节码在一个规模接近3000行的超大函数内循环解释执行。

* 指令参数所需的相关名字列表
* 类似于SP、PC寄存器，下一条指令及栈帧顶位置
* 解释循环
  	* 检查并处理GIL
  	* 下一条指令和参数
  	* 指令执行
  	* 每条指令都有C实现具体过程

用户栈内存按地址从低向高分配，每次执行递增指令计数器。

##### 终止

在结束前，还要完成一系列清理操作。

* 等待前台线程结束
* 调用atexit注册的退出函数
* 垃圾回收，执行析构方法
* 释放导入的模块
* 执行相关结束函数
* 清理解释器和主线程状态
* 执行内置类型的结束函数

#### 1.4、内存分配

在执行源码解读中，有相关内存分配器的基本工作原理。

以某个阈值为界，分为大小对象。大对象直接分配内存，只有小对象才使用专用内存分配器。小对象按固定长度对齐后，在分成不同类别(size class)，以便与复用和管理。

首先，向系统申请大块Arena内存，按页大小将其分成多个Pool块，这是一级重用单元，每个Pool为一种类别的对象提供内存。Pool被回收后，可重新为另一类别的对象服务。

其次，按指定类别大小，将Pool再次切分成多个Block块。每个Block块可存储一个相同类别的小对象，这是二级重用单元。

简单来说，就是在通用内存分配基础上，以缓存方式实现小对象快速分配。

在深入细节前，先了解其API基本层次结构，这涉及不同类型的分配选择，也便于在源码中查找对应函数。

![](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1560826139647.png)

小对象按8字节对齐，分成64个不同类别。

底层PyMem API没啥值得关注内容。可从PyObject开始分析。

