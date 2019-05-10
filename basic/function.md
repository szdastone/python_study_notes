# 函数

函数就是带名字的代码块，用来完成一些具体的事情。要执行函数定义的特定任务时，就要调用该函数。定义函数是为了在程序中能多次调用，而无需反复编写代码。使用函数，会让程序编写、阅读、测试、DEBUG都变得容易。

#### 定义函数

下面定义一个简单函数，如：

```
def greet(user):
	print("Hello "+user.title())
	
greet('Python')	
```

上面函数传递了一个参数，那么在函数greet中，user就是参数，也可以称为形参。而调用greet('Python')中的’Python‘就是实参。greet('Python')就是将实参'Python'传递给函数greet中的形参user中。

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

#### 传递列表参数

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

#### 任意数量参数

有些函数定义时并不知道有多少实参，那么函数定义形参时加入*，表示可接收任意数量参数，比如：

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

有时，需要接收任意数量的实参，而且不知道会传递给函数什么参数。这时，可将函数编写成接收任意数量键值对，使用**来标识。比如:

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