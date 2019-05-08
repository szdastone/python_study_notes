# 虚拟环境

什么是Python的虚拟环境呢？其实Python虚拟环境就是为项目创建的相对独立的开发环境，也就是可为每个项目安装各自独立使用的依赖模块。

解释了可能还是不明白，我最初也是不明白的，到底我只是学习python，搞那么多环境干啥呢？这里在举例解释下，比如你有2个python项目，同时都用到了某个第三方模块numpy，如果这2个项目使用同一个版本的numpy模块，那么不可能产生疑问。但如果使用了不同版本的numpy模块呢？由于python导入模块时并不能区分模块的版本，那么这2个使用不同版本numpy模块的项目就会出现问题了。

那怎么使用虚拟环境呢？在Python3中，默认安装了pyvenv。直接使用就可以了。

创建一个虚拟环境:

```
$python3 -m venv pyvenv2
```

其中pyvenv2是虚拟环境建立的路径，可根据需求自行定义，该命令会自动在当前目录下建立目录，并将python环境copy到这个目录，该目录结构为：

![1554967797901](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1554967797901.png)

要激活刚才创建的虚拟环境pyvenv2，那么只需要输入如下代码：

```
$source pyvenv2/bin/activate
```

成功后，则提示符前会加入环境名称，如下：

```
(pyvenv2) $
```

如果要退出虚拟环境，那么直接输入：

```
(pyvenv2) $deactivate
```

好了，验证下虚拟环境。在Python非虚拟环境下，输入命令：

```
$which python
/usr/bin/python
```

激活虚拟环境pyvenv2后，再次查看python路径：

```
$source pyvenv2/bin/activate
（pyvenv2）$which python
/home/dastone/pyvenv2/bin/python
```

如果我们查看下环境变量$PATH，会发现这个变量在激活前后是不同的。

我们也可在python中查看环境变量以及pip的site packages目录：

![1554970396705](https://szdastone-1258479409.cos.ap-hongkong.myqcloud.com/python_study_notes/1554967797901.png)

