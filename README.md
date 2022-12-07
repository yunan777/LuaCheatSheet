# LUA 5.1

> <https://www.lua.org/pil/contents.html>

## 数据类型

- **nil**

```lua
print(nil) --> nil
print(type(nil)) --> nil
```

- **boolean**

```lua
print(true) --> true
print(false) --> false
print(type(true)) --> boolean
```

- **number**

```lua
print(1.2) --> 1.2
print(type(1.2)) --> number
```

- **string**

```lua
print("Hello, world!" .. "!!") --> Hello, world!!!
print(type("Hello, world!")) --> string
```

> 以上都是`value`類型，以下是`reference`類型

- **function**

```lua
-- 定义
local function example1(arg1, arg2)
    -- Function body
    return result1, result2
end

-- 传递引用
local example2 = example1

-- 调用
local var1, var2 = example2("value1", "value2")
local var3 = example2("value1", "value2")
local _, var4 = example2("value1", "value2")

-- 匿名函数写法
local example3 = function(arg1, arg2)
    -- Function body
    return result
end

-- 可变参数
local function example4(...)
    print(...)
    local t = {...}
    for value in t do
        print(value)
    end
end

-- 支持闭包
local function example5(value)
    return function(prefix)
        print(prefix .. value)
    end
end

-- 关键字参数使用单个表格参数来实现，注意调用的语法
local function add(arg)
    return arg.left + arg.right
end
local addResult = add{left = 2, right = 3}
print(addResult) --> 5
```

- **table**

```lua
local table1 = {
    x = "XXXX",
    n = 12,
}
print(table1.x) --> XXXX
print(table1["n"]) --> 12
print(table1.noexist) --> nil
print(type(table1.noexsist)) --> nil
print(noexist.field) --> error

local table2 = {"a", "b", "c", "d"}
print(table2[1]) --> a  索引从1开始
print(table2[5]) --> nil

local table3 = {1, 2, 3, 4}
print(#table3) --> 4 表的长度
table3.field = "string"
print(#table3) --> 4
table3[4] = nil
print(#table3) --> 3
table3[2] = nil
print(#table3) --> 1 删除中间的索引会出现不确定的结果，长度判断会有误，此处是BUG
print(table3[3]) --> 3

local symbol = {} --> 还可以当作symbol来用
print({} == {}) --> false
```

- **thread**

```lua
local thread = coroutine.create(function()
    print("Output first result")
    coroutine.yield("a")
    print("Output second result")
    coroutine.yield("b")
    print("Attempt to output third result")
    error("Third result error")
end)

print(coroutine.status(thread))

print(coroutine.resume(thread))
print(coroutine.resume(thread))
print(coroutine.resume(thread))

print(coroutine.status(thread))

--[[ 
    suspended
    Output first result
    true  a
    Output second result
    true  b
    Attempt to output third result
    false  Third result error
    dead
 ]]
```

> 创建thread时传入的函数中如果抛出错误，会在执行`coroutine.resume`时隐式抛出，类似`pcall`
>
> 注意，LUA并没有多线程，thread类型顶多算是异步，若阻塞了线程也会卡死

```lua
local producer = coroutine.create(function()
    local second_last, last = 0, 1
    while true do
        coroutine.yield(second_last)
        second_last, last = last, second_last + last
    end
end)

local function take(producer, num)
    return coroutine.create(function()
        for i = 1, num, 1 do
            local continue, value = coroutine.resume(producer)
            if continue then
                coroutine.yield(value)
            else
                break
            end
        end
        error("Exit")
    end)
end

local function recive(producer)
    while true do
        local continue, value = coroutine.resume(producer)
        if continue then
            print("Recieved: " .. value)
        else
            break
        end
    end
end

recive(take(producer, 12))
```

## 全局与局部变量

```lua
MYVAR = "myvalue" --> 直接赋值就是声明全局变量
_G.var1 = "value1" --> 放进_G全局变量里，其类型是table
local var2 = "value2" --> local 声明局部变量
```

## 比较操作符

> `>` `<` `>=` `<=` `==` `~=`

## 逻辑操作符

> `and` `or` `not`

```lua
print(4 and 5) --> 5
print(nil and 13) --> nil
print(false and 13) --> false
print(4 or 5) --> 4
print(false or 5) --> 5
print(not nil) --> true
print(not false) --> true
print(not 0) --> false
print(not not nil) --> false
```

## 流程控制

```lua
if a > 5 then
    -- do something
elseif a > 0 and a <= 5 then
    -- do something
else
    -- do something
end

while a < 10 do
    -- do something
    a = a + 1
end

repeat
    -- do something
    a = a + 1
until a = 10

for i = 1, 10, 1 do --> start, end, step(默认1)
    print(i)
    if i == 5 do
        break
    end
end --> 1 2 3 4 5  并且i默认为局部变量
```

