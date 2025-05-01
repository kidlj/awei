---
title: Lua
---

### Install

	$ brew install lua
	$ brew install lua@5.1

### OpenResty

	$ brew install openresty

### Luarocks

为 Lua 5.1 安装依赖并指定安装位置：

	$ luarocks --lua-dir=/usr/local/opt/lua@5.1 --tree=/usr/local/ install lua-resty-balancer

如此会将 .lua 库安装到 `/usr/local/share/lua/5.1/` 目录，将 .so 库安装到 `/usr/local/lib/lua/5.1/` 目录。

### 表

	local Account = {balance = 0}

	function Account.deposit(value)
	Account.balance = Account.balance + value
	end

	function Account.withdraw(value)
	if value > Account.balance then
		error("insufficient funds", 2)
	end 
	Account.balance = Account.balance - value
	end

	Account.deposit(100)
	Account.withdraw(10)
	print(Account.balance)
	-- Output:
	-- 90

	a = Account
	Account = nil 
	print(a.balance) -- 100
	print(a.deposit) -- function: 0x7f9eedc05440
	a.deposit(20) -- table.lua:4: attempt to index a nil value (upvalue 'Account')

### 方法

	local Account = {balance = 0}

	function Account.deposit(self, value)
	self.balance = self.balance + value
	end

	function Account:withdraw(value)
	if value > self.balance then
		error("insufficient funds", 2)
	end 
	self.balance = self.balance - value
	end

	Account.deposit(Account, 100)
	Account:withdraw(10)
	print(Account.balance)
	-- Output:
	-- 90

	a = {balance = 0, deposit = Account.deposit, withdraw = Account.withdraw}
	a.deposit(a, 100)
	a.withdraw(a, 20) 
	print(a.balance)
	-- Output:
	-- 80

### 元表

元表可以修改一个值在面对未知操作时的行为。例如，假设 a 和  b 都是表，那么可以通过元表定义 Lua 语言如何计算表达式 `a + b`。当 Lua 语言试图将两个表相加时，它会先检查两者之一是否有元表(metatable)且该元表中是否有 `__add` 字段。如果找到了该字段，就调用该字段的值，即所谓的元方法(metamethod)(是一个函数)，在本例中就是用于计算表的和的函数。

### __index 元方法

当访问一个表中不存在的字段时会得到 nil。这是正确的，但不是完整的真相。实际上，这些访问还会引发解释器查找元表中一个名为 `__index` 的元方法。如果没有这个元方法（在元表上定义），那么像一般情况下一样，结果即是 nil；否则，则由这个元方法来提供最终结果。

	prototype = {x = 0, y = 0, width = 100, height = 100}

	local mt = {}

	function new(o)
		o = o or {}
		setmetatable(o, mt) 
		return o
	end

	mt.__index = function(_, key)
		return prototype[key]
	end

	w = new({x = 10, y = 20})
	print(w.width)
	-- Output:
	-- 100

Lua 语言会发现 w 中没有对应的字段 width，但却有一个带有 __index 元方法的元表。因此，Lua 语言会以 `w`(表) 和 `width`(不存在的键)为参数来调用这个方法。元方法随后会用这个键检索原型并返回结果。

虽然叫做方法，但元方法 __index 不一定必须是一个函数，它还可以是一个表。当元方法是一个表时，Lua 语言就访问这个表。因此，在此前的示例中，可以把 __index 简单地声明为如下样式：

	mt.__index = prototype

将一个表用作 __index 元方法为实现继承提供了一种简单快捷的方法。虽然将函数用作元方法开销更昂贵，但函数却更灵活：我们可以通过函数实现多继承、缓存以及其它一些变体。

能不能略过元表 `mt` 而直接设置 `o.__index = prototype`，答案是不能。

没有 `setmetatable` 的配合，仅仅指定 `__index` 元方法并不起作用，更重要的是也不会屏蔽对表自身的成员访问：

	prototype = {x = 0, y = 0, width = 100, height = 100}

	function new(o)
		o = o or {}
		--setmetatable(o, mt)
		o.__index = prototype
		return o
	end

	w = new({x = 10, y = 20})
	print(w.x, w.y, w.width)
	-- Output:
	-- 10  20  nil

省略 `__index` 元方法的设置，仅仅 `setmetatable(o, prototype)` 也并不会自动访问 prototype 表的成员：

	prototype = {x = 0, y = 0, width = 100, height = 100}

	function new(o)
		o = o or {}
		setmetatable(o, prototype)
		--o.__index = prototype
		return o
	end

	w = new({x = 10, y = 20})
	print(w.x, w.y, w.width)
	-- Output:
	-- 10  20  nil

可见，`__index` 需要和 `setmetatable` 配合使用才会起作用。

### 类

	local mt = {__index = Account}

	function Account.new(o)
		o = o or {}
		setmetatable(o, mt) 
		return o
	end

	a2 = Account.new({balance = 0}) 
	a2:deposit(100) -- 相当于 getmetatable(a2).__index.deposit(a2, 100)
	a2:withdraw(30)
	print(a2.balance)
	-- Output:
	-- 70

### 继承

	local Account = {
		balance = 0
	}

	function Account:withdraw(value)
		if value > self.balance then
			error("insufficient funds", 2)
		end
		self.balance = self.balance - value
	end

	function Account:deposit(value)
		self.balance = self.balance + value
	end

-- 注意这里是为返回的新表 o 设置元表到 Account 表，而不是为 Account 表设置元表。
-- 更确切的表述是，为表 o 设置元表到 self，即 new 方法的 receiver 表，如此一来可取消依赖某个特定的元表，这是实现继承的最关键一点。

	function Account:new(o)
		o = o or {}
		self.__index = self
		setmetatable(o, self)
		return o
	end

	local SpecialAccount = Account:new()

	function SpecialAccount:getLimit()
		return self.limit or 0
	end

	function SpecialAccount:withdraw(value)
		if value > self.balance + self:getLimit() then -- not self.getLimit()
			error("insufficient funds", 2)
		end
		self.balance = self.balance - value
	end

	-- 继承的效果是，s 的元表是 SpecialAccount 表，
	-- 而 SpecialAccount 的元表是 Account 表。
	local s = SpecialAccount:new({limit = 100})

	-- override SpecialAccount:getLimit()
	function s:getLimit()
		return self.balance * 0.10
	end

	s:deposit(100)
	s:withdraw(110)
	print(s.balance)
	-- Output:
	-- -10

	a = Account:new()
	a:deposit(10)
	print(a.balance)
	-- Output:
	-- 10
	a:withdraw(100) -- ERROR: insufficient funds

注：以上代码的效果是 `s` 继承自 `SpecialAccount`，而 `SpecialAccount` 又继承自 `Account`。
