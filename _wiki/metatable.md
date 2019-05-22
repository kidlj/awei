---
title: Metatable
---

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
    a.deposit(20) -- ERROR: attempt to index a nil value

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

    local Account = { 
      balance = 0 
    }

### 类

    local mt = {__index = Account}

    function Account.new(o)
      o = o or {}
      setmetatable(o, mt) 
      return o
    end

    a2 = Account.new({balance = 0}) 
    a2:deposit(100)
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
