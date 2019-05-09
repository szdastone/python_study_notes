# 字典

字典与列表和元组不同，存储的信息会更多，更准确。字典使用花括号({})来表示，使用一系列的键-值，可使用键来访问对应的值。比如:

```
product = {'color': 'red', 'price': 51.00}
print(product[color])
print(product[price])
```

#### 添加、修改、删除键值

字典是动态的，可随时添加键值以及修改、删除键值。如：

```
product = {'color': 'red', 'price': 51.00}
product['qty'] = 5
product['disc'] = -3
product[price] = 53
del product['disc']
```

#### 遍历字典

如列表、元组一样，可遍历字典的所有键值。

```
product = {'color': 'red', 'price': 51.00}
for key, value in product.items:
	print("Key:"+key)
	print("\nValue:"+value)
```

字典的所有key实际组成了一个列表，你可如列表一样进行遍历：

```
for key in product.keys():
	print("Key:"+key)
```

同样，所有的value也组成了列表，可通过product.values()访问。

既然字典的keys()、values()都是列表，那一样适用列表的属性，比如可使用sorted进行临时排序。

#### 嵌套

字典内的元素不一定就是固定一种类型，其值可以是字符串，数字，布尔值，元组，列表，甚至字典。也就是字典中可嵌套列表或字典，同样，列表也可嵌套字典、列表等。

```
product1 = {'color':'red','price':53}
product2 = {'color':'blue','price':44}
products = [product1,product2]
for product in products:
	print(product)

pizza = {
	'crust': 'thick',
	'toppints': ['mushrooms', 'extra cheese'],
}

users = {
	'john': {
		'first': 'jack',
		'last': 'zhao'
	},
	'rose': {
		'first': 'rose',
		'last': 'li'
	},
}
```

