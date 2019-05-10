# 循环

在列表、元组及字典中经常都用到了for循环，用来对集合中每个元素进行遍历。下面介绍while循环。

#### while循环

while循环就是不断运行，直到指定条件不满足为止。比如：

```
current =1
while current <=5:
	print(current)
	current +=1
```

在使用Input()函数进行输入时，希望能让用户不停输入，直到约到quit后退出。那么可使用while循环来实现。如：

```
message = ""
while message != 'quit':
	message = input("请输入，直到'quit'退出")
	if message !='quit':
		print(message)
```

#### break退出循环

在while循环中，可不管任何条件，都可使用break来退出循环。比如：

```
while True:
	message = input("请输入，直到'quit'退出")
	
	if message == 'quit':
		break
	else:
    	print(message)
```

当然，除了while循环，在for循环中，一样可使用break来退出循环。

#### continue继续循环

如要返回循环开头，并根据条件测试结果来决定是否继续执行循环，则可使用continue语句。如，从1到10 ，只打印奇数的循环：

```
num = 0
while num < 10:
	num +=1
	if num % 2 ==0:
		continue
	print(num)	
```

尽量避免无限循环，在交互的情况下，如无限循环，可使用Ctrl+C来退出，然后修改代码。

#### while处理列表和字典

for循环可遍历列表，但不可修改列表。如要移动或者修改或者删除列表时，那么最好用while循环。

```
src = ['a','b','c']
dst = []
while src:
	current = src.pop()
	dst.append(current)
for d in dst:
	print(d.title())
	
## 使用remove
pets = ['dog','cat','dog','fish','cat','rabbit']
while 'cat' in pets:
	pets.remove('cat')
	
```



