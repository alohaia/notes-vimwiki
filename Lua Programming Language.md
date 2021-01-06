# Lua Programming Language

## 命令行技巧

1. `-e` 在命令行测试简单代码

```bash
$ lua -e "print(math.sin(12))"
```

2. `-l` 加载库（见 3）

3. `-i` 命令行命令执行完后进入命令行交互模式

```bash
$ lua -i -lmylib -e "greeting('melody')"
Lua 5.4.1  Copyright (C) 1994-2020 Lua.org, PUC-Rio
Hello, melody! I' aloha.greeting('melody')
Hello, melody! I' aloha.
>
```

4. 命令行执行的命令会作为一个表 `arg` 传入执行的程序，脚本名作为索引为 `0` 的元素。

```bash
$ lua -e "sin=math=sin" script a b
```

```lua
arg[-3] = "lua"
arg[-2] = "-e"
arg[-1] = "sin=math.sin"
arg[0] = "script"
arg[1] = "a"
arg[2] = "b"
```

## 各种运算符

**运算符优先级**：

| 运算符                      | 说明       |
|-----------------------------|------------|
| `^`                         | 指数       |
| `-` `#` `~` `not`           | 一元运算符 |
| `+` `-`                     |            |
| `..`                        | 字符串连接 |
| `<<` `>>`                   | 位操作     |
| `&`                         | 位与       |
| `~`                         | 位异或     |
| `|`                         | 位或       |
| `<` `>` `<=` `>=` `~=` `==` |            |
| `and`                       |            |
| `or`                        |            |

### 关系运算符

`<` `>` `<=` `>=` `==` `~=`

`~=` 是“不等于”

### 逻辑运算符

`and` `or` `not`

**为变量指定默认初始值**

```lua
x = x or v
-- equinalent to
if not x then x = v end
```

**三元运算符**

```lua
-- a ? b : c
(a and b or c)
-- or
((a and b) or c)
```


## 数据类型

### 数字

#### 数字类型

从 Lua 5.3 开始，Lua 使用两种方式表示数字：
- *integer*: 64 位整数，`math.maxinteger`到`math.mininteger`
- *float*: 双精度浮点数

直到 Lua 5.2，所有数字都用是浮点数表示

> <font color='red'>双精度浮点数只能准确表示 2<sup>53</sup> 以下的数字。</font>

```lua
type(4)     --> number
type(4.0)   --> number
math.type(4)    --> integer
math.type(4.0)  --> float
```

#### 进制

Lua 通过以 *0x* 作为前缀来支持十六进制常量。<br>
另外，Lua 还支持**浮点十六进制常量**。

```lua
4
0.4
4.57e-3
0.3e12
5E+20       -- 5e+20

-- hexadecimal 十六进制数
0xff
0x1A3
0X0.2       -- 0.125
0x1p-1      -- 0.5
0xa.bp2     -- 42.75

-- 打印十六进制数
string.fotmat("%a", 419)    --> 0x1.a3p+8
string.format("%a", 0.1)    --> 0x1.999999999999ap-4
```

#### 整数除法和取模运算

Lua 支持整数除法和取模运算：

```lua
-- 整数除法
3   // 2     --> 1
3.0 // 2     --> 1.0
6   // 2     --> 3
6.0 // 2.0   --> 3.0
-9  // 2     --> -5
1.5 // 0.5   --> 3.0
-- 取模运算与整数除法
a % b == a - ((a // b) * b)     --> true
-- 取模运算可用于负数
-180 % 360  --> 180
180 % -360  --> 180
-180 % -360 --> 180
```

取模运算可以用来限制浮点数精度：

```lua
x = math.pi
x - x%0.01  --> 3.14
x - x%0.001 --> 3.141
```

#### 转化

##### 显式指定一个数字为浮点数：` + 0.0`

```lua
-3 + 0.0                    --> -3.0
0x7fffffffffffffff          --> 9223372036854775807
0x7fffffffffffffff + 0.0    --> 9.2233720368548e+18
```

> <font color='red'>不大于 2<sup>53</sup>（9007199254740992）的整数可以被精确地转换为浮点数</font>

```lua
9007199254740991 + 0.0 == 9007199254740991 --> true
9007199254740992 + 0.0 == 9007199254740992 --> true
9007199254740993 + 0.0 == 9007199254740993 --> **false**
```

##### 显式指定一个数字为整数

1. OR 0
```lua
2^53        --> 9.007199254741e+15     (float)
2^53 | 0    --> 9007199254740992       (integer)
```
要这样做，数字必须能被准确表示为 64 位整数。
```lua
3.2 | 0     -- fractional part
-- stdin:1: number has no integer representation
2^64 | 0    -- out of range
-- stdin:1: number has no integer representation
math.random(1, 3.5)
-- stdin:1: bad argument #2 to 'random
-- (number has no integer representation)
```

2. `math.tointeger`

```lua
math.tointeger(-258.0) --> -258
math.tointeger(2^30)   --> 1073741824
math.tointeger(5.01)   --> nil (not an integral value)
math.tointeger(2^64)   --> nil (out of range)
```

不可被转化时会得到`nil`，可以用来检验是否可以转化为整数。

```lua
function cond2int(x)
    return math.tointeger(x) or x
end
```

#### <font color='green'>舍入</font>

> 对于大的整数并，直接计算`x+0.5`的下限可能会得到预期外的结果。<br>

```lua
x = 2^51 + 1
print(string.format("%d %d", x, math.floor(x + 0.5)))
-- 2251799813685249 2251799813685249
```

这是因为 `2^51 + 1.5` 没有准确地表示为一个浮点数。可以这样避免：

<!-- :存疑: -->
```lua
-- 四舍五入
function round (x)
    local f = math.floor(x)
    if x == f then
        return f
    else
        return math.floor(x + 0.5)
    end
end
-- 无偏舍入（将半整数摄入到最接近的**偶数**）unbiased rounding
function round_unbiased (x)
    local f = math.floor(x)
    if (x == f) or (x % 2.0 == 0.5) then
        return f
    else
        return math.floor(x + 0.5)
    end
end
```

#### 数学库`math`

```lua
math.sin(math.pi / 2)   --> 1.0
math.max(10.4, 7,1 -3)  --> 10.4
math.huge               --> inf
```

```lua
math.ceil(12.4)         --> 13
math.ceil(-12.4)        --> -12
math.floor(15.9)        --> 15

math.modf(3.3)      --> 0.3
math.modf(-3.3)     --> -0.3
```

#### 伪随机数

```lua
-- 设置 seed
math.randomseed(os.time())
-- 有三种方法获取想要的随机数
random()        --> [0, 1)  float
random(n)       --> [1, n]  integer
random(l, u)    --> [l, u]  integer
```

### Table

## Exercises

- [Eight-Queen Puzzle](Eight-Queen_Puzzle.md)

## Lua extension

- [Use Lua with C++](Embedding_Lua_in_C++.md)

