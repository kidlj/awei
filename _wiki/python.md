---
title: Python
---

### virtualenv

#### 创建

在项目目录内使用 virtualenv 创建一个 Python 执行环境，每次激活这个环境以后，那么我们所使用的 Python 解释器，以及第三方库都将是这个独立环境里的版本，此时使用 `pip`新安装进来的第三方库也将安装在这个独立环境之中。

首先，在项目目录里创建这个环境(同时指定 Python 版本)：

	$ cd ~/Python/MyProject
	$ virtualenv --python=/usr/bin/python3.3 env

这将会在该项目目录内创建一个`env`目录，里面是独立的 Python 执行环境。

#### 激活

那么当我们进入某个项目的时候，首先要做的就是激活该项目的独立环境。所谓激活实际就是执行一个 shell 脚本，该脚本通过更改 PATH 等环境变量让我们更优先接入安装于本项目环境目录(`env/`)之内的解释器和库等内容，而不使用安装在系统级别的解释器和库。

	$ source env/bin/active

如果要离开这个项目环境，简单地执行：

	$ deactivate

#### .gitignore

如果用 Git 管理 Python 项目的话，可以将所有项目的环境目录统一命名为`env`，然后在`.gitignore`文件里只要加上一行，Git 就不会追踪所有项目的环境目录了。

	# add this line to .gitignore
	env/

#### Python 3

Python 3.4 自带 virtualvenv，叫做 pyvenv，而且它会在虚拟环境里安装好 pip。


### Pip

常用命令：

	$ pip install SomePackage
	$ pip list
	$ pip list --outdated
	$ pip install --upgrade SomePackage
	$ pip uninstall SomePackage
	$ pip show --files SomePackage


### Module search path

当 import 一个模块的时候，解释器首先寻找 builtin-in 模块，如果没有找到，再查看 `sys.path` 里的一系列目录。`sys.path` 的初始化过程如下：

- 当前输入脚本的目录（如果没有输入，则缺省为当前工作目录）
- `PYTHONPATH` 环境变量里设定的一系列目录
- The installtion-depedent default

初始化以后，Python 程序可以改变 `sys.path`，当前执行脚本的目录被放置在搜寻路径的最开始，在标准库路径的前面。

(via Python 3 tutorial 6.1.2 The Module Search Path)


### Python 3 vs Python 2

总结 Python 2 和 Python 3 的区别。

#### 字典迭代

- Python 2:

	字典有两个方法用于迭代，一个是 `items()`，一个是 `iteritems()`，前者返回一个元组的列表，后者返回一个迭代器。

- Python 3:

	字典迭代只有 `items()` 方法可用。它为快速迭代作了优化。


### 任意长度参数

调用 `print` 函数时可以有不同数量的参数，我们也可以定义这样的函数：

	def func(positional, *args):

_定义_函数时，`*args` 应作为最后一个参数。这样调用时该函数就能接收任意数量的参数。同时，Python 会将该位置的所有参数打包为一个列表 `args`。可以在函数定义内访问和更改这个列表。综合举例见下文。

### 从序列中解包参数

跟上一技巧过程相反，在_调用_函数时，我们可以用 `*` 语法从列表或元组中解包参数来供函数使用：

	>>> args = [3, 6]
	>>> list(range(*args))
	[3, 4, 5]

	>>> matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
	>>> list(zip(*matrix))
	[(1, 4, 7), (2, 5, 8), (3, 6, 9)]

综合举例：

	def func1(x, y, z): # 固定接收三个参数
		print(x)
		print(y)
		print(z)

	def func2(*args):	# 定义函数，打包参数(可接收任意数量参数)
		args[0] = 'Hello'
		args[1] = 'awesome'
		func1(*args)	# 调用函数，解包参数(只能接收三个参数)

	func2('Goodby', 'cruel', 'world!')
	# will print
	Hello	# 参数已被更改
	awesome
	world!


### Lambda

可以用 `lambda` 关键字来定义一个简短的匿名函数，它的格式为：

	lambda parameters: expression

- `lambda` 可以用于所有需要一个函数对象的地方，比如嵌套函数：

		>>> def make_incrementor(n):
				return lambda x: x + n
		>>> f = make_incrementor(43)
		>>> f(0)
		42
		>>> f(1)
		43

	这等价于：

		def make_incrementor(n):
			def f(x):
				return x + n
			return f

