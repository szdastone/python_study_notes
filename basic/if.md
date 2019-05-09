# 流程控制

### IF语句

If语句就是进行流程判断，然后做出相应的处理。比如：

```
cars = ['audi','bmw','subaru','toyota']
for car in cars:
	if car == 'bmw':
		print(car.upper())
	else:
    	print(car.title())
```

#### 条件测试

使用 == 来判断是否相等，如:

```
car = 'bmw'
car == 'bmw' # True
car = 'Audi'
car == 'audi' # False
car.lower() == 'audi' # True
```

使用!=来判断是否不相等

```
car = 'bmw'
car !='audi' #True
```

上面都是字符串比较，那么数字呢？除了 ==，!=外，还有小于（<），大于(>)，大于等于(>=)，小于等于(<=)。

如果是列表呢？就可以使用in了。如:

```
cars = ['audi','bmw','subaru','toyota']
'bmw' in cars # True
'apple' not in cars # True
```



#### 多条件判断

使用and 和or 来进行多条件判断。

```
age=10
age>=6 and age<12  # True
age>10 and age<=15 # False
age>=10 or age<15 True
```

#### if-else语句

当条件通过执行一个操作，没通过则执行另一个操作时，就要用到if-else语句了。

```
age =17
if age>=18:
	print('你可以选举了')
else:
	print('对不起，你还不能选举')
```

#### if-elif-else语句

多条件判断，当所有条件不满足，则执行else语句，比如：

```
age =12
if age>4:
	print('免费')
elif age<18:
	print('半价')
else:
	print('全价')
```

