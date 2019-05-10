# 单元测试

编写完代码后，只有通过了测试，才可确定代码是否按要求完成指定的工作。

#### 函数测试

比如有段代码如下，保存在name.py中。

```
def get_name(first,last):
	full_name = first+’ ‘+last
	return full_name.title()
```

要进行测试比如导入模块unittest以及要测试的函数。在创建一个unittest.TestCase类，并编写一系列方法对函数进行不同测试。

```
import unittest
from name import get_name

class NamesTestCase(unittest.TestCase):
	def test_full_name(self):
		name = get_name('janis','joplin')
		self.assertEqual(name, 'Janis Joplin')
		
unittest.main()		
```

上面使用了unittest类常用功能之一：断言方法。断言方法用来核实得到的结果是否与预期的结果一致。如果相同，则测试通过，否则，则测试通不过。

如果测试未通过怎么办呢？需要检查条件是否错误，如果条件正确，而结果不对，则函数可能有问题，修改代码，然后继续测试，知道符合预期结果。

#### 测试类

unittest.TestCase类中提供了很多断言方法。断言方法检查应该满足的条件是否满足，如果满足，则程序正确，如果不满足，则程序可能代码有问题或者发生了异常。下面表格介绍断言方法：

| 方法                    | 用途               |
| ----------------------- | ------------------ |
| assertEqual(a, b)       | 核实 a == b        |
| assertNotEqual(a, b)    | 核实 a != b        |
| asserTrue(x)            | 核实x为True        |
| assertFalse(x)          | 核实x为False       |
| assertIn(Item, list)    | 核实item在list中   |
| assertNotIn(Item, list) | 核实item不在list中 |

除了断言方法，还有个方法setUp()，该方法只需在测试类中创建测试对象一次，然后在每个测试方法中就可使用它了。也就是setUp()方法可实例要初始的类以及初始化类的属性，以便在测试方法中能重复使用实例，而不需要重复创建。