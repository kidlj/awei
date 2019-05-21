---
title: Metatable
---

    local Account = { 
      balance = 0 
    }

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