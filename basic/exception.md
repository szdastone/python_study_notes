# 异常

在执行期出现的错误就是异常。当Python在处理发生的错误时会创建一个异常对象。如果编写了处理该异常的代码，则程序继续运行，如未对异常进行处理，则程序停止，并显示traceback，其中会包含异常的报告。

异常使用try-except代码块来处理。比如 

```
print(10/0)
```

Python会显示如下traceback：

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
```

但使用了try-except代码块处理异常后，则不同了，如下：

```
try:
	print(10/0)
except ZeroDivisionError:
	print("you can't divide by zero!")
```

这样就不会出现traceback了，会正常打印一条友好的错误消息。

如发生错误，而程序还没有完成，则妥善处理错误会很重要。如处理的好，则程序不会崩溃。

try-except-else代码块：

上面介绍了try-except代码块，可用来提高程序处理错误的能力。try代码块错误后会进入到except代码块，但执行成功后会进入到else代码块中。只有能引发异常的代码才需要放在try语句中。如try语句执行成功后需要运行代码应该放在else代码块中。except代码块是程序尝试运行try代码块引发指定异常后进行处理，如果想什么都不处理，将错误一声不吭的处理掉，可使用代码pass。

文件处理时很容易出现错误，比如文件找不大，这时就会出现FileNotFoundError异常，那么在处理文件时就需要处理FileNotFoundError异常。比如:

```
filename = 'a.txt'
try:
	with open(filename) as f：
		contents = f.read()
except FileNotFoundError:
	print('the file '+filename+' does not exist')
else:
	print(contents)
```