- 用于列表的排序

		>>> pairs = [(1, "one"), (2, "two"), (3, "three"), (4, "four)]
		>>> sorted(pairs, key=lambda pair: pair[1])
		[(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]
		>>> pairs.sort(key=lambda pair: pair[1])
		>>> pairs
		[(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]


### 列表

- 分片赋值

		>>> a = ['a', 'b', 'c', 'd', 'e']
		>>> a[1: 3] = ['B', 'C']
		>>> a
		['a', 'B', 'C', 'd', 'e']

- 方法

	`list.append(x)` 等价于 `a[len(a) : ] = [x]`;  
	`list.extend(L)` 等价于 `a[len(a) : ] = L` 

- 用作栈或者队列

	列表很适合用作栈，直接使用 `append` 和 `pop` 方法即可。  
	列表不适合用作队列，因为将列表第一个元素移除需要将其它元素全部左移。


### 嵌套的列表推导式

列表推导式里的表达式可以是另一个列表推导式，是为嵌套列表推导式：

	>>> matrix = [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]]
	>>> [[row[i] for row in matrix] for i in range(4)]
	[[1, 5, 9], [2, 6, 10], [3, 7, 11], [4, 8, 12]]

注意和下边这个列表推导式区分：
	
	>>> [row[i] for row in matrix for i in range(4)]
	[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]


### 元组

元组打包：

	t = 1, 2, 3

初始化一个空元组：

	t = ()

初始化只有一个元素的元组：

	t = ('hello',)	# ``,`` 是必需的
	t = ('hello')	# 这样是不对的

如果一个元组包含可变对象，那么这个元组是非 hashable 的：

	>>> {(1, [2, 3]): 'test'}
	TypeError: unhashable type: 'list'


### 序列解包

任何序列都支持序列解包操作：

	>>> t = (1, 2, 3)
	>>> x, y, z = t

实际上，多重赋值就是一种元组打包和序列解包的结合：

	>>> x, y, z = 1, 2, 3
	

### 字典

- 构造器

	可以用 `dict()` 从元素为 "key, value" 结构的序列中方便地构建字典：

		>>> dict([[1, 'one'], [2, 'two']])
		{1: 'one', 2: 'two'}

		>>> dict(zip(['a', 'b'], [1, 2]))
		{'a': 1, 'b', 2}

	也可以用 keyword arguments 来构建字典：

		>>> dict(language="Python", platform="Linux")
		{'platform': 'Linux', 'language': 'Python'}


- 字典推导式

	跟列表推导式相似：

		>>> {x: x**2 for x in (2, 4, 6)}
		{2: 4, 4: 16, 6: 36}


### 迭代技巧

- 并行迭代：

		>>> truth = {'Python 2': 'the good', 'Python 3', 'the best'}
		>>> for k, v in truth.items():
		...		print(k, v)
		...
		Python 2 the good
		Python 3 the best

- 当迭代一个序列，可以用 `enumerate` 函数来一同获取位置索引：

		>>> for i, v in enumerate(['a', 'b', 'c']):
		...		print(i, v)
		...
		0 a
		1 b
		2 c

- 当同时迭代两个或多个序列，可以用 `zip()` 函数将序列元素配对：

		>>> questions = ['name', 'website', 'favorite color']
		>>> answers = ['me', 'example.com', 'cyan']'
		>>> for q, a in zip(questions, answers):
		...		print("What is your {0}? It is {1}".format(q, a))
		...
		What is your name? It is me
		What is your website? It is example.com
		What is your favorite color? It is cyan

- 反向迭代：

		>>> for i in reversed(range(3)):
		...		print(i)
		...
		2
		1
		0

- 先排序再迭代, `sorted()`会返回一个新列表：

		>>> letters = ['a', 'c', 'd', 'b', 'g', 'e']
		>>> for i in sorted(set(letters)):
		...		print(i)
		...
		['a', 'c', 'd', 'b', 'g', 'e']

- 如果迭代过程中要改变迭代对象，那么应该先拷贝一份迭代对象：

		>>> words = ['apple', 'banana', 'favoritefruit']
		>>> for word in words[:]:
		...		if len(word) > 6:
		...			words.insert(0, word)
		...
		>>> words
		['favoritefruit', 'apple', 'banana', 'favoritefruit']

	
### 字符串格式化

书上详细介绍了用 `format()` 函数来格式化字符串，这里记录字符串的 `format` 方法。


- 位置占位符

	基本用法：

		>>>	print("We are the {} who say {}!".format("Knights", "Ni"))
		We are the Knights who say Ni!
		
		>>> food = "burger"
		>>> adjective = "good"
		>>> print("The {0} is {1}".format(food, adjective))
		The burger is good

	被格式化的变量可以是高级数据结构：

		>>> table = {"food": "burger", "adjective": "good"}
		>>> print("The {0[food]} is {0[adjective]}".format(table)
		The burger is good

- keyword 占位符

	基本用法：

		>>> print("This {food} is {adjective}".format(
		...		food="burger", adjective="good"))
		This burger is good

	高级用法：

		>>> table = {"food": "burger", "adjective": "good"}
		>>> print("This {food} is {adjective}".format(**table))
		The burger is good

### 文件

用 `with` 来自动关闭文件。

	>>> with open('test.txt', 'r') as f:
	... 	read_data = f.read()


### 类变量与实例变量

实例变量用来指代与具体实例相关的数据，类变量用来指代与所有实例相关的数据。

	class Dog:
		kind = 'canine'

		def __init__(self, name):
			self.name = name

### 迭代器

我们可以用 `for` 来迭代列表、字符串、文件对象和字典等容器对象。本质上，`for` 语句调用 `iter()` 函数作用于容器对象上，然后返回一个定义了 `__next__` 方法的迭代器对象。每次调用该迭代器对象的 `__next__` 方法将返回容器对象的一个元素，直到元素用光，它将会抛出 `StopIteration` 异常。

	>>> s = 'abc'
	>>> it = iter(s)
	>>> it
	<str_iterator object at 0xb6f6e52c>
	>>> next(it)
	'a'
	>>> next(it)
	'b'

了解了这些，便可以将自己的类定义成可迭代的。首先定义 `__iter__` 方法来返回一个定义了 `__next__` 方法的对象。如果该类本身就定义了 `__next__`，可以直接返回 `self`。

	class Reverse:
		"""Iterator for looping over a sequence backwards"""
		def __init__(self, data):
			self.index = len(data)
			self.data = data

		def __iter__(self):
			return self

		def __next__(self):
			if self.index == 0:
				raise StopIteration

			self.index -= 1
			return self.data[self.index]


### 生成器


生成器用于快速简单地生成迭代器。它定义起来就像函数，不过在返回数据的地方不是用 `return` 而是用 `yield`。

	def reverse(data):
		for index in range(len(data) - 1, -1, -1):
			yield data[index]

	>>> for char in reverse('abcd'):
	...		print(char)
	...
	d
	c
	b
	a


### 生成器表达式

一些简单的生成器可以写成表达式的形式，它就像列表推导式，不过不同于前者被 `[]` 包围，它们直接被函数的括号包围。如果都适用的话，用生成器表达式很多时候比用列表推导式高效。

注意，这_不是_“元组生成器”，Python 中没有这个概念。

	>>> data = 'abcd'
	>>> list(data[i] for i in range(len(data) - 1, -1, -1))
	['d', 'c', 'b', 'a']

	>>> sum(i*i for in range(3))
	5

	>>> unique_words = set(word for line in file for word in line.split())

	>>> top = max((student.gpa, student.name) for student in graduates)


### 装饰器

装饰器是用函数来改造函数，比较常用的装饰器是 `@classmethod` 和 `@staticmethod`。`classmethod()` 和 `staticmethod()` 是两个内置函数，`@wrapper` 的形式仅仅是语法糖而已，以下两个定义等价：

	@staticmethod
	def f():
		pass

	def f():
		pass
	f = staticmethod(f)


- `staticmethod(function)` 函数

	接受一个函数作为参数，返回一个静态方法。

	定义静态方法的形式为：

		class C:
			@staticmethod
			def f(arg1, arg2, ...):
				...

	可以在类或者类的实例之上调用静态方法，形如 `C.f()` 或者 `C().f()`，它不像实例方法那样，不会隐含传入第一个参数。

	（静态方法跟实例无关，也跟类无关，就相当于外部函数。定义成方法是为了使相关代码的垂直距离更近，能够更好的组织和维护）


- `classmethod(function)` 函数

	接受一个函数作为参数，返回一个类方法。

	定义类方法的形式为：

		class C:
			@classmethod
			def f(cls, arg1, arg2, ...)
				...

	可以在类或者类的实例上调用类方法，形如 `C.f()` 或者 `C().f()`，跟实例方法将调用它的实例作为其第一个隐含参数一样，类方法将调用它的类（或者由实例中获得的类）隐含地作为它的第一个参数。如果是派生类在调用类方法，那么传入的隐含参数是这个派生类本身。
