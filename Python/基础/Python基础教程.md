# Python基础教程

> 本教程基于Python 3.15最新特性编写，全面覆盖Python编程基础知识

---

## 目录

1. [变量与数据类型](#1-变量与数据类型)
2. [运算符和表达式](#2-运算符和表达式)
3. [流程控制](#3-流程控制)
4. [基本数据结构](#4-基本数据结构)

---

## 1. 变量与数据类型

### 1.1 变量的概念

**变量**是用于存储数据值的容器。在Python中，变量不需要显式声明类型，Python会根据赋值自动推断变量类型（动态类型）。

#### 核心概念：

- **变量命名规则**：
  - 必须以字母或下划线开头
  - 只能包含字母、数字和下划线
  - 区分大小写
  - 不能使用Python保留字

- **动态类型**：变量的类型可以在运行时改变
- **变量引用**：Python中的变量是对象的引用，而不是值本身

### 1.2 变量的基本使用

#### 1.2.1 简单赋值

```python
# 基本变量赋值
name = "Alice"
age = 25
height = 1.68
is_student = True

# 查看变量类型
print(type(name))      # <class 'str'>
print(type(age))       # <class 'int'>
print(type(height))    # <class 'float'>
print(type(is_student)) # <class 'bool'>
```

#### 1.2.2 多重赋值

```python
# 同时给多个变量赋相同的值
x = y = z = 0

# 同时给多个变量赋不同的值（元组解包）
a, b, c = 1, 2, 3
print(f"a={a}, b={b}, c={c}")  # a=1, b=2, c=3

# 交换变量值（Python特色）
x, y = 10, 20
x, y = y, x  # 交换后：x=20, y=10
print(f"x={x}, y={y}")
```

#### 1.2.3 变量注解（类型提示）

从Python 3.6开始，支持变量类型注解，这有助于代码可读性和静态类型检查：

```python
# 使用类型注解声明变量
count: int = 0
name: str = "Bob"
price: float = 99.99
is_valid: bool = True

# 类型注解不会强制类型检查，仅用于提示
count = "这不会报错，但不符合类型提示"  # Python允许但不推荐

# 先声明类型，后赋值
value: int
value = 42
```

### 1.3 Python基本数据类型

#### 1.3.1 数值类型

##### **整数 (int)**

Python 3中的整数没有大小限制，可以表示任意大的整数。

```python
# 整数表示
decimal = 100          # 十进制
binary = 0b1010        # 二进制（前缀0b）
octal = 0o12           # 八进制（前缀0o）
hexadecimal = 0xFF     # 十六进制（前缀0x）

print(decimal, binary, octal, hexadecimal)  # 100 10 10 255

# 大整数
big_number = 123456789012345678901234567890
print(big_number ** 2)  # Python可以处理任意大的整数

# 使用下划线分隔数字（Python 3.6+）提高可读性
population = 1_234_567_890
print(population)  # 1234567890
```

##### **浮点数 (float)**

Python使用双精度浮点数（64位）。

```python
# 浮点数表示
pi = 3.14159
scientific = 1.5e-3    # 科学计数法：1.5 × 10^-3 = 0.0015

# 浮点数运算
print(0.1 + 0.2)       # 0.30000000000000004（浮点数精度问题）

# 使用decimal模块进行精确计算
from decimal import Decimal
precise_sum = Decimal('0.1') + Decimal('0.2')
print(precise_sum)     # 0.3
```

##### **复数 (complex)**

```python
# 复数表示（实部 + 虚部j）
z = 3 + 4j
print(z.real)      # 3.0 (实部)
print(z.imag)      # 4.0 (虚部)
print(abs(z))      # 5.0 (模)

# 复数运算
z1 = 1 + 2j
z2 = 3 + 4j
print(z1 + z2)     # (4+6j)
print(z1 * z2)     # (-5+10j)
```

#### 1.3.2 字符串类型 (str)

字符串是不可变的字符序列。

```python
# 字符串创建
single_quote = 'Hello'
double_quote = "World"
triple_quote = '''多行
字符串'''

# 字符串拼接
greeting = single_quote + " " + double_quote
print(greeting)  # Hello World

# 字符串索引和切片
text = "Python"
print(text[0])      # P (第一个字符)
print(text[-1])     # n (最后一个字符)
print(text[0:3])    # Pyt (切片：从索引0到2)
print(text[::-1])   # nohtyP (反转字符串)

# 字符串方法
print(text.upper())      # PYTHON
print(text.lower())      # python
print(text.replace('P', 'J'))  # Jython
print(text.split('t'))   # ['Py', 'hon']
```

##### **f-string 格式化（Python 3.6+）**

```python
name = "Fred"
age = 25

# f-string：最现代的字符串格式化方式
message = f"He said his name is {name!r}."
print(message)  # He said his name is 'Fred'.

# 在f-string中使用表达式
print(f"{name} is {age} years old")
print(f"Next year, {name} will be {age + 1}")

# 格式化数字
pi = 3.14159
print(f"Pi is approximately {pi:.2f}")  # Pi is approximately 3.14

# Python 3.8+ 可以显示变量名
print(f"{name=}, {age=}")  # name='Fred', age=25
```

#### 1.3.3 布尔类型 (bool)

布尔类型只有两个值：`True` 和 `False`。

```python
# 布尔值
is_active = True
is_deleted = False

# 布尔运算
print(True and False)   # False
print(True or False)    # True
print(not True)         # False

# 隐式布尔转换
# 以下值被视为False：
# False, None, 0, 0.0, '', [], {}, ()
if 0:
    print("不会执行")
    
if [1, 2]:
    print("非空列表为True")  # 会执行

# 显式布尔转换
print(bool(0))       # False
print(bool(1))       # True
print(bool(""))      # False
print(bool("Hello")) # True
```

#### 1.3.4 None类型

`None` 是Python的空值对象，表示"无值"或"空"。

```python
# None的使用
result = None

# 检查None
if result is None:
    print("result为空")

# 注意：使用is而不是==来检查None
# 正确的方式
if result is None:
    pass

# 不推荐的方式
if result == None:
    pass

# 函数默认返回None
def do_nothing():
    pass

print(do_nothing())  # None
```

### 1.4 类型转换

```python
# 数值类型转换
x = 10
y = 3.14
z = "25"

# 转换为整数
print(int(y))       # 3 (截断小数)
print(int(z))       # 25
print(int(True))    # 1

# 转换为浮点数
print(float(x))     # 10.0
print(float(z))     # 25.0

# 转换为字符串
print(str(x))       # "10"
print(str(y))       # "3.14"

# 转换为布尔值
print(bool(0))      # False
print(bool(42))     # True
print(bool(""))     # False
print(bool("text")) # True
```

### 1.5 变量的特殊用法

#### 1.5.1 下划线变量

```python
# 在交互式解释器中，_ 存储上一次表达式的结果
>>> 4 * 3.75 - 1
14.0
>>> tax = 12.5 / 100
>>> price = 100.50
>>> price * tax
12.5625
>>> price + _  # _ 是上一次的结果 12.5625
113.0625
>>> round(_, 2)
113.06

# 使用_作为临时变量或忽略的变量
for _ in range(5):
    print("重复5次")

# 在解包时忽略某些值
x, _, z = (1, 2, 3)  # 忽略中间的值
print(x, z)  # 1 3
```

#### 1.5.2 海象运算符（Python 3.8+）

`:=` 运算符允许在表达式中赋值：

```python
# 传统方式
data = input("请输入：")
if len(data) > 10:
    print(f"输入太长了：{len(data)}个字符")

# 使用海象运算符
if (n := len(input("请输入："))) > 10:
    print(f"输入太长了：{n}个字符")

# 在while循环中使用
import random
# 传统方式
while True:
    value = random.randint(1, 10)
    if value == 5:
        break
    print(value)

# 使用海象运算符
while (value := random.randint(1, 10)) != 5:
    print(value)

# 在列表推导式中使用
import re
text = "discount: 20% off, save 15% today"
discounts = [float(match.group(1)) / 100 
             for line in [text] 
             if (match := re.search(r'(\d+)%', line))]
print(discounts)  # [0.2]
```

### 1.6 变量作用域

```python
# 全局变量
global_var = "我是全局变量"

def demo_scope():
    # 局部变量
    local_var = "我是局部变量"
    print(global_var)  # 可以访问全局变量
    print(local_var)

demo_scope()
# print(local_var)  # 错误！外部无法访问局部变量

# 使用global关键字修改全局变量
counter = 0

def increment():
    global counter
    counter += 1

increment()
print(counter)  # 1

# nonlocal关键字（用于嵌套函数）
def outer():
    x = "outer"
    
    def inner():
        nonlocal x
        x = "inner"
    
    inner()
    print(x)  # inner

outer()
```

### 1.7 实践练习

```python
# 练习1：变量交换
a, b = 5, 10
print(f"交换前: a={a}, b={b}")
a, b = b, a
print(f"交换后: a={a}, b={b}")

# 练习2：类型转换计算
num_str = "123"
result = int(num_str) + 77
print(f"结果: {result}")  # 200

# 练习3：使用f-string格式化
name = "张三"
age = 28
height = 1.75
info = f"姓名: {name}, 年龄: {age}, 身高: {height:.2f}m"
print(info)

# 练习4：海象运算符应用
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
# 找出所有平方大于20的数
large_squares = [x for x in numbers if (square := x ** 2) > 20]
print(large_squares)  # [5, 6, 7, 8, 9, 10]

# 练习5：变量注解
def calculate_total(price: float, quantity: int) -> float:
    """计算总价"""
    total: float = price * quantity
    return total

print(calculate_total(19.99, 3))  # 59.97
```

---

## 2. 运算符和表达式

### 2.1 算术运算符

算术运算符用于执行数学运算。

#### 基本算术运算符

| 运算符 | 说明 | 示例 | 结果 |
|--------|------|------|------|
| `+` | 加法 | `5 + 3` | `8` |
| `-` | 减法 | `5 - 3` | `2` |
| `*` | 乘法 | `5 * 3` | `15` |
| `/` | 除法（浮点） | `5 / 2` | `2.5` |
| `//` | 整除 | `5 // 2` | `2` |
| `%` | 取模（余数） | `5 % 2` | `1` |
| `**` | 幂运算 | `2 ** 3` | `8` |

```python
# 基本算术运算
a = 10
b = 3

print(f"加法: {a} + {b} = {a + b}")      # 13
print(f"减法: {a} - {b} = {a - b}")      # 7
print(f"乘法: {a} * {b} = {a * b}")      # 30
print(f"除法: {a} / {b} = {a / b}")      # 3.333...
print(f"整除: {a} // {b} = {a // b}")    # 3
print(f"取模: {a} % {b} = {a % b}")      # 1
print(f"幂运算: {a} ** {b} = {a ** b}") # 1000

# 混合类型运算（整数与浮点数）
result = 4 * 3.75 - 1
print(result)  # 14.0（整数自动转换为浮点数）

# 负数运算
print(-5 // 2)   # -3（向下取整）
print(-5 % 2)    # 1

# 复杂表达式
x = 2 + 3 * 4    # 先乘除后加减
print(x)         # 14

y = (2 + 3) * 4  # 括号优先
print(y)         # 20
```

#### 运算符优先级

从高到低：
1. `**` （幂运算）
2. `+x`, `-x`, `~x` （正号、负号、按位取反）
3. `*`, `/`, `//`, `%` （乘、除、整除、取模）
4. `+`, `-` （加、减）

```python
# 优先级示例
result = 2 + 3 * 4 ** 2  # 4**2=16, 3*16=48, 2+48=50
print(result)  # 50

# 使用括号改变优先级
result = (2 + 3) * 4 ** 2  # (2+3)=5, 4**2=16, 5*16=80
print(result)  # 80
```

### 2.2 比较运算符

比较运算符用于比较两个值，返回布尔值。

| 运算符 | 说明 | 示例 | 结果 |
|--------|------|------|------|
| `==` | 等于 | `5 == 5` | `True` |
| `!=` | 不等于 | `5 != 3` | `True` |
| `>` | 大于 | `5 > 3` | `True` |
| `<` | 小于 | `5 < 3` | `False` |
| `>=` | 大于等于 | `5 >= 5` | `True` |
| `<=` | 小于等于 | `3 <= 5` | `True` |

```python
# 比较运算
x = 10
y = 5

print(x == y)   # False
print(x != y)   # True
print(x > y)    # True
print(x < y)    # False
print(x >= 10)  # True
print(y <= 5)   # True

# 链式比较（Python特色）
age = 25
print(18 <= age < 60)  # True（等价于 18 <= age and age < 60）

# 字符串比较（按字典序）
print("apple" < "banana")  # True
print("Python" == "python")  # False（区分大小写）

# 列表比较（逐元素比较）
print([1, 2, 3] < [1, 2, 4])  # True
print([1, 2] < [1, 2, 3])     # True
```

### 2.3 逻辑运算符

逻辑运算符用于组合布尔表达式。

| 运算符 | 说明 | 示例 | 结果 |
|--------|------|------|------|
| `and` | 逻辑与 | `True and False` | `False` |
| `or` | 逻辑或 | `True or False` | `True` |
| `not` | 逻辑非 | `not True` | `False` |

```python
# 逻辑运算符真值表
print(True and True)    # True
print(True and False)   # False
print(False and False)  # False

print(True or True)     # True
print(True or False)    # True
print(False or False)   # False

print(not True)         # False
print(not False)        # True

# 实际应用
age = 25
has_license = True

# 使用and
if age >= 18 and has_license:
    print("可以开车")

# 使用or
is_weekend = False
is_holiday = True
if is_weekend or is_holiday:
    print("可以休息")

# 使用not
is_raining = False
if not is_raining:
    print("可以出门")

# 短路求值
def func1():
    print("func1执行")
    return True

def func2():
    print("func2执行")
    return False

# and短路：第一个为False，不执行第二个
result = func2() and func1()  # 只输出"func2执行"

# or短路：第一个为True，不执行第二个
result = func1() or func2()   # 只输出"func1执行"
```

#### 逻辑运算的真值

```python
# Python中的"假"值
falsy_values = [False, None, 0, 0.0, '', [], {}, ()]
for value in falsy_values:
    if not value:
        print(f"{repr(value)} 为假")

# Python中的"真"值（非假值）
truthy_values = [True, 1, "text", [1], {"key": "value"}]
for value in truthy_values:
    if value:
        print(f"{repr(value)} 为真")
```

### 2.4 赋值运算符

赋值运算符用于给变量赋值。

#### 基本赋值

```python
x = 10  # 基本赋值
```

#### 复合赋值运算符

| 运算符 | 等价于 | 示例 |
|--------|--------|------|
| `+=` | `x = x + y` | `x += 5` |
| `-=` | `x = x - y` | `x -= 3` |
| `*=` | `x = x * y` | `x *= 2` |
| `/=` | `x = x / y` | `x /= 4` |
| `//=` | `x = x // y` | `x //= 2` |
| `%=` | `x = x % y` | `x %= 3` |
| `**=` | `x = x ** y` | `x **= 2` |

```python
# 复合赋值运算符
x = 10

x += 5   # x = x + 5
print(x)  # 15

x -= 3   # x = x - 3
print(x)  # 12

x *= 2   # x = x * 2
print(x)  # 24

x //= 5  # x = x // 5
print(x)  # 4

x **= 3  # x = x ** 3
print(x)  # 64

# 字符串的+=
text = "Hello"
text += " World"
print(text)  # Hello World

# 列表的+=
numbers = [1, 2, 3]
numbers += [4, 5]
print(numbers)  # [1, 2, 3, 4, 5]
```

### 2.5 位运算符

位运算符对整数的二进制位进行操作。

| 运算符 | 说明 | 示例 | 结果 |
|--------|------|------|------|
| `&` | 按位与 | `5 & 3` | `1` |
| `\|` | 按位或 | `5 \| 3` | `7` |
| `^` | 按位异或 | `5 ^ 3` | `6` |
| `~` | 按位取反 | `~5` | `-6` |
| `<<` | 左移 | `5 << 1` | `10` |
| `>>` | 右移 | `5 >> 1` | `2` |

```python
# 位运算示例
a = 60  # 0011 1100
b = 13  # 0000 1101

# 按位与
print(f"{a} & {b} = {a & b}")   # 12 (0000 1100)

# 按位或
print(f"{a} | {b} = {a | b}")   # 61 (0011 1101)

# 按位异或
print(f"{a} ^ {b} = {a ^ b}")   # 49 (0011 0001)

# 按位取反
print(f"~{a} = {~a}")           # -61

# 左移（相当于乘以2）
print(f"{a} << 2 = {a << 2}")   # 240 (左移2位)

# 右移（相当于除以2）
print(f"{a} >> 2 = {a >> 2}")   # 15 (右移2位)

# 实际应用：权限管理
READ = 1    # 001
WRITE = 2   # 010
EXECUTE = 4 # 100

# 设置权限
permissions = READ | WRITE  # 011 (可读可写)
print(f"权限: {permissions}")

# 检查权限
if permissions & READ:
    print("有读权限")

if permissions & EXECUTE:
    print("有执行权限")
else:
    print("没有执行权限")
```

### 2.6 成员运算符

成员运算符用于测试序列中是否包含指定的值。

| 运算符 | 说明 | 示例 |
|--------|------|------|
| `in` | 存在于序列中 | `'a' in 'abc'` |
| `not in` | 不存在于序列中 | `'d' not in 'abc'` |

```python
# 字符串成员测试
text = "Hello Python"
print('Python' in text)      # True
print('Java' not in text)    # True

# 列表成员测试
fruits = ['apple', 'banana', 'orange']
print('apple' in fruits)     # True
print('grape' in fruits)     # False

# 字典成员测试（检查键）
person = {'name': 'Alice', 'age': 25}
print('name' in person)      # True
print('address' in person)   # False

# 集合成员测试
numbers = {1, 2, 3, 4, 5}
print(3 in numbers)          # True
print(10 not in numbers)     # True

# 范围成员测试
print(5 in range(1, 10))     # True
print(15 in range(1, 10))    # False
```

### 2.7 身份运算符

身份运算符用于比较两个对象的内存地址。

| 运算符 | 说明 | 示例 |
|--------|------|------|
| `is` | 是同一个对象 | `x is y` |
| `is not` | 不是同一个对象 | `x is not y` |

```python
# is vs ==
a = [1, 2, 3]
b = [1, 2, 3]
c = a

# == 比较值
print(a == b)    # True（值相同）

# is 比较身份（内存地址）
print(a is b)    # False（不是同一个对象）
print(a is c)    # True（是同一个对象）

# None的比较应该使用is
x = None
print(x is None)     # 正确的方式
print(x == None)     # 可以工作但不推荐

# 小整数和字符串的缓存
x = 256
y = 256
print(x is y)    # True（Python缓存小整数）

x = 257
y = 257
print(x is y)    # False（超出缓存范围）

# 字符串驻留
s1 = "hello"
s2 = "hello"
print(s1 is s2)  # True（字符串驻留）

s3 = "hello world"
s4 = "hello world"
print(s3 is s4)  # 可能是True或False（取决于实现）
```

### 2.8 三元运算符（条件表达式）

Python的三元运算符允许在一行中编写简单的if-else语句。

```python
# 语法：value_if_true if condition else value_if_false

# 基本示例
age = 20
status = "成年人" if age >= 18 else "未成年人"
print(status)  # 成年人

# 比较与传统if-else
# 传统方式
if age >= 18:
    status = "成年人"
else:
    status = "未成年人"

# 三元运算符方式（更简洁）
status = "成年人" if age >= 18 else "未成年人"

# 嵌套三元运算符（不推荐，可读性差）
score = 85
grade = "A" if score >= 90 else "B" if score >= 80 else "C"
print(grade)  # B

# 实际应用
def get_discount(is_member, amount):
    return amount * 0.9 if is_member else amount

print(get_discount(True, 100))   # 90.0
print(get_discount(False, 100))  # 100
```

### 2.9 表达式实践

```python
# 练习1：计算圆的面积和周长
import math

radius = 5
area = math.pi * radius ** 2
circumference = 2 * math.pi * radius

print(f"半径: {radius}")
print(f"面积: {area:.2f}")
print(f"周长: {circumference:.2f}")

# 练习2：温度转换
celsius = 25
fahrenheit = celsius * 9/5 + 32
print(f"{celsius}°C = {fahrenheit}°F")

# 练习3：判断闰年
year = 2024
is_leap = (year % 4 == 0 and year % 100 != 0) or (year % 400 == 0)
print(f"{year}年是{'闰年' if is_leap else '平年'}")

# 练习4：位运算交换两个数
a, b = 10, 20
print(f"交换前: a={a}, b={b}")
a = a ^ b
b = a ^ b
a = a ^ b
print(f"交换后: a={a}, b={b}")

# 练习5：复合条件判断
username = "admin"
password = "123456"
is_active = True

login_success = (username == "admin" and 
                 password == "123456" and 
                 is_active)

print(f"登录{'成功' if login_success else '失败'}")

# 练习6：使用海象运算符简化代码
# 统计文本中长单词的数量
text = "Python is a powerful programming language"
long_words = [word for word in text.split() if (length := len(word)) > 5]
print(f"长单词: {long_words}")
```

---

## 3. 流程控制

流程控制语句用于控制程序的执行顺序，包括条件判断、循环和跳转等。

### 3.1 条件语句（if-elif-else）

#### 3.1.1 基本if语句

```python
# 基本if语句
age = 18
if age >= 18:
    print("你已成年")

# if-else语句
score = 85
if score >= 60:
    print("及格")
else:
    print("不及格")
```

#### 3.1.2 if-elif-else语句

```python
# 多条件判断
x = int(input("请输入一个整数: "))

if x < 0:
    x = 0
    print('负数已转换为零')
elif x == 0:
    print('零')
elif x == 1:
    print('单个')
else:
    print('多个')

# 成绩等级判断
score = 85

if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
elif score >= 60:
    grade = 'D'
else:
    grade = 'F'

print(f"成绩: {score}, 等级: {grade}")
```

#### 3.1.3 嵌套if语句

```python
# 嵌套条件判断
age = 25
has_license = True

if age >= 18:
    if has_license:
        print("可以开车")
    else:
        print("需要先考驾照")
else:
    print("年龄不够")

# 更好的写法：使用逻辑运算符
if age >= 18 and has_license:
    print("可以开车")
elif age >= 18:
    print("需要先考驾照")
else:
    print("年龄不够")
```

#### 3.1.4 条件表达式的高级用法

```python
# 使用in进行多值判断
day = "Saturday"
if day in ["Saturday", "Sunday"]:
    print("周末")
else:
    print("工作日")

# 使用not in
username = "admin"
forbidden_names = ["root", "administrator", "system"]
if username not in forbidden_names:
    print("用户名可用")
else:
    print("用户名被禁用")

# 检查空值
data = []
if not data:  # 空列表为False
    print("数据为空")

# 同时检查多个条件
x, y = 10, 20
if 0 < x < 100 and 0 < y < 100:
    print("x和y都在有效范围内")
```

### 3.2 循环语句

#### 3.2.1 for循环

`for`循环用于遍历序列（列表、元组、字符串等）或其他可迭代对象。

##### **基本for循环**

```python
# 遍历列表
fruits = ['apple', 'banana', 'orange']
for fruit in fruits:
    print(fruit)

# 遍历字符串
for char in "Python":
    print(char)

# 遍历字典
person = {'name': 'Alice', 'age': 25, 'city': 'Beijing'}

# 遍历键
for key in person:
    print(key)

# 遍历值
for value in person.values():
    print(value)

# 遍历键值对
for key, value in person.items():
    print(f"{key}: {value}")
```

##### **使用range()函数**

```python
# range(stop) - 从0到stop-1
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

# range(start, stop) - 从start到stop-1
for i in range(2, 6):
    print(i)  # 2, 3, 4, 5

# range(start, stop, step) - 指定步长
for i in range(0, 10, 2):
    print(i)  # 0, 2, 4, 6, 8

# 倒序
for i in range(10, 0, -1):
    print(i)  # 10, 9, 8, ..., 1

# 注意：i的修改不影响循环
for i in range(10):
    print(i)
    i = 5  # 这不会影响循环，因为i会被下一个索引覆盖
```

##### **enumerate()函数**

```python
# 同时获取索引和值
fruits = ['apple', 'banana', 'orange']
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# 指定起始索引
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}: {fruit}")
```

##### **zip()函数**

```python
# 同时遍历多个序列
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
cities = ['Beijing', 'Shanghai', 'Guangzhou']

for name, age, city in zip(names, ages, cities):
    print(f"{name}, {age}岁, 来自{city}")

# zip会在最短序列结束时停止
list1 = [1, 2, 3, 4, 5]
list2 = ['a', 'b', 'c']
for num, letter in zip(list1, list2):
    print(num, letter)  # 只输出3对
```

#### 3.2.2 while循环

`while`循环在条件为真时重复执行代码块。

```python
# 基本while循环
count = 0
while count < 5:
    print(f"计数: {count}")
    count += 1

# 使用海象运算符
import random
while (value := random.randint(1, 10)) != 5:
    print(f"随机数: {value}")
print("得到5，循环结束")

# 无限循环（需要break退出）
while True:
    user_input = input("输入'quit'退出: ")
    if user_input == 'quit':
        break
    print(f"你输入了: {user_input}")

# 计数器模式
n = 10
while n > 0:
    print(n)
    n -= 1
print("发射！")
```

#### 3.2.3 循环控制语句

##### **break语句**

`break`用于立即终止循环。

```python
# 在for循环中使用break
for i in range(10):
    if i == 5:
        break
    print(i)  # 输出0, 1, 2, 3, 4

# 在while循环中使用break
count = 0
while True:
    if count >= 5:
        break
    print(count)
    count += 1

# 查找第一个匹配项
numbers = [1, 3, 5, 7, 9, 2, 4, 6]
for num in numbers:
    if num % 2 == 0:
        print(f"找到第一个偶数: {num}")
        break
```

##### **continue语句**

`continue`跳过当前迭代，继续下一次循环。

```python
# 跳过偶数
for i in range(10):
    if i % 2 == 0:
        continue
    print(i)  # 只输出奇数: 1, 3, 5, 7, 9

# 跳过特定值
for i in range(1, 11):
    if i == 5:
        continue
    print(i)  # 输出1-10，但跳过5

# 实际应用：过滤数据
data = [1, -2, 3, -4, 5, -6, 7]
for num in data:
    if num < 0:
        continue
    print(num)  # 只输出正数
```

##### **else子句**

循环可以有`else`子句，当循环正常结束（没有被break中断）时执行。

```python
# for循环的else
for n in range(2, 10):
    for x in range(2, n):
        if n % x == 0:
            print(f"{n} = {x} * {n//x}")
            break
    else:
        # 循环正常结束，没有找到因子
        print(f"{n} 是质数")

# while循环的else
count = 0
while count < 5:
    print(count)
    count += 1
else:
    print("循环正常结束")

# break会跳过else
for i in range(5):
    if i == 3:
        break
else:
    print("这不会执行")  # 因为循环被break中断
```

### 3.3 模式匹配（match-case）Python 3.10+

`match`语句提供了强大的模式匹配功能，类似于其他语言的switch语句，但更强大。

#### 3.3.1 基本match语句

```python
# 基本模式匹配
def http_error(status):
    match status:
        case 400:
            return "Bad request"
        case 404:
            return "Not found"
        case 418:
            return "I'm a teapot"
        case _:
            return "Something's wrong with the internet"

print(http_error(404))  # Not found
print(http_error(500))  # Something's wrong with the internet
```

#### 3.3.2 组合模式

```python
# 使用|组合多个字面值
def http_error(status):
    match status:
        case 400:
            return "Bad request"
        case 401 | 403 | 404:
            return "Not allowed"
        case 418:
            return "I'm a teapot"
        case _:
            return "Something else"

print(http_error(401))  # Not allowed
```

#### 3.3.3 解构模式

```python
# 匹配序列
point = (0, 5)

match point:
    case (0, 0):
        print("原点")
    case (0, y):
        print(f"Y轴上的点: y={y}")
    case (x, 0):
        print(f"X轴上的点: x={x}")
    case (x, y):
        print(f"点坐标: ({x}, {y})")

# 匹配对象属性
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

def describe_point(point):
    match point:
        case Point(x=0, y=0):
            print("原点")
        case Point(x=0, y=y):
            print(f"Y轴: y={y}")
        case Point(x=x, y=0):
            print(f"X轴: x={x}")
        case Point(x=x, y=y):
            print(f"坐标: ({x}, {y})")

describe_point(Point(0, 5))  # Y轴: y=5
```

#### 3.3.4 守卫条件

```python
# 使用if添加额外条件
def categorize_number(num):
    match num:
        case n if n < 0:
            return "负数"
        case 0:
            return "零"
        case n if n > 0 and n < 10:
            return "个位正数"
        case n if n >= 10:
            return "两位及以上正数"

print(categorize_number(-5))   # 负数
print(categorize_number(5))    # 个位正数
print(categorize_number(15))   # 两位及以上正数
```

### 3.4 流程控制实践

```python
# 练习1：九九乘法表
print("九九乘法表:")
for i in range(1, 10):
    for j in range(1, i + 1):
        print(f"{j}×{i}={i*j}", end="\t")
    print()  # 换行

# 练习2：猜数字游戏
import random

secret = random.randint(1, 100)
attempts = 0
max_attempts = 7

print("猜数字游戏！我想了一个1-100之间的数字。")

while attempts < max_attempts:
    guess = int(input(f"第{attempts + 1}次猜测: "))
    attempts += 1
    
    if guess == secret:
        print(f"恭喜！你猜对了！用了{attempts}次。")
        break
    elif guess < secret:
        print("太小了！")
    else:
        print("太大了！")
else:
    print(f"游戏结束！正确答案是{secret}")

# 练习3：斐波那契数列
def fibonacci(n):
    """生成前n个斐波那契数"""
    a, b = 0, 1
    result = []
    for _ in range(n):
        result.append(a)
        a, b = b, a + b
    return result

print(fibonacci(10))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# 练习4：查找质数
def is_prime(n):
    """判断是否为质数"""
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

# 找出100以内的所有质数
primes = [n for n in range(2, 100) if is_prime(n)]
print(f"100以内的质数: {primes}")

# 练习5：使用match处理命令
def process_command(command):
    match command.split():
        case ["quit"]:
            return "退出程序"
        case ["load", filename]:
            return f"加载文件: {filename}"
        case ["save", filename]:
            return f"保存文件: {filename}"
        case ["delete", *files]:
            return f"删除文件: {', '.join(files)}"
        case _:
            return "未知命令"

print(process_command("load data.txt"))      # 加载文件: data.txt
print(process_command("delete a.txt b.txt")) # 删除文件: a.txt, b.txt

# 练习6：嵌套循环 - 打印图案
print("打印三角形:")
n = 5
for i in range(1, n + 1):
    print("*" * i)

print("\n打印金字塔:")
for i in range(1, n + 1):
    print(" " * (n - i) + "*" * (2 * i - 1))

# 练习7：列表推导式中的条件
# 找出1-50中能被3整除但不能被5整除的数
numbers = [x for x in range(1, 51) if x % 3 == 0 and x % 5 != 0]
print(f"符合条件的数: {numbers}")

# 练习8：使用else子句优化查找
def find_user(users, target_id):
    """在用户列表中查找指定ID"""
    for user in users:
        if user['id'] == target_id:
            print(f"找到用户: {user['name']}")
            break
    else:
        print(f"未找到ID为{target_id}的用户")

users = [
    {'id': 1, 'name': 'Alice'},
    {'id': 2, 'name': 'Bob'},
    {'id': 3, 'name': 'Charlie'}
]

find_user(users, 2)  # 找到用户: Bob
find_user(users, 5)  # 未找到ID为5的用户
```

---

## 4. 基本数据结构

Python提供了多种内置数据结构，用于存储和组织数据。

### 4.1 列表（List）

列表是Python中最常用的数据结构，是一个**可变的、有序的**元素集合。

#### 4.1.1 创建列表

```python
# 空列表
empty_list = []
empty_list2 = list()

# 包含元素的列表
numbers = [1, 2, 3, 4, 5]
fruits = ['apple', 'banana', 'orange']
mixed = [1, 'hello', 3.14, True, [1, 2, 3]]  # 可以包含不同类型

# 使用list()函数转换
from_string = list("Python")  # ['P', 'y', 't', 'h', 'o', 'n']
from_tuple = list((1, 2, 3))  # [1, 2, 3]
from_range = list(range(5))   # [0, 1, 2, 3, 4]
```

#### 4.1.2 访问列表元素

```python
fruits = ['apple', 'banana', 'orange', 'grape', 'mango']

# 正向索引（从0开始）
print(fruits[0])   # apple
print(fruits[2])   # orange

# 负向索引（从-1开始）
print(fruits[-1])  # mango（最后一个）
print(fruits[-2])  # grape（倒数第二个）

# 切片 [start:stop:step]
print(fruits[1:3])     # ['banana', 'orange']
print(fruits[:3])      # ['apple', 'banana', 'orange']（从开头到索引2）
print(fruits[2:])      # ['orange', 'grape', 'mango']（从索引2到结尾）
print(fruits[::2])     # ['apple', 'orange', 'mango']（步长为2）
print(fruits[::-1])    # 反转列表
```

#### 4.1.3 修改列表

```python
# 修改单个元素
fruits = ['apple', 'banana', 'orange']
fruits[1] = 'blueberry'
print(fruits)  # ['apple', 'blueberry', 'orange']

# 修改切片
numbers = [1, 2, 3, 4, 5]
numbers[1:3] = [20, 30]
print(numbers)  # [1, 20, 30, 4, 5]

# 添加元素
fruits.append('grape')           # 在末尾添加
fruits.insert(1, 'kiwi')         # 在指定位置插入
fruits.extend(['mango', 'pear']) # 扩展列表

# 删除元素
fruits.remove('apple')  # 删除第一个匹配的元素
popped = fruits.pop()   # 删除并返回最后一个元素
popped = fruits.pop(0)  # 删除并返回指定索引的元素
del fruits[1]           # 删除指定索引的元素
del fruits[1:3]         # 删除切片
fruits.clear()          # 清空列表
```

#### 4.1.4 列表方法

```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5]

# 排序
numbers.sort()              # 原地排序（升序）
numbers.sort(reverse=True)  # 降序排序
sorted_nums = sorted(numbers)  # 返回新的排序列表

# 反转
numbers.reverse()  # 原地反转

# 统计
count = numbers.count(1)  # 统计元素出现次数
index = numbers.index(5)  # 查找元素第一次出现的索引

# 复制
copy1 = numbers.copy()     # 浅拷贝
copy2 = numbers[:]         # 切片复制
copy3 = list(numbers)      # list()复制

# 列表长度
length = len(numbers)
```

#### 4.1.5 列表推导式

```python
# 基本列表推导式
squares = [x**2 for x in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 带条件的列表推导式
evens = [x for x in range(20) if x % 2 == 0]
print(evens)  # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# 多重循环
matrix = [[i*j for j in range(1, 4)] for i in range(1, 4)]
print(matrix)  # [[1, 2, 3], [2, 4, 6], [3, 6, 9]]

# 嵌套列表展平
nested = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for sublist in nested for num in sublist]
print(flattened)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# 条件表达式
numbers = [-4, -2, 0, 2, 4]
abs_values = [abs(x) if x < 0 else x for x in numbers]
print(abs_values)  # [4, 2, 0, 2, 4]
```

### 4.2 元组（Tuple）

元组是**不可变的、有序的**元素集合。

#### 4.2.1 创建元组

```python
# 空元组
empty_tuple = ()
empty_tuple2 = tuple()

# 包含元素的元组
numbers = (1, 2, 3, 4, 5)
fruits = ('apple', 'banana', 'orange')
mixed = (1, 'hello', 3.14, True)

# 单元素元组（注意逗号）
single = (42,)  # 正确
not_tuple = (42)  # 这是整数，不是元组

# 不使用括号（元组打包）
coordinates = 10, 20, 30
print(type(coordinates))  # <class 'tuple'>

# 使用tuple()函数转换
from_list = tuple([1, 2, 3])
from_string = tuple("abc")
```

#### 4.2.2 访问元组元素

```python
fruits = ('apple', 'banana', 'orange', 'grape')

# 索引和切片（与列表相同）
print(fruits[0])      # apple
print(fruits[-1])     # grape
print(fruits[1:3])    # ('banana', 'orange')
print(fruits[::-1])   # 反转

# 元组解包
x, y, z = (1, 2, 3)
print(x, y, z)  # 1 2 3

# 使用*收集剩余元素
first, *rest = (1, 2, 3, 4, 5)
print(first)  # 1
print(rest)   # [2, 3, 4, 5]

# 交换变量
a, b = 10, 20
a, b = b, a
print(a, b)  # 20 10
```

#### 4.2.3 元组方法

```python
numbers = (1, 2, 3, 2, 4, 2, 5)

# 统计
count = numbers.count(2)  # 3
index = numbers.index(3)  # 2

# 长度
length = len(numbers)  # 7

# 元组是不可变的
# numbers[0] = 10  # TypeError: 'tuple' object does not support item assignment

# 但元组可以包含可变对象
tuple_with_list = ([1, 2], [3, 4])
tuple_with_list[0].append(3)  # 可以修改列表内容
print(tuple_with_list)  # ([1, 2, 3], [3, 4])
```

#### 4.2.4 元组的应用

```python
# 函数返回多个值
def get_user_info():
    return "Alice", 25, "Beijing"

name, age, city = get_user_info()

# 作为字典的键（列表不可以）
locations = {
    (0, 0): "原点",
    (1, 0): "X轴",
    (0, 1): "Y轴"
}

# 命名元组（更具可读性）
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x, p.y)  # 10 20
print(p[0], p[1])  # 10 20

# 转换字典到命名元组
d = {'x': 11, 'y': 22}
p2 = Point(**d)
print(p2)  # Point(x=11, y=22)
```

### 4.3 字典（Dictionary）

字典是**可变的、无序的**键值对集合（Python 3.7+保持插入顺序）。

#### 4.3.1 创建字典

```python
# 空字典
empty_dict = {}
empty_dict2 = dict()

# 包含元素的字典
person = {
    'name': 'Alice',
    'age': 25,
    'city': 'Beijing'
}

# 使用dict()函数
person2 = dict(name='Bob', age=30, city='Shanghai')

# 从键值对列表创建
pairs = [('a', 1), ('b', 2), ('c', 3)]
dict_from_pairs = dict(pairs)

# 字典推导式
squares = {x: x**2 for x in range(6)}
print(squares)  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

#### 4.3.2 访问字典元素

```python
person = {'name': 'Alice', 'age': 25, 'city': 'Beijing'}

# 使用键访问
print(person['name'])  # Alice

# 使用get()方法（更安全）
print(person.get('name'))      # Alice
print(person.get('phone'))     # None（键不存在）
print(person.get('phone', 'N/A'))  # N/A（提供默认值）

# 检查键是否存在
if 'age' in person:
    print(f"年龄: {person['age']}")

# 获取所有键、值、键值对
keys = person.keys()      # dict_keys(['name', 'age', 'city'])
values = person.values()  # dict_values(['Alice', 25, 'Beijing'])
items = person.items()    # dict_items([('name', 'Alice'), ...])
```

#### 4.3.3 修改字典

```python
person = {'name': 'Alice', 'age': 25}

# 添加或修改元素
person['city'] = 'Beijing'  # 添加新键值对
person['age'] = 26          # 修改现有值

# update()方法
person.update({'phone': '123456', 'email': 'alice@example.com'})

# 删除元素
del person['phone']         # 删除指定键
age = person.pop('age')     # 删除并返回值
person.popitem()            # 删除并返回最后一个键值对（3.7+）
person.clear()              # 清空字典

# setdefault()方法
person = {'name': 'Alice'}
person.setdefault('age', 25)  # 如果键不存在，设置默认值
print(person)  # {'name': 'Alice', 'age': 25}
```

#### 4.3.4 字典方法和技巧

```python
# 复制字典
original = {'a': 1, 'b': 2}
copy1 = original.copy()  # 浅拷贝
copy2 = dict(original)

# 合并字典（Python 3.9+）
dict1 = {'a': 1, 'b': 2}
dict2 = {'c': 3, 'd': 4}
merged = dict1 | dict2
print(merged)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# 字典推导式
numbers = {'a': 1, 'b': 2, 'c': 3}
doubled = {k: v*2 for k, v in numbers.items()}
print(doubled)  # {'a': 2, 'b': 4, 'c': 6}

# 过滤字典
filtered = {k: v for k, v in numbers.items() if v > 1}
print(filtered)  # {'b': 2, 'c': 3}

# 嵌套字典
users = {
    'user1': {'name': 'Alice', 'age': 25},
    'user2': {'name': 'Bob', 'age': 30}
}
print(users['user1']['name'])  # Alice

# defaultdict（自动创建默认值）
from collections import defaultdict

word_count = defaultdict(int)
text = "hello world hello"
for word in text.split():
    word_count[word] += 1
print(dict(word_count))  # {'hello': 2, 'world': 1}
```

### 4.4 集合（Set）

集合是**可变的、无序的、不重复**元素集合。

#### 4.4.1 创建集合

```python
# 空集合（注意：{}创建的是空字典）
empty_set = set()

# 包含元素的集合
numbers = {1, 2, 3, 4, 5}
fruits = {'apple', 'banana', 'orange'}

# 从列表创建（自动去重）
numbers_list = [1, 2, 2, 3, 3, 3, 4]
unique_numbers = set(numbers_list)
print(unique_numbers)  # {1, 2, 3, 4}

# 从字符串创建
letters = set("abracadabra")
print(letters)  # {'a', 'r', 'b', 'c', 'd'}

# 集合推导式
squares = {x**2 for x in range(6)}
print(squares)  # {0, 1, 4, 9, 16, 25}
```

#### 4.4.2 集合操作

```python
a = {1, 2, 3, 4, 5}
b = {4, 5, 6, 7, 8}

# 并集
print(a | b)           # {1, 2, 3, 4, 5, 6, 7, 8}
print(a.union(b))      # 同上

# 交集
print(a & b)           # {4, 5}
print(a.intersection(b))  # 同上

# 差集
print(a - b)           # {1, 2, 3}
print(a.difference(b)) # 同上

# 对称差集（不在两者交集中的元素）
print(a ^ b)                      # {1, 2, 3, 6, 7, 8}
print(a.symmetric_difference(b))  # 同上

# 子集和超集
c = {1, 2}
print(c.issubset(a))    # True（c是a的子集）
print(a.issuperset(c))  # True（a是c的超集）

# 不相交
print(a.isdisjoint(b))  # False（有交集）
```

#### 4.4.3 修改集合

```python
fruits = {'apple', 'banana'}

# 添加元素
fruits.add('orange')
fruits.update(['grape', 'mango'])  # 添加多个元素

# 删除元素
fruits.remove('apple')    # 如果不存在会报错
fruits.discard('kiwi')    # 如果不存在不报错
popped = fruits.pop()     # 随机删除并返回一个元素
fruits.clear()            # 清空集合

# 集合运算并修改
a = {1, 2, 3}
b = {3, 4, 5}

a |= b   # a = a | b（并集）
a &= b   # a = a & b（交集）
a -= b   # a = a - b（差集）
a ^= b   # a = a ^ b（对称差集）
```

#### 4.4.4 frozenset（不可变集合）

```python
# 创建不可变集合
frozen = frozenset([1, 2, 3, 4])

# 可以作为字典的键或集合的元素
dict_with_frozen_key = {frozen: 'value'}
set_of_sets = {frozenset([1, 2]), frozenset([3, 4])}

# 不可变集合不能修改
# frozen.add(5)  # AttributeError
```

### 4.5 数据结构综合应用

```python
# 练习1：统计单词频率
text = "python is great python is powerful python is easy"
words = text.split()
word_freq = {}
for word in words:
    word_freq[word] = word_freq.get(word, 0) + 1
print(word_freq)

# 使用Counter（更简单）
from collections import Counter
word_freq = Counter(words)
print(word_freq.most_common(3))  # 最常见的3个单词

# 练习2：去重并保持顺序
def unique_ordered(items):
    seen = set()
    result = []
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

numbers = [1, 2, 2, 3, 1, 4, 3, 5]
print(unique_ordered(numbers))  # [1, 2, 3, 4, 5]

# 练习3：字典列表排序
students = [
    {'name': 'Alice', 'score': 85},
    {'name': 'Bob', 'score': 92},
    {'name': 'Charlie', 'score': 78}
]

# 按分数排序
sorted_students = sorted(students, key=lambda x: x['score'], reverse=True)
print(sorted_students)

# 练习4：嵌套数据结构
company = {
    'departments': {
        'IT': {
            'employees': ['Alice', 'Bob'],
            'budget': 100000
        },
        'HR': {
            'employees': ['Charlie', 'David'],
            'budget': 50000
        }
    }
}

# 访问嵌套数据
it_employees = company['departments']['IT']['employees']
print(it_employees)  # ['Alice', 'Bob']

# 练习5：集合应用 - 找出共同好友
alice_friends = {'Bob', 'Charlie', 'David', 'Eve'}
bob_friends = {'Alice', 'Charlie', 'Frank', 'Grace'}

common_friends = alice_friends & bob_friends
print(f"共同好友: {common_friends}")  # {'Charlie'}

# 练习6：列表、元组、集合转换
original_list = [1, 2, 2, 3, 3, 3, 4]
unique_tuple = tuple(set(original_list))
print(unique_tuple)  # (1, 2, 3, 4)

# 练习7：矩阵转置（使用zip）
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
transposed = list(zip(*matrix))
print(transposed)  # [(1, 4, 7), (2, 5, 8), (3, 6, 9)]

# 练习8：使用字典实现缓存
def fibonacci_cached(n, cache={}):
    """带缓存的斐波那契数列"""
    if n in cache:
        return cache[n]
    if n <= 1:
        return n
    cache[n] = fibonacci_cached(n-1, cache) + fibonacci_cached(n-2, cache)
    return cache[n]

print(fibonacci_cached(100))  # 快速计算大数

# 练习9：数据结构性能比较
import time

# 列表查找
large_list = list(range(100000))
start = time.time()
99999 in large_list
list_time = time.time() - start

# 集合查找
large_set = set(range(100000))
start = time.time()
99999 in large_set
set_time = time.time() - start

print(f"列表查找时间: {list_time:.6f}秒")
print(f"集合查找时间: {set_time:.6f}秒")
print(f"集合快 {list_time/set_time:.0f} 倍")
```

---

## 总结

本教程详细介绍了Python的基础知识：

1. **变量与数据类型**：掌握了变量的声明、类型注解、基本数据类型及类型转换
2. **运算符和表达式**：学习了算术、比较、逻辑、位运算等各种运算符
3. **流程控制**：掌握了if-elif-else条件语句、for/while循环、break/continue控制语句和match模式匹配
4. **基本数据结构**：深入理解了列表、元组、字典和集合的使用方法和应用场景

### 学习建议

- **多练习**：通过大量编码练习巩固知识
- **阅读文档**：养成查阅官方文档的习惯
- **写项目**：将所学知识应用到实际项目中
- **代码规范**：遵循PEP 8编码规范
- **持续学习**：Python生态丰富，保持学习热情

### 下一步学习方向

- 函数与模块
- 面向对象编程
- 文件操作与异常处理
- 常用标准库
- 第三方库（NumPy、Pandas等）
- Web开发框架（Django、Flask等）

---

**Happy Coding! 🐍**