## 迭代器和for...in...do

```lua
local function get_table_value_iterator(table)
    local i = 0
    return function()
        i = i + 1
        if i <= #table then
            return table[i]
        end
    end
end

local function get_table_key_value_iterator(table)
    local i = 0
    return function()
        i = i + 1
        if i <= #table then
            return i, table[i]
        end
    end
end

local function stateless_iterator(table, index)
    if index + 1 <= #table then
        return index + 1, table[index + 1]
    end
end

local function get_table_stateless_iterator(table)
    return stateless_iterator, table, 0
end

local t = {"a", "b", "c", "d", "e"}

for value in get_table_value_iterator(t) do
    print(value)
end

for key, value in get_table_key_value_iterator(t) do
    print(key .. " --> " .. value)
end

for key, value in get_table_stateless_iterator(t) do
    print(key .. " --> " .. value)
end
```

## 错误处理

```lua
local var1, var2, var3 = assert("success", 42, "message")
print(var1) --> success
print(var2) --> 42
print(var3) --> message
local var4, var5, var6 = assert(false, "error message", "ignored")
-- Error: error message
```

```lua
local function get_values(raise_error)
    if raise_error then
        error("Error message")
    end
    return "a", "b", "c"
end

local result, value1, value2, value3 = pcall(function() return get_values(true) end)
if result then
    print(value1) --> a
    print(value2) --> b
    print(value3) --> c
else
    print(value1) --> Error message
    print(value2) --> nil
    print(value3) --> nil
end
```

> 注意`pcall`当中并非直接调用了可能出错的代码，而是在定义的匿名函数中调用的，如果函数不需要传入参数的话，也可直接把函数传入pcall

## 面向对象

### MetaTable

通过给目标table设置它的元表，可以定义这个table的一些特殊行为，例如如何对不存在的索引进行响应，如何进行+-等操作

```lua
local myTable = {}
local myTableMeta = {}
setmetatable(myTable, myTableMeta)
```

### 元表的 __index(table, key) 方法

定义访问索引不存在时的行为

```lua
local table = {}
local metaTable = {}

print(table.meta); --> nil

setmetatable(table, metaTable)
metaTable.__index = function(table, index)
    return index .. index
end

print(table.meta); --> metameta

local prototype = {meta = "mmmmeta"}
metaTable.__index = prototype;

print(table.meta); --> mmmmeta
```

> 相对应的，`__newindex(table, key)` 定义了对不存在索引的赋值操作
> 将`__index` 与 `__newindex` 结合可实现proxy设计模式，或者进行只读限制

### 冒号: 语法糖

```lua
-- 调用时：
object:method(xxx);
-- 等价于
object.method(object, xxx);

-- 定义时：
function object:method(arg1)
    -- function body
    return 
end
-- 等价于
function object.method(self, arg1)
    -- function body
    return 
end
```

### 类 继承 多继承

```lua
local Persen = {
    Name = "",
    Age = 0,

    SayHello = function(self)
        print("Hi. My name is " .. self.Name .. ". My age is " .. self.Age .. ".")
    end,

    new = function(self, obj)
        obj = obj or {}
        setmetatable(obj, self)
        self.__index = self
        return obj
    end,
}

local Student = Persen:new()
Student.Score = 0
Student.SayHello = function(self)
    Persen.SayHello(self)
    print("And my score is " .. self.Score .. ".")
end

local jack = Persen:new{Name = "Jack", Age = 30}
local tom = Persen:new{Name = "Tom", Age = 32}
jack:SayHello() --> Hi. My name is Jack. My age is 30.
tom:SayHello() --> Hi. My name is Tom. My age is 32.

local bob = Student:new{Name = "Bob", Age = 17, Score = 98}
bob:SayHello()
--> Hi. My name is Bob. My age is 17.
--> And my score is 98.

local Animal = {
    Type = "",

    new = function(self, obj)
        obj = obj or {}
        setmetatable(obj, self)
        self.__index = self
        return obj
    end,
}

local function SearchFromClass(key, classes)
    for i = 1, #classes do
        local value = classes[i][key]
        if value then
            return value
        end
    end
    return nil
end

local function Inheritance(...)
    local parents = {...}
    local class = {}
    setmetatable(class, { __index = function(table, key) return SearchFromClass(key, parents) end })
    class.new = function(self, obj)
        obj = obj or {}
        setmetatable(obj, self)
        self.__index = self
        return obj
    end
    return class
end

local AnimalPersen = Inheritance(Persen, Animal)

local lily = AnimalPersen:new{Name = "Lily", Age = 17, Type = "Cat"}
print(lily.Type) --> Cat
lily:SayHello() --> Hi. My name is Lily. My age is 17.
```
