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
    a.deposit(20) -- ERROR!

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

    function Account:withdraw(value)
      if self.balance - value < 0 then
        error("balance is not enough", 2)
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

    function SpecialAccount:withdraw(value)
      if self.balance - value + self.limit < 0 then
        error("balance is not enough", 2)
      end 
      self.balance = self.balance -value
    end

    local s = SpecialAccount:new({limit = 100})

    s:deposit(10)
    s:withdraw(100)
    print(s.balance)
    -- Output:
    -- -90

    a = Account:new()
    a:deposit(10)
    print(a.balance)
    a:withdraw(100)
    -- Output:
    -- balance is not enough