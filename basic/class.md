# 类

面向对象编程是最流行的编写软件方法之一。在面向对象编程中，基于类来创建对象就是很通用的方法。根据类来创建对象被称为实例化。

#### 创建和使用类

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

#### 继承

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



