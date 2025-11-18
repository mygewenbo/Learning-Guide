# Python进阶教程

> 本教程基于Python 3.15最新特性编写，深入讲解函数、面向对象编程、模块、异常处理和文件操作

---

## 目录

1. [函数](#1-函数)
2. [面向对象编程](#2-面向对象编程)
3. [模块和包](#3-模块和包)
4. [异常处理](#4-异常处理)
5. [文件操作](#5-文件操作)

---

## 1. 函数

函数是组织好的、可重复使用的、用来实现单一或相关联功能的代码段。

### 1.1 函数的基本概念

**函数**是一段具有特定功能的、可重用的代码块。使用函数可以提高代码的模块化程度和重用性。

#### 核心概念：

- **函数定义**：使用`def`关键字定义函数
- **函数调用**：通过函数名加括号来调用函数
- **参数**：函数定义时的变量名（形参）
- **实参**：函数调用时传入的实际值
- **返回值**：函数执行后返回的结果

### 1.2 定义和调用函数

#### 1.2.1 基本函数定义

```python
# 最简单的函数
def greet():
    """打招呼函数"""
    print("Hello, World!")

# 调用函数
greet()  # 输出: Hello, World!

# 带参数的函数
def greet_person(name):
    """向指定人打招呼"""
    print(f"Hello, {name}!")

greet_person("Alice")  # 输出: Hello, Alice!

# 带返回值的函数
def add(a, b):
    """计算两个数的和"""
    return a + b

result = add(3, 5)
print(result)  # 输出: 8
```

#### 1.2.2 函数文档字符串（Docstring）

```python
def calculate_area(radius):
    """
    计算圆的面积
    
    参数:
        radius (float): 圆的半径
    
    返回:
        float: 圆的面积
    
    示例:
        >>> calculate_area(5)
        78.53981633974483
    """
    import math
    return math.pi * radius ** 2

# 访问文档字符串
print(calculate_area.__doc__)

# 使用help()查看文档
help(calculate_area)
```

### 1.3 函数参数

#### 1.3.1 位置参数

```python
def power(base, exponent):
    """计算幂运算"""
    return base ** exponent

# 按位置传递参数
print(power(2, 3))  # 8

# 参数顺序很重要
print(power(3, 2))  # 9
```

#### 1.3.2 关键字参数

```python
def describe_person(name, age, city):
    """描述一个人的信息"""
    print(f"{name}今年{age}岁，住在{city}")

# 使用关键字参数（顺序可以任意）
describe_person(age=25, city="北京", name="张三")

# 混合使用位置参数和关键字参数
describe_person("李四", city="上海", age=30)
# 注意：位置参数必须在关键字参数之前
```

#### 1.3.3 默认参数

```python
def greet(name, greeting="你好"):
    """带默认问候语的函数"""
    print(f"{greeting}, {name}!")

greet("Alice")              # 你好, Alice!
greet("Bob", "Hello")       # Hello, Bob!

# 默认参数的陷阱：可变对象作为默认值
def add_item(item, items=[]):  # 错误示范！
    items.append(item)
    return items

print(add_item(1))  # [1]
print(add_item(2))  # [1, 2] - 意外！

# 正确的做法
def add_item_correct(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

print(add_item_correct(1))  # [1]
print(add_item_correct(2))  # [2] - 正确！
```

#### 1.3.4 可变参数（*args）

```python
def sum_all(*numbers):
    """计算任意数量数字的和"""
    total = 0
    for num in numbers:
        total += num
    return total

print(sum_all(1, 2, 3))           # 6
print(sum_all(1, 2, 3, 4, 5))     # 15

# *args 将参数打包成元组
def print_args(*args):
    print(f"参数类型: {type(args)}")
    print(f"参数内容: {args}")

print_args(1, 2, 3, "hello")
# 参数类型: <class 'tuple'>
# 参数内容: (1, 2, 3, 'hello')
```

#### 1.3.5 关键字可变参数（**kwargs）

```python
def print_info(**kwargs):
    """打印任意数量的关键字参数"""
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=25, city="Beijing")
# name: Alice
# age: 25
# city: Beijing

# **kwargs 将参数打包成字典
def show_kwargs(**kwargs):
    print(f"参数类型: {type(kwargs)}")
    print(f"参数内容: {kwargs}")

show_kwargs(a=1, b=2, c=3)
# 参数类型: <class 'dict'>
# 参数内容: {'a': 1, 'b': 2, 'c': 3}
```

#### 1.3.6 混合使用参数

```python
def complex_function(pos1, pos2, *args, key1="default", **kwargs):
    """
    演示各种参数类型的组合
    
    pos1, pos2: 位置参数
    *args: 可变位置参数
    key1: 带默认值的关键字参数
    **kwargs: 可变关键字参数
    """
    print(f"位置参数: {pos1}, {pos2}")
    print(f"额外位置参数: {args}")
    print(f"关键字参数 key1: {key1}")
    print(f"额外关键字参数: {kwargs}")

complex_function(1, 2, 3, 4, key1="custom", extra1="a", extra2="b")
# 位置参数: 1, 2
# 额外位置参数: (3, 4)
# 关键字参数 key1: custom
# 额外关键字参数: {'extra1': 'a', 'extra2': 'b'}

# 参数顺序规则：
# 1. 位置参数
# 2. *args
# 3. 关键字参数（带默认值）
# 4. **kwargs
```

#### 1.3.7 仅限位置参数（Python 3.8+）

```python
def func(a, b, /, c, d):
    """
    a, b 只能通过位置传递
    c, d 可以通过位置或关键字传递
    / 之前的参数只能是位置参数
    """
    print(f"a={a}, b={b}, c={c}, d={d}")

func(1, 2, 3, 4)           # 正确
func(1, 2, c=3, d=4)       # 正确
# func(a=1, b=2, c=3, d=4) # 错误！a和b不能用关键字
```

#### 1.3.8 仅限关键字参数

```python
def func(a, b, *, c, d):
    """
    a, b 可以通过位置或关键字传递
    c, d 只能通过关键字传递
    * 之后的参数只能是关键字参数
    """
    print(f"a={a}, b={b}, c={c}, d={d}")

func(1, 2, c=3, d=4)       # 正确
# func(1, 2, 3, 4)         # 错误！c和d必须用关键字
```

### 1.4 函数返回值

#### 1.4.1 单个返回值

```python
def square(x):
    """返回平方值"""
    return x ** 2

result = square(5)
print(result)  # 25
```

#### 1.4.2 多个返回值

```python
def get_user_info():
    """返回多个值（实际返回元组）"""
    name = "Alice"
    age = 25
    city = "Beijing"
    return name, age, city

# 元组解包
name, age, city = get_user_info()
print(f"{name}, {age}, {city}")  # Alice, 25, Beijing

# 也可以作为元组接收
info = get_user_info()
print(info)  # ('Alice', 25, 'Beijing')
print(type(info))  # <class 'tuple'>
```

#### 1.4.3 没有返回值

```python
def print_message(msg):
    """没有return语句的函数返回None"""
    print(msg)

result = print_message("Hello")
print(result)  # None

# 显式返回None
def do_nothing():
    return None

# 或者只写return
def do_nothing2():
    return
```

### 1.5 函数类型注解（Type Hints）

```python
def greet(name: str) -> str:
    """
    带类型注解的函数
    
    参数:
        name: 字符串类型的名字
    
    返回:
        字符串类型的问候语
    """
    return f"Hello, {name}!"

# 复杂类型注解
from typing import List, Dict, Optional, Union, Tuple

def process_data(
    items: List[int],
    config: Dict[str, str],
    optional_param: Optional[str] = None
) -> Tuple[int, str]:
    """
    处理数据的函数
    
    参数:
        items: 整数列表
        config: 字符串到字符串的字典
        optional_param: 可选的字符串参数
    
    返回:
        包含整数和字符串的元组
    """
    total = sum(items)
    message = config.get('message', 'default')
    return total, message

# 查看函数注解
print(greet.__annotations__)
# {'name': <class 'str'>, 'return': <class 'str'>}

# 注意：类型注解不会强制类型检查，仅用于提示
result = greet(123)  # Python不会报错，但IDE会提示
```

### 1.6 Lambda函数（匿名函数）

```python
# 基本lambda函数
square = lambda x: x ** 2
print(square(5))  # 25

# 等价的def函数
def square_def(x):
    return x ** 2

# 多个参数的lambda
add = lambda x, y: x + y
print(add(3, 5))  # 8

# 在高阶函数中使用lambda
numbers = [1, 2, 3, 4, 5]

# 使用map
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# 使用filter
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4]

# 使用sorted的key参数
students = [
    {'name': 'Alice', 'score': 85},
    {'name': 'Bob', 'score': 92},
    {'name': 'Charlie', 'score': 78}
]
sorted_students = sorted(students, key=lambda s: s['score'], reverse=True)
print(sorted_students)

# lambda的局限性：只能包含单个表达式
# 不能包含语句（如print、赋值等）
# 复杂逻辑应该使用def定义函数
```

### 1.7 高阶函数

高阶函数是指接受函数作为参数或返回函数的函数。

#### 1.7.1 函数作为参数

```python
def apply_operation(x, y, operation):
    """应用指定的操作"""
    return operation(x, y)

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

print(apply_operation(5, 3, add))       # 8
print(apply_operation(5, 3, multiply))  # 15

# 使用lambda
print(apply_operation(5, 3, lambda a, b: a - b))  # 2
```

#### 1.7.2 函数作为返回值

```python
def make_multiplier(n):
    """返回一个乘法函数"""
    def multiplier(x):
        return x * n
    return multiplier

times_2 = make_multiplier(2)
times_3 = make_multiplier(3)

print(times_2(5))  # 10
print(times_3(5))  # 15
```

#### 1.7.3 内置高阶函数

```python
# map(): 对序列中的每个元素应用函数
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# filter(): 过滤序列
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4]

# reduce(): 累积计算（需要导入）
from functools import reduce
total = reduce(lambda x, y: x + y, numbers)
print(total)  # 15

# sorted(): 排序（key参数）
words = ['apple', 'pie', 'zoo', 'a']
sorted_words = sorted(words, key=len)
print(sorted_words)  # ['a', 'pie', 'zoo', 'apple']
```

### 1.8 闭包（Closure）

闭包是指在函数内部定义的函数，并且内部函数引用了外部函数的变量。

```python
def outer(x):
    """外部函数"""
    def inner(y):
        """内部函数（闭包）"""
        return x + y  # 引用外部函数的变量x
    return inner

add_5 = outer(5)
print(add_5(3))  # 8
print(add_5(10)) # 15

# 闭包的实际应用：计数器
def make_counter():
    count = 0
    def counter():
        nonlocal count  # 声明使用外部变量
        count += 1
        return count
    return counter

c1 = make_counter()
print(c1())  # 1
print(c1())  # 2
print(c1())  # 3

c2 = make_counter()
print(c2())  # 1（独立的计数器）

# 查看闭包变量
print(c1.__closure__)  # 包含闭包变量的cell对象
```

### 1.9 装饰器（Decorator）

装饰器是一种特殊的高阶函数，用于修改或增强函数的功能。

#### 1.9.1 基本装饰器

```python
def my_decorator(func):
    """简单的装饰器"""
    def wrapper():
        print("函数执行前")
        func()
        print("函数执行后")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# 输出:
# 函数执行前
# Hello!
# 函数执行后

# 等价于：
# say_hello = my_decorator(say_hello)
```

#### 1.9.2 带参数的装饰器

```python
def my_decorator(func):
    """处理带参数函数的装饰器"""
    def wrapper(*args, **kwargs):
        print(f"调用 {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} 返回: {result}")
        return result
    return wrapper

@my_decorator
def add(a, b):
    return a + b

result = add(3, 5)
# 输出:
# 调用 add
# add 返回: 8
```

#### 1.9.3 保留函数元数据

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # 保留原函数的元数据
    def wrapper(*args, **kwargs):
        """wrapper的文档"""
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """问候函数"""
    return f"Hello, {name}!"

print(greet.__name__)  # greet（而不是wrapper）
print(greet.__doc__)   # 问候函数（而不是wrapper的文档）
```

#### 1.9.4 带参数的装饰器

```python
def repeat(times):
    """重复执行函数的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
# 输出:
# Hello, Alice!
# Hello, Alice!
# Hello, Alice!
```

#### 1.9.5 实用装饰器示例

```python
import time
from functools import wraps

# 计时装饰器
def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} 执行时间: {end - start:.4f}秒")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "完成"

slow_function()

# 缓存装饰器
def memoize(func):
    cache = {}
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # 快速计算

# 类装饰器
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"调用次数: {self.count}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello!")

say_hello()  # 调用次数: 1
say_hello()  # 调用次数: 2
```

### 1.10 生成器函数

生成器函数使用`yield`语句返回值，可以暂停和恢复执行。

```python
def simple_generator():
    """简单的生成器"""
    yield 1
    yield 2
    yield 3

gen = simple_generator()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
# print(next(gen))  # StopIteration异常

# 在循环中使用生成器
for value in simple_generator():
    print(value)

# 生成器表达式
squares = (x**2 for x in range(10))
print(list(squares))  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 实用生成器：斐波那契数列
def fibonacci(n):
    """生成前n个斐波那契数"""
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

for num in fibonacci(10):
    print(num, end=' ')  # 0 1 1 2 3 5 8 13 21 34

# 无限生成器
def infinite_sequence():
    num = 0
    while True:
        yield num
        num += 1

# 读取大文件的生成器
def read_large_file(file_path):
    """逐行读取大文件"""
    with open(file_path, 'r') as file:
        for line in file:
            yield line.strip()
```

### 1.11 函数实践练习

```python
# 练习1：递归函数 - 阶乘
def factorial(n):
    """计算阶乘"""
    if n == 0 or n == 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))  # 120

# 练习2：递归 - 汉诺塔
def hanoi(n, source, target, auxiliary):
    """解决汉诺塔问题"""
    if n == 1:
        print(f"移动盘子 1 从 {source} 到 {target}")
        return
    hanoi(n-1, source, auxiliary, target)
    print(f"移动盘子 {n} 从 {source} 到 {target}")
    hanoi(n-1, auxiliary, target, source)

hanoi(3, 'A', 'C', 'B')

# 练习3：函数式编程 - 数据处理管道
def pipeline(*functions):
    """创建数据处理管道"""
    def process(data):
        result = data
        for func in functions:
            result = func(result)
        return result
    return process

# 定义处理函数
double = lambda x: x * 2
add_ten = lambda x: x + 10
square = lambda x: x ** 2

# 创建管道
process = pipeline(double, add_ten, square)
print(process(5))  # ((5*2)+10)^2 = 400

# 练习4：柯里化（Currying）
def curry(func):
    """将多参数函数转换为单参数函数链"""
    def curried(*args):
        if len(args) >= func.__code__.co_argcount:
            return func(*args)
        return lambda *more_args: curried(*(args + more_args))
    return curried

@curry
def add_three(a, b, c):
    return a + b + c

print(add_three(1)(2)(3))  # 6
print(add_three(1, 2)(3))  # 6
print(add_three(1, 2, 3))  # 6

# 练习5：参数验证装饰器
def validate_types(**type_checks):
    """验证函数参数类型的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # 获取函数签名
            import inspect
            sig = inspect.signature(func)
            bound_args = sig.bind(*args, **kwargs)
            
            # 验证类型
            for param_name, expected_type in type_checks.items():
                if param_name in bound_args.arguments:
                    value = bound_args.arguments[param_name]
                    if not isinstance(value, expected_type):
                        raise TypeError(
                            f"{param_name} 应该是 {expected_type.__name__}, "
                            f"但得到 {type(value).__name__}"
                        )
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_types(name=str, age=int)
def create_user(name, age):
    return f"用户: {name}, 年龄: {age}"

print(create_user("Alice", 25))  # 正常
# print(create_user("Bob", "30"))  # TypeError
```

---

## 2. 面向对象编程（OOP）

面向对象编程是一种程序设计范式，它将数据和操作数据的方法组织在一起，形成"对象"。

### 2.1 类和对象的基本概念

#### 核心概念：

- **类（Class）**：对象的蓝图或模板，定义了对象的属性和方法
- **对象（Object）**：类的实例，具有类定义的属性和方法
- **属性（Attribute）**：对象的数据
- **方法（Method）**：对象的行为（函数）
- **封装（Encapsulation）**：隐藏内部实现细节
- **继承（Inheritance）**：子类继承父类的属性和方法
- **多态（Polymorphism）**：不同对象对同一消息的不同响应

### 2.2 定义类和创建对象

#### 2.2.1 基本类定义

```python
# 定义一个简单的类
class Dog:
    """狗类"""
    
    # 类属性（所有实例共享）
    species = "Canis familiaris"
    
    # 初始化方法（构造函数）
    def __init__(self, name, age):
        """初始化狗对象"""
        # 实例属性
        self.name = name
        self.age = age
    
    # 实例方法
    def bark(self):
        """狗叫"""
        return f"{self.name} says Woof!"
    
    def get_info(self):
        """获取狗的信息"""
        return f"{self.name} is {self.age} years old"

# 创建对象（实例化）
dog1 = Dog("Buddy", 3)
dog2 = Dog("Lucy", 5)

# 访问属性
print(dog1.name)      # Buddy
print(dog1.age)       # 3
print(dog1.species)   # Canis familiaris

# 调用方法
print(dog1.bark())    # Buddy says Woof!
print(dog2.get_info()) # Lucy is 5 years old

# 修改属性
dog1.age = 4
print(dog1.age)       # 4
```

#### 2.2.2 self参数

```python
class Person:
    def __init__(self, name):
        self.name = name  # self指向当前实例
    
    def greet(self):
        # self用于访问实例属性和方法
        return f"Hello, I'm {self.name}"

person = Person("Alice")
print(person.greet())  # Hello, I'm Alice

# self是约定俗成的名称，可以用其他名字，但不推荐
class Example:
    def method(this):  # 不推荐
        return "This works but is confusing"
```

### 2.3 类属性和实例属性

```python
class Counter:
    # 类属性（所有实例共享）
    count = 0
    
    def __init__(self, name):
        # 实例属性（每个实例独有）
        self.name = name
        Counter.count += 1  # 通过类名访问类属性
    
    @classmethod
    def get_count(cls):
        """类方法，访问类属性"""
        return cls.count

# 创建实例
c1 = Counter("First")
c2 = Counter("Second")
c3 = Counter("Third")

# 访问类属性
print(Counter.count)      # 3
print(c1.count)           # 3（通过实例也可以访问）
print(Counter.get_count()) # 3

# 实例属性
print(c1.name)  # First
print(c2.name)  # Second

# 注意：通过实例修改类属性会创建同名实例属性
c1.count = 100
print(c1.count)      # 100（实例属性）
print(Counter.count) # 3（类属性未变）
print(c2.count)      # 3（其他实例不受影响）
```

### 2.4 方法类型

#### 2.4.1 实例方法

```python
class Calculator:
    def __init__(self, value=0):
        self.value = value
    
    # 实例方法（第一个参数是self）
    def add(self, n):
        """实例方法，操作实例数据"""
        self.value += n
        return self.value

calc = Calculator(10)
print(calc.add(5))  # 15
```

#### 2.4.2 类方法

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_string):
        """类方法，用于创建实例的替代构造函数"""
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)  # cls指向类本身
    
    @classmethod
    def today(cls):
        """类方法，返回今天的日期"""
        import datetime
        today = datetime.date.today()
        return cls(today.year, today.month, today.day)
    
    def __str__(self):
        return f"{self.year}-{self.month:02d}-{self.day:02d}"

# 使用普通构造函数
date1 = Date(2024, 1, 15)
print(date1)  # 2024-01-15

# 使用类方法作为替代构造函数
date2 = Date.from_string("2024-03-20")
print(date2)  # 2024-03-20

date3 = Date.today()
print(date3)
```

#### 2.4.3 静态方法

```python
class MathUtils:
    @staticmethod
    def is_even(n):
        """静态方法，不需要访问类或实例"""
        return n % 2 == 0
    
    @staticmethod
    def factorial(n):
        """计算阶乘"""
        if n == 0 or n == 1:
            return 1
        return n * MathUtils.factorial(n - 1)

# 通过类调用
print(MathUtils.is_even(4))   # True
print(MathUtils.factorial(5)) # 120

# 也可以通过实例调用（但不常见）
utils = MathUtils()
print(utils.is_even(3))  # False
```

### 2.5 特殊方法（魔术方法）

特殊方法以双下划线开头和结尾，用于实现Python的内置行为。

#### 2.5.1 基本特殊方法

```python
class Book:
    def __init__(self, title, author, pages):
        """构造函数"""
        self.title = title
        self.author = author
        self.pages = pages
    
    def __str__(self):
        """字符串表示（用户友好）"""
        return f"《{self.title}》 by {self.author}"
    
    def __repr__(self):
        """对象表示（开发者友好）"""
        return f"Book('{self.title}', '{self.author}', {self.pages})"
    
    def __len__(self):
        """返回长度"""
        return self.pages
    
    def __eq__(self, other):
        """相等比较"""
        if not isinstance(other, Book):
            return False
        return self.title == other.title and self.author == other.author

book = Book("Python编程", "张三", 500)

print(str(book))   # 《Python编程》 by 张三
print(repr(book))  # Book('Python编程', '张三', 500)
print(len(book))   # 500

book2 = Book("Python编程", "张三", 500)
print(book == book2)  # True
```

#### 2.5.2 运算符重载

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __add__(self, other):
        """加法运算符 +"""
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other):
        """减法运算符 -"""
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar):
        """乘法运算符 *（标量乘法）"""
        return Vector(self.x * scalar, self.y * scalar)
    
    def __abs__(self):
        """绝对值（向量长度）"""
        return (self.x ** 2 + self.y ** 2) ** 0.5
    
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(2, 3)
v2 = Vector(1, 4)

v3 = v1 + v2
print(v3)  # Vector(3, 7)

v4 = v1 - v2
print(v4)  # Vector(1, -1)

v5 = v1 * 3
print(v5)  # Vector(6, 9)

print(abs(v1))  # 3.605551275463989
```

#### 2.5.3 容器类型特殊方法

```python
class MyList:
    def __init__(self):
        self.items = []
    
    def __getitem__(self, index):
        """索引访问 obj[index]"""
        return self.items[index]
    
    def __setitem__(self, index, value):
        """索引赋值 obj[index] = value"""
        self.items[index] = value
    
    def __delitem__(self, index):
        """删除元素 del obj[index]"""
        del self.items[index]
    
    def __len__(self):
        """长度 len(obj)"""
        return len(self.items)
    
    def __contains__(self, item):
        """成员测试 item in obj"""
        return item in self.items
    
    def append(self, item):
        """添加元素"""
        self.items.append(item)
    
    def __iter__(self):
        """迭代器"""
        return iter(self.items)
    
    def __str__(self):
        return str(self.items)

my_list = MyList()
my_list.append(1)
my_list.append(2)
my_list.append(3)

print(my_list[0])     # 1
print(len(my_list))   # 3
print(2 in my_list)   # True

for item in my_list:
    print(item)       # 1, 2, 3
```

#### 2.5.4 上下文管理器

```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        """进入with语句时调用"""
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出with语句时调用"""
        if self.file:
            self.file.close()
        # 返回False表示不抑制异常
        return False

# 使用with语句
with FileManager('test.txt', 'w') as f:
    f.write('Hello, World!')
# 文件自动关闭
```

### 2.6 继承

继承允许创建新类（子类）来扩展现有类（父类）的功能。

#### 2.6.1 单继承

```python
# 父类（基类）
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "Some sound"
    
    def info(self):
        return f"I am {self.name}"

# 子类（派生类）
class Dog(Animal):
    def __init__(self, name, breed):
        # 调用父类的构造函数
        super().__init__(name)
        self.breed = breed
    
    # 重写父类方法
    def speak(self):
        return "Woof!"
    
    # 新增方法
    def fetch(self):
        return f"{self.name} is fetching the ball"

class Cat(Animal):
    def speak(self):
        return "Meow!"

# 使用子类
dog = Dog("Buddy", "Golden Retriever")
print(dog.name)     # Buddy（继承的属性）
print(dog.breed)    # Golden Retriever（新增的属性）
print(dog.speak())  # Woof!（重写的方法）
print(dog.info())   # I am Buddy（继承的方法）
print(dog.fetch())  # Buddy is fetching the ball（新增的方法）

cat = Cat("Whiskers")
print(cat.speak())  # Meow!

# isinstance()检查实例类型
print(isinstance(dog, Dog))     # True
print(isinstance(dog, Animal))  # True（子类实例也是父类实例）
print(isinstance(dog, Cat))     # False

# issubclass()检查类继承关系
print(issubclass(Dog, Animal))  # True
print(issubclass(Animal, Dog))  # False
```

#### 2.6.2 多继承

```python
class Flyable:
    def fly(self):
        return "Flying in the sky"

class Swimmable:
    def swim(self):
        return "Swimming in water"

class Duck(Animal, Flyable, Swimmable):
    def __init__(self, name):
        super().__init__(name)
    
    def speak(self):
        return "Quack!"

duck = Duck("Donald")
print(duck.speak())  # Quack!
print(duck.fly())    # Flying in the sky
print(duck.swim())   # Swimming in water

# 方法解析顺序（MRO）
print(Duck.__mro__)
# (<class '__main__.Duck'>, <class '__main__.Animal'>, 
#  <class '__main__.Flyable'>, <class '__main__.Swimmable'>, 
#  <class 'object'>)
```

#### 2.6.3 super()函数

```python
class Base:
    def __init__(self, value):
        self.value = value
        print(f"Base.__init__ called with {value}")
    
    def method(self):
        print("Base.method")

class Derived(Base):
    def __init__(self, value, extra):
        # 调用父类的__init__
        super().__init__(value)
        self.extra = extra
        print(f"Derived.__init__ called with {extra}")
    
    def method(self):
        # 调用父类的method
        super().method()
        print("Derived.method")

obj = Derived(10, 20)
# 输出:
# Base.__init__ called with 10
# Derived.__init__ called with 20

obj.method()
# 输出:
# Base.method
# Derived.method
```

### 2.7 封装和属性访问控制

Python使用命名约定来表示访问级别。

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner           # 公开属性
        self._balance = balance      # 受保护属性（约定）
        self.__pin = "1234"          # 私有属性（名称改写）
    
    # 公开方法
    def deposit(self, amount):
        if amount > 0:
            self._balance += amount
            return True
        return False
    
    def withdraw(self, amount, pin):
        if pin == self.__pin and amount <= self._balance:
            self._balance -= amount
            return True
        return False
    
    # 受保护方法
    def _validate_amount(self, amount):
        return amount > 0
    
    # 私有方法
    def __check_pin(self, pin):
        return pin == self.__pin
    
    def get_balance(self):
        """获取余额"""
        return self._balance

account = BankAccount("Alice", 1000)

# 公开属性和方法
print(account.owner)  # Alice
account.deposit(500)
print(account.get_balance())  # 1500

# 受保护属性（可以访问，但不推荐）
print(account._balance)  # 1500

# 私有属性（名称改写为_ClassName__attribute）
# print(account.__pin)  # AttributeError
print(account._BankAccount__pin)  # 1234（不推荐这样访问）
```

### 2.8 属性装饰器（@property）

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
    
    @property
    def celsius(self):
        """获取摄氏温度"""
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        """设置摄氏温度"""
        if value < -273.15:
            raise ValueError("温度不能低于绝对零度")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        """获取华氏温度"""
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        """设置华氏温度"""
        self.celsius = (value - 32) * 5/9
    
    @property
    def kelvin(self):
        """获取开尔文温度（只读）"""
        return self._celsius + 273.15

temp = Temperature(25)

# 像访问属性一样使用
print(temp.celsius)     # 25
print(temp.fahrenheit)  # 77.0
print(temp.kelvin)      # 298.15

# 设置值
temp.celsius = 30
print(temp.fahrenheit)  # 86.0

temp.fahrenheit = 100
print(temp.celsius)     # 37.77777777777778

# temp.kelvin = 300  # AttributeError（只读属性）
```

### 2.9 抽象基类

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    """抽象基类"""
    
    @abstractmethod
    def area(self):
        """抽象方法：计算面积"""
        pass
    
    @abstractmethod
    def perimeter(self):
        """抽象方法：计算周长"""
        pass
    
    def description(self):
        """具体方法"""
        return f"这是一个形状，面积为{self.area()}"

# shape = Shape()  # TypeError: 不能实例化抽象类

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        import math
        return math.pi * self.radius ** 2
    
    def perimeter(self):
        import math
        return 2 * math.pi * self.radius

rect = Rectangle(5, 3)
print(rect.area())        # 15
print(rect.perimeter())   # 16
print(rect.description()) # 这是一个形状，面积为15

circle = Circle(5)
print(circle.area())      # 78.53981633974483
```

### 2.10 数据类（dataclass）Python 3.7+

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Person:
    """使用dataclass装饰器自动生成特殊方法"""
    name: str
    age: int
    email: str = ""  # 默认值
    
    def greet(self):
        return f"Hello, I'm {self.name}"

# 自动生成__init__, __repr__, __eq__等方法
person1 = Person("Alice", 25, "alice@example.com")
person2 = Person("Alice", 25, "alice@example.com")

print(person1)  # Person(name='Alice', age=25, email='alice@example.com')
print(person1 == person2)  # True

@dataclass
class Student:
    name: str
    grades: List[int] = field(default_factory=list)  # 可变默认值
    
    def average_grade(self):
        return sum(self.grades) / len(self.grades) if self.grades else 0

student = Student("Bob")
student.grades.append(90)
student.grades.append(85)
print(student.average_grade())  # 87.5

# 继承dataclass
@dataclass
class GraduateStudent(Student):
    thesis_topic: str = ""
    advisor: str = ""

grad = GraduateStudent("Charlie", [95, 92], "AI Research", "Dr. Smith")
print(grad)
```

### 2.11 面向对象实践

```python
# 练习1：设计一个图书管理系统
class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.is_borrowed = False
    
    def __str__(self):
        status = "已借出" if self.is_borrowed else "可借阅"
        return f"《{self.title}》 - {self.author} [{status}]"

class Library:
    def __init__(self, name):
        self.name = name
        self.books = []
    
    def add_book(self, book):
        self.books.append(book)
        print(f"已添加: {book.title}")
    
    def borrow_book(self, isbn):
        for book in self.books:
            if book.isbn == isbn:
                if not book.is_borrowed:
                    book.is_borrowed = True
                    return f"成功借阅: {book.title}"
                else:
                    return f"{book.title} 已被借出"
        return "未找到该书"
    
    def return_book(self, isbn):
        for book in self.books:
            if book.isbn == isbn:
                if book.is_borrowed:
                    book.is_borrowed = False
                    return f"成功归还: {book.title}"
                else:
                    return f"{book.title} 未被借出"
        return "未找到该书"
    
    def list_available_books(self):
        available = [book for book in self.books if not book.is_borrowed]
        return available

# 使用图书管理系统
library = Library("市图书馆")

book1 = Book("Python编程", "张三", "ISBN001")
book2 = Book("数据结构", "李四", "ISBN002")

library.add_book(book1)
library.add_book(book2)

print(library.borrow_book("ISBN001"))
print(library.borrow_book("ISBN001"))  # 已借出
print(library.return_book("ISBN001"))

for book in library.list_available_books():
    print(book)

# 练习2：单例模式
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        self.value = 0

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True（同一个实例）

s1.value = 100
print(s2.value)  # 100

# 练习3：观察者模式
class Subject:
    def __init__(self):
        self._observers = []
        self._state = None
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self):
        for observer in self._observers:
            observer.update(self._state)
    
    def set_state(self, state):
        self._state = state
        self.notify()

class Observer:
    def __init__(self, name):
        self.name = name
    
    def update(self, state):
        print(f"{self.name} 收到更新: {state}")

# 使用观察者模式
subject = Subject()

observer1 = Observer("观察者1")
observer2 = Observer("观察者2")

subject.attach(observer1)
subject.attach(observer2)

subject.set_state("新状态")
# 输出:
# 观察者1 收到更新: 新状态
# 观察者2 收到更新: 新状态
```

---

## 3. 模块和包

模块和包是Python代码组织和重用的基本单位。

### 3.1 模块的基本概念

#### 核心概念：

- **模块（Module）**：一个包含Python代码的.py文件
- **包（Package）**：包含多个模块的目录，必须有`__init__.py`文件
- **导入（Import）**：使用其他模块中的代码
- **命名空间**：避免名称冲突的机制

### 3.2 创建和使用模块

#### 3.2.1 创建模块

创建一个名为`mymodule.py`的文件：

```python
# mymodule.py
"""这是一个示例模块"""

# 模块级变量
PI = 3.14159
VERSION = "1.0.0"

# 函数
def greet(name):
    """问候函数"""
    return f"Hello, {name}!"

def add(a, b):
    """加法函数"""
    return a + b

# 类
class Calculator:
    """计算器类"""
    def __init__(self):
        self.result = 0
    
    def add(self, n):
        self.result += n
        return self.result

# 私有函数（约定）
def _internal_function():
    """内部函数，不应被外部使用"""
    return "Internal"

# 模块级代码（导入时执行）
print(f"模块 {__name__} 已加载")
```

#### 3.2.2 导入模块

```python
# 方式1：导入整个模块
import mymodule

print(mymodule.PI)           # 3.14159
print(mymodule.greet("Alice"))  # Hello, Alice!
calc = mymodule.Calculator()

# 方式2：导入特定内容
from mymodule import greet, add, PI

print(PI)           # 3.14159
print(greet("Bob")) # Hello, Bob!
print(add(3, 5))    # 8

# 方式3：导入所有内容（不推荐）
from mymodule import *

# 方式4：使用别名
import mymodule as mm
from mymodule import Calculator as Calc

print(mm.PI)
calc = Calc()

# 方式5：从包中导入
from package.subpackage import module
```

### 3.3 模块搜索路径

```python
import sys

# 查看模块搜索路径
print(sys.path)
# 输出类似：
# ['', 'C:\\Python39\\lib', 'C:\\Python39\\lib\\site-packages', ...]

# 添加自定义路径
sys.path.append('/path/to/my/modules')

# 查看模块位置
import os
print(os.__file__)  # 显示os模块的文件路径
```

### 3.4 `__name__`和`__main__`

```python
# mymodule.py
def main():
    """主函数"""
    print("模块作为脚本运行")

# 只有直接运行此文件时才执行
if __name__ == "__main__":
    main()
    print(f"__name__ = {__name__}")  # __main__
else:
    print(f"模块被导入，__name__ = {__name__}")  # mymodule
```

### 3.5 包（Package）

#### 3.5.1 创建包

包是包含`__init__.py`文件的目录：

```
mypackage/
    __init__.py
    module1.py
    module2.py
    subpackage/
        __init__.py
        module3.py
```

```python
# mypackage/__init__.py
"""包初始化文件"""
print("mypackage 包已加载")

# 定义包级变量
__version__ = "1.0.0"
__all__ = ['module1', 'module2']  # 控制 from package import *

# 可以在这里导入子模块，方便使用
from .module1 import func1
from .module2 import func2
```

```python
# mypackage/module1.py
def func1():
    return "Function from module1"

class Class1:
    pass
```

```python
# mypackage/module2.py
def func2():
    return "Function from module2"
```

#### 3.5.2 使用包

```python
# 导入包
import mypackage
print(mypackage.__version__)

# 导入包中的模块
from mypackage import module1
print(module1.func1())

# 导入包中的特定内容
from mypackage.module1 import func1, Class1

# 相对导入（在包内部使用）
# mypackage/module2.py
from . import module1          # 从当前包导入
from .. import other_module    # 从父包导入
from .subpackage import module3  # 从子包导入
```

### 3.6 常用标准库模块

#### 3.6.1 os模块 - 操作系统接口

```python
import os

# 当前工作目录
print(os.getcwd())

# 改变目录
os.chdir('/path/to/directory')

# 列出目录内容
print(os.listdir('.'))

# 创建目录
os.mkdir('new_folder')
os.makedirs('parent/child/grandchild')  # 递归创建

# 删除目录
os.rmdir('new_folder')
os.removedirs('parent/child/grandchild')  # 递归删除

# 路径操作
path = os.path.join('folder', 'subfolder', 'file.txt')
print(os.path.exists(path))
print(os.path.isfile(path))
print(os.path.isdir(path))
print(os.path.basename(path))  # file.txt
print(os.path.dirname(path))   # folder/subfolder

# 环境变量
print(os.environ.get('PATH'))
os.environ['MY_VAR'] = 'value'

# 执行系统命令
os.system('echo Hello')
```

#### 3.6.2 sys模块 - 系统相关

```python
import sys

# 命令行参数
print(sys.argv)  # ['script.py', 'arg1', 'arg2']

# Python版本
print(sys.version)
print(sys.version_info)

# 平台信息
print(sys.platform)  # 'win32', 'linux', 'darwin'

# 退出程序
sys.exit(0)  # 0表示正常退出

# 标准输入输出
sys.stdout.write("Hello\n")
line = sys.stdin.readline()
```

#### 3.6.3 datetime模块 - 日期时间

```python
from datetime import datetime, date, time, timedelta

# 当前日期时间
now = datetime.now()
print(now)  # 2024-01-15 14:30:00.123456

today = date.today()
print(today)  # 2024-01-15

# 创建日期时间
dt = datetime(2024, 1, 15, 14, 30, 0)
d = date(2024, 1, 15)
t = time(14, 30, 0)

# 格式化
print(now.strftime("%Y-%m-%d %H:%M:%S"))
print(now.strftime("%Y年%m月%d日"))

# 解析字符串
dt = datetime.strptime("2024-01-15", "%Y-%m-%d")

# 时间运算
tomorrow = today + timedelta(days=1)
next_week = today + timedelta(weeks=1)
one_hour_later = now + timedelta(hours=1)

# 时间差
diff = datetime(2024, 12, 31) - now
print(diff.days)  # 剩余天数
```

#### 3.6.4 json模块 - JSON处理

```python
import json

# Python对象转JSON字符串
data = {
    'name': 'Alice',
    'age': 25,
    'hobbies': ['reading', 'coding']
}

json_str = json.dumps(data, indent=2, ensure_ascii=False)
print(json_str)

# JSON字符串转Python对象
parsed = json.loads(json_str)
print(parsed['name'])

# 写入文件
with open('data.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2, ensure_ascii=False)

# 从文件读取
with open('data.json', 'r', encoding='utf-8') as f:
    loaded = json.load(f)
```

#### 3.6.5 random模块 - 随机数

```python
import random

# 随机整数
print(random.randint(1, 10))      # 1到10之间
print(random.randrange(0, 10, 2)) # 0到10，步长2

# 随机浮点数
print(random.random())      # 0到1之间
print(random.uniform(1, 10)) # 1到10之间

# 随机选择
items = ['apple', 'banana', 'orange']
print(random.choice(items))
print(random.choices(items, k=2))  # 可重复选择
print(random.sample(items, 2))     # 不重复选择

# 打乱列表
random.shuffle(items)
print(items)

# 设置随机种子（可重现结果）
random.seed(42)
```

#### 3.6.6 re模块 - 正则表达式

```python
import re

text = "Contact: alice@example.com or bob@test.org"

# 搜索
match = re.search(r'\w+@\w+\.\w+', text)
if match:
    print(match.group())  # alice@example.com

# 查找所有
emails = re.findall(r'\w+@\w+\.\w+', text)
print(emails)  # ['alice@example.com', 'bob@test.org']

# 替换
new_text = re.sub(r'\w+@\w+\.\w+', '[EMAIL]', text)
print(new_text)

# 分割
parts = re.split(r'\s+', "hello   world  python")
print(parts)  # ['hello', 'world', 'python']

# 编译正则表达式（提高性能）
pattern = re.compile(r'\w+@\w+\.\w+')
matches = pattern.findall(text)
```

### 3.7 第三方包管理

#### 3.7.1 使用pip

```bash
# 安装包
pip install package_name

# 安装特定版本
pip install package_name==1.2.3

# 升级包
pip install --upgrade package_name

# 卸载包
pip uninstall package_name

# 列出已安装的包
pip list

# 查看包信息
pip show package_name

# 导出依赖
pip freeze > requirements.txt

# 安装依赖
pip install -r requirements.txt
```

#### 3.7.2 虚拟环境

```bash
# 创建虚拟环境
python -m venv myenv

# 激活虚拟环境
# Windows:
myenv\Scripts\activate
# Linux/Mac:
source myenv/bin/activate

# 退出虚拟环境
deactivate
```

### 3.8 模块和包实践

```python
# 练习1：创建一个实用工具包
# utils/__init__.py
"""实用工具包"""
__version__ = "1.0.0"

from .string_utils import *
from .math_utils import *

# utils/string_utils.py
def capitalize_words(text):
    """首字母大写"""
    return ' '.join(word.capitalize() for word in text.split())

def reverse_string(text):
    """反转字符串"""
    return text[::-1]

# utils/math_utils.py
def is_prime(n):
    """判断质数"""
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

def factorial(n):
    """计算阶乘"""
    if n == 0 or n == 1:
        return 1
    return n * factorial(n - 1)

# 使用工具包
from utils import capitalize_words, is_prime

print(capitalize_words("hello world"))  # Hello World
print(is_prime(17))  # True

# 练习2：配置管理模块
# config.py
import json
import os

class Config:
    """配置管理类"""
    def __init__(self, config_file='config.json'):
        self.config_file = config_file
        self.data = self.load()
    
    def load(self):
        """加载配置"""
        if os.path.exists(self.config_file):
            with open(self.config_file, 'r') as f:
                return json.load(f)
        return {}
    
    def save(self):
        """保存配置"""
        with open(self.config_file, 'w') as f:
            json.dump(self.data, f, indent=2)
    
    def get(self, key, default=None):
        """获取配置项"""
        return self.data.get(key, default)
    
    def set(self, key, value):
        """设置配置项"""
        self.data[key] = value
        self.save()

# 使用配置管理
config = Config()
config.set('app_name', 'MyApp')
config.set('version', '1.0.0')
print(config.get('app_name'))
```

---

## 4. 异常处理

异常处理是处理程序运行时错误的机制。

### 4.1 异常的基本概念

#### 核心概念：

- **异常（Exception）**：程序运行时发生的错误
- **抛出异常**：使用`raise`语句
- **捕获异常**：使用`try-except`语句
- **异常层次**：所有异常都继承自`BaseException`

### 4.2 常见异常类型

```python
# ZeroDivisionError - 除零错误
# result = 10 / 0

# ValueError - 值错误
# number = int("abc")

# TypeError - 类型错误
# result = "hello" + 5

# IndexError - 索引错误
# lst = [1, 2, 3]
# print(lst[10])

# KeyError - 键错误
# d = {'a': 1}
# print(d['b'])

# FileNotFoundError - 文件未找到
# with open('nonexistent.txt') as f:
#     pass

# AttributeError - 属性错误
# x = 5
# x.append(1)

# NameError - 名称错误
# print(undefined_variable)

# ImportError - 导入错误
# import nonexistent_module
```

### 4.3 try-except语句

#### 4.3.1 基本用法

```python
# 捕获特定异常
try:
    number = int(input("请输入数字: "))
    result = 10 / number
    print(f"结果: {result}")
except ValueError:
    print("输入不是有效的数字")
except ZeroDivisionError:
    print("不能除以零")

# 捕获多个异常
try:
    # 可能出错的代码
    pass
except (ValueError, TypeError) as e:
    print(f"发生错误: {e}")

# 捕获所有异常（不推荐）
try:
    # 代码
    pass
except Exception as e:
    print(f"发生异常: {e}")
```

#### 4.3.2 else和finally子句

```python
try:
    file = open('data.txt', 'r')
    data = file.read()
except FileNotFoundError:
    print("文件不存在")
else:
    # 没有异常时执行
    print("文件读取成功")
    print(data)
finally:
    # 无论是否发生异常都执行
    if 'file' in locals():
        file.close()
    print("清理完成")
```

#### 4.3.3 完整的异常处理结构

```python
def divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("除数不能为零")
        return None
    except TypeError:
        print("参数类型错误")
        return None
    else:
        print("计算成功")
        return result
    finally:
        print("函数执行完毕")

print(divide(10, 2))   # 正常
print(divide(10, 0))   # 除零
print(divide(10, "2")) # 类型错误
```

### 4.4 抛出异常

#### 4.4.1 使用raise语句

```python
def validate_age(age):
    """验证年龄"""
    if not isinstance(age, int):
        raise TypeError("年龄必须是整数")
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄不合理")
    return True

try:
    validate_age(-5)
except ValueError as e:
    print(f"验证失败: {e}")

# 重新抛出异常
try:
    validate_age("abc")
except TypeError:
    print("捕获到类型错误，重新抛出")
    raise  # 重新抛出当前异常
```

#### 4.4.2 异常链

```python
def process_data(data):
    try:
        result = int(data)
    except ValueError as e:
        # 抛出新异常，保留原异常信息
        raise RuntimeError("数据处理失败") from e

try:
    process_data("abc")
except RuntimeError as e:
    print(f"错误: {e}")
    print(f"原因: {e.__cause__}")
```

### 4.5 自定义异常

```python
# 定义自定义异常类
class ValidationError(Exception):
    """验证错误"""
    pass

class AgeValidationError(ValidationError):
    """年龄验证错误"""
    def __init__(self, age, message="年龄验证失败"):
        self.age = age
        self.message = message
        super().__init__(self.message)
    
    def __str__(self):
        return f"{self.message}: 年龄={self.age}"

class EmailValidationError(ValidationError):
    """邮箱验证错误"""
    pass

# 使用自定义异常
def validate_user(age, email):
    if age < 0 or age > 150:
        raise AgeValidationError(age)
    if '@' not in email:
        raise EmailValidationError(f"无效的邮箱: {email}")
    return True

try:
    validate_user(-5, "test@example.com")
except AgeValidationError as e:
    print(e)
except EmailValidationError as e:
    print(e)
except ValidationError as e:
    print(f"验证错误: {e}")
```

### 4.6 上下文管理器和异常

```python
# 使用with语句自动处理异常
with open('file.txt', 'r') as f:
    data = f.read()
# 文件自动关闭，即使发生异常

# 自定义上下文管理器
class DatabaseConnection:
    def __init__(self, db_name):
        self.db_name = db_name
        self.connection = None
    
    def __enter__(self):
        print(f"连接到数据库: {self.db_name}")
        self.connection = f"Connection to {self.db_name}"
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"关闭数据库连接: {self.db_name}")
        if exc_type is not None:
            print(f"发生异常: {exc_type.__name__}: {exc_val}")
        # 返回True表示抑制异常，False表示传播异常
        return False

# 使用上下文管理器
try:
    with DatabaseConnection("mydb") as conn:
        print(f"使用连接: {conn}")
        # raise ValueError("测试异常")
except ValueError as e:
    print(f"捕获异常: {e}")

# 使用contextlib简化
from contextlib import contextmanager

@contextmanager
def file_manager(filename, mode):
    try:
        f = open(filename, mode)
        yield f
    finally:
        f.close()

with file_manager('test.txt', 'w') as f:
    f.write('Hello, World!')
```

### 4.7 异常处理最佳实践

```python
# 1. 具体异常优先
try:
    value = int(input("输入数字: "))
except ValueError:  # 具体异常
    print("不是有效数字")
except Exception:   # 通用异常
    print("其他错误")

# 2. 不要捕获所有异常
# 错误示范
try:
    # 代码
    pass
except:  # 不推荐！会捕获KeyboardInterrupt等
    pass

# 正确做法
try:
    # 代码
    pass
except Exception as e:  # 至少指定Exception
    print(f"错误: {e}")

# 3. 记录异常信息
import logging

logging.basicConfig(level=logging.ERROR)

try:
    result = 10 / 0
except ZeroDivisionError as e:
    logging.error("除零错误", exc_info=True)

# 4. 清理资源
file = None
try:
    file = open('data.txt', 'r')
    data = file.read()
except FileNotFoundError:
    print("文件不存在")
finally:
    if file:
        file.close()

# 5. 异常处理实践示例
class DataProcessor:
    def __init__(self, filename):
        self.filename = filename
        self.data = None
    
    def load_data(self):
        """加载数据"""
        try:
            with open(self.filename, 'r') as f:
                self.data = f.read()
            return True
        except FileNotFoundError:
            print(f"文件不存在: {self.filename}")
            return False
        except PermissionError:
            print(f"没有权限读取: {self.filename}")
            return False
        except Exception as e:
            print(f"未知错误: {e}")
            return False
    
    def process(self):
        """处理数据"""
        if self.data is None:
            raise RuntimeError("数据未加载")
        
        try:
            # 处理逻辑
            result = len(self.data)
            return result
        except Exception as e:
            raise RuntimeError("数据处理失败") from e

# 使用
processor = DataProcessor('data.txt')
if processor.load_data():
    try:
        result = processor.process()
        print(f"处理结果: {result}")
    except RuntimeError as e:
        print(f"处理错误: {e}")
```

---

## 5. 文件操作

文件操作是程序与外部数据交互的重要方式。

### 5.1 文件操作基本概念

#### 核心概念：

- **文件对象**：Python中表示文件的对象
- **文件模式**：指定文件的打开方式（读、写、追加等）
- **文件指针**：指示当前读写位置
- **编码**：文本文件的字符编码（如UTF-8）

### 5.2 打开和关闭文件

#### 5.2.1 基本文件操作

```python
# 打开文件
file = open('example.txt', 'r', encoding='utf-8')

# 读取内容
content = file.read()
print(content)

# 关闭文件
file.close()

# 推荐：使用with语句（自动关闭）
with open('example.txt', 'r', encoding='utf-8') as file:
    content = file.read()
    print(content)
# 文件自动关闭
```

#### 5.2.2 文件模式

```python
# 文本模式
# 'r'  - 只读（默认）
# 'w'  - 写入（覆盖）
# 'a'  - 追加
# 'x'  - 创建（文件存在则报错）
# 'r+' - 读写
# 'w+' - 读写（覆盖）
# 'a+' - 读写（追加）

# 二进制模式（添加'b'）
# 'rb' - 二进制只读
# 'wb' - 二进制写入
# 'ab' - 二进制追加

# 示例
with open('data.txt', 'w', encoding='utf-8') as f:
    f.write('Hello, World!')

with open('data.txt', 'r', encoding='utf-8') as f:
    print(f.read())

with open('data.txt', 'a', encoding='utf-8') as f:
    f.write('\nNew line')
```

### 5.3 读取文件

#### 5.3.1 读取整个文件

```python
# read() - 读取全部内容
with open('example.txt', 'r', encoding='utf-8') as f:
    content = f.read()
    print(content)

# 读取指定字节数
with open('example.txt', 'r', encoding='utf-8') as f:
    chunk = f.read(100)  # 读取前100个字符
    print(chunk)
```

#### 5.3.2 逐行读取

```python
# readline() - 读取一行
with open('example.txt', 'r', encoding='utf-8') as f:
    line1 = f.readline()
    line2 = f.readline()
    print(line1)
    print(line2)

# readlines() - 读取所有行到列表
with open('example.txt', 'r', encoding='utf-8') as f:
    lines = f.readlines()
    for line in lines:
        print(line.strip())  # strip()去除换行符

# 推荐：迭代文件对象（内存高效）
with open('example.txt', 'r', encoding='utf-8') as f:
    for line in f:
        print(line.strip())
```

#### 5.3.3 读取大文件

```python
# 分块读取大文件
def read_large_file(filename, chunk_size=1024):
    """逐块读取大文件"""
    with open(filename, 'r', encoding='utf-8') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk

# 使用生成器读取
for chunk in read_large_file('large_file.txt'):
    process(chunk)  # 处理数据块

# 逐行读取大文件（推荐）
with open('large_file.txt', 'r', encoding='utf-8') as f:
    for line in f:
        process(line)  # 逐行处理
```

### 5.4 写入文件

#### 5.4.1 基本写入

```python
# write() - 写入字符串
with open('output.txt', 'w', encoding='utf-8') as f:
    f.write('Hello, World!\n')
    f.write('Second line\n')

# writelines() - 写入字符串列表
lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
with open('output.txt', 'w', encoding='utf-8') as f:
    f.writelines(lines)

# 追加模式
with open('output.txt', 'a', encoding='utf-8') as f:
    f.write('Appended line\n')
```

#### 5.4.2 格式化写入

```python
# 使用print写入文件
with open('output.txt', 'w', encoding='utf-8') as f:
    print('Hello, World!', file=f)
    print('Second line', file=f)

# 写入格式化数据
data = [
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 30}
]

with open('data.txt', 'w', encoding='utf-8') as f:
    for item in data:
        f.write(f"{item['name']},{item['age']}\n")
```

### 5.5 文件指针操作

```python
with open('example.txt', 'r', encoding='utf-8') as f:
    # tell() - 获取当前位置
    print(f"当前位置: {f.tell()}")
    
    # 读取一些内容
    content = f.read(10)
    print(f"读取后位置: {f.tell()}")
    
    # seek(offset, whence) - 移动文件指针
    # whence: 0-文件开头, 1-当前位置, 2-文件末尾
    f.seek(0)  # 回到开头
    print(f"移动后位置: {f.tell()}")
    
    # 读取全部内容
    all_content = f.read()
    
    # 回到开头
    f.seek(0)
    first_line = f.readline()
```

### 5.6 二进制文件操作

```python
# 读取二进制文件
with open('image.png', 'rb') as f:
    binary_data = f.read()
    print(f"文件大小: {len(binary_data)} 字节")

# 写入二进制文件
data = b'\x00\x01\x02\x03\x04'
with open('binary.dat', 'wb') as f:
    f.write(data)

# 复制二进制文件
def copy_file(source, destination):
    """复制文件"""
    with open(source, 'rb') as src:
        with open(destination, 'wb') as dst:
            # 分块复制
            chunk_size = 4096
            while True:
                chunk = src.read(chunk_size)
                if not chunk:
                    break
                dst.write(chunk)

copy_file('source.jpg', 'destination.jpg')
```

### 5.7 文件和目录操作

#### 5.7.1 使用os模块

```python
import os

# 检查文件/目录是否存在
if os.path.exists('file.txt'):
    print("文件存在")

# 检查是否为文件
if os.path.isfile('file.txt'):
    print("是文件")

# 检查是否为目录
if os.path.isdir('folder'):
    print("是目录")

# 获取文件大小
size = os.path.getsize('file.txt')
print(f"文件大小: {size} 字节")

# 获取文件修改时间
import time
mtime = os.path.getmtime('file.txt')
print(f"修改时间: {time.ctime(mtime)}")

# 重命名文件
os.rename('old_name.txt', 'new_name.txt')

# 删除文件
os.remove('file.txt')

# 创建目录
os.mkdir('new_folder')
os.makedirs('parent/child/grandchild')  # 递归创建

# 删除目录
os.rmdir('empty_folder')  # 只能删除空目录
import shutil
shutil.rmtree('folder')   # 递归删除目录及内容

# 列出目录内容
files = os.listdir('.')
for file in files:
    print(file)

# 遍历目录树
for root, dirs, files in os.walk('.'):
    print(f"目录: {root}")
    for file in files:
        filepath = os.path.join(root, file)
        print(f"  文件: {filepath}")
```

#### 5.7.2 使用pathlib模块（推荐）

```python
from pathlib import Path

# 创建Path对象
p = Path('example.txt')

# 检查存在
if p.exists():
    print("文件存在")

# 检查类型
if p.is_file():
    print("是文件")
if p.is_dir():
    print("是目录")

# 读写文件
p = Path('data.txt')
p.write_text('Hello, World!', encoding='utf-8')
content = p.read_text(encoding='utf-8')

# 二进制读写
p.write_bytes(b'\x00\x01\x02')
data = p.read_bytes()

# 文件信息
print(f"文件名: {p.name}")
print(f"扩展名: {p.suffix}")
print(f"文件大小: {p.stat().st_size}")

# 路径操作
p = Path('folder') / 'subfolder' / 'file.txt'
print(p)  # folder/subfolder/file.txt

# 创建目录
p = Path('new_folder')
p.mkdir(exist_ok=True)  # exist_ok=True避免已存在时报错
p.mkdir(parents=True, exist_ok=True)  # 递归创建

# 遍历目录
p = Path('.')
for item in p.iterdir():
    print(item)

# 查找文件（glob模式）
for txt_file in p.glob('*.txt'):
    print(txt_file)

# 递归查找
for py_file in p.rglob('*.py'):
    print(py_file)

# 删除文件
p = Path('file.txt')
p.unlink()  # 删除文件

# 重命名
p.rename('new_name.txt')
```

### 5.8 CSV文件操作

```python
import csv

# 写入CSV文件
data = [
    ['Name', 'Age', 'City'],
    ['Alice', 25, 'Beijing'],
    ['Bob', 30, 'Shanghai'],
    ['Charlie', 35, 'Guangzhou']
]

with open('data.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerows(data)

# 读取CSV文件
with open('data.csv', 'r', encoding='utf-8') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

# 使用字典读写CSV
# 写入
data = [
    {'name': 'Alice', 'age': 25, 'city': 'Beijing'},
    {'name': 'Bob', 'age': 30, 'city': 'Shanghai'}
]

with open('data.csv', 'w', newline='', encoding='utf-8') as f:
    fieldnames = ['name', 'age', 'city']
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(data)

# 读取
with open('data.csv', 'r', encoding='utf-8') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(f"{row['name']}: {row['age']}")
```

### 5.9 JSON文件操作

```python
import json

# 写入JSON文件
data = {
    'name': 'Alice',
    'age': 25,
    'hobbies': ['reading', 'coding'],
    'address': {
        'city': 'Beijing',
        'country': 'China'
    }
}

with open('data.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, indent=2, ensure_ascii=False)

# 读取JSON文件
with open('data.json', 'r', encoding='utf-8') as f:
    loaded_data = json.load(f)
    print(loaded_data['name'])

# 处理JSON字符串
json_str = json.dumps(data, indent=2, ensure_ascii=False)
parsed = json.loads(json_str)
```

### 5.10 压缩文件操作

#### 5.10.1 ZIP文件

```python
import zipfile

# 创建ZIP文件
with zipfile.ZipFile('archive.zip', 'w') as zipf:
    zipf.write('file1.txt')
    zipf.write('file2.txt')
    zipf.write('folder/file3.txt')

# 读取ZIP文件
with zipfile.ZipFile('archive.zip', 'r') as zipf:
    # 列出文件
    print(zipf.namelist())
    
    # 提取所有文件
    zipf.extractall('extracted')
    
    # 提取单个文件
    zipf.extract('file1.txt', 'extracted')
    
    # 读取文件内容
    with zipf.open('file1.txt') as f:
        content = f.read()
        print(content.decode('utf-8'))
```

#### 5.10.2 其他压缩格式

```python
import gzip
import bz2

# gzip压缩
with gzip.open('file.txt.gz', 'wt', encoding='utf-8') as f:
    f.write('Hello, World!')

with gzip.open('file.txt.gz', 'rt', encoding='utf-8') as f:
    content = f.read()
    print(content)

# bz2压缩
with bz2.open('file.txt.bz2', 'wt', encoding='utf-8') as f:
    f.write('Hello, World!')

with bz2.open('file.txt.bz2', 'rt', encoding='utf-8') as f:
    content = f.read()
    print(content)
```

### 5.11 临时文件

```python
import tempfile

# 创建临时文件
with tempfile.TemporaryFile(mode='w+', encoding='utf-8') as f:
    f.write('Temporary data')
    f.seek(0)
    print(f.read())
# 文件自动删除

# 创建命名临时文件
with tempfile.NamedTemporaryFile(mode='w+', delete=False, encoding='utf-8') as f:
    print(f"临时文件: {f.name}")
    f.write('Temporary data')

# 创建临时目录
with tempfile.TemporaryDirectory() as tmpdir:
    print(f"临时目录: {tmpdir}")
    # 使用临时目录
    temp_file = Path(tmpdir) / 'temp.txt'
    temp_file.write_text('data')
# 目录自动删除
```

### 5.12 文件操作实践

```python
# 练习1：文件统计工具
def file_statistics(filename):
    """统计文件信息"""
    try:
        with open(filename, 'r', encoding='utf-8') as f:
            lines = f.readlines()
            
        stats = {
            'lines': len(lines),
            'words': sum(len(line.split()) for line in lines),
            'chars': sum(len(line) for line in lines),
            'size': Path(filename).stat().st_size
        }
        
        return stats
    except FileNotFoundError:
        return None

# 使用
stats = file_statistics('example.txt')
if stats:
    print(f"行数: {stats['lines']}")
    print(f"单词数: {stats['words']}")
    print(f"字符数: {stats['chars']}")
    print(f"文件大小: {stats['size']} 字节")

# 练习2：日志记录器
class Logger:
    """简单的日志记录器"""
    def __init__(self, filename):
        self.filename = filename
    
    def log(self, message, level='INFO'):
        """记录日志"""
        from datetime import datetime
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        log_entry = f"[{timestamp}] [{level}] {message}\n"
        
        with open(self.filename, 'a', encoding='utf-8') as f:
            f.write(log_entry)
    
    def info(self, message):
        self.log(message, 'INFO')
    
    def warning(self, message):
        self.log(message, 'WARNING')
    
    def error(self, message):
        self.log(message, 'ERROR')

# 使用
logger = Logger('app.log')
logger.info('应用启动')
logger.warning('这是一个警告')
logger.error('发生错误')

# 练习3：配置文件管理
class ConfigManager:
    """配置文件管理器"""
    def __init__(self, config_file='config.json'):
        self.config_file = config_file
        self.config = self.load()
    
    def load(self):
        """加载配置"""
        if Path(self.config_file).exists():
            with open(self.config_file, 'r', encoding='utf-8') as f:
                return json.load(f)
        return {}
    
    def save(self):
        """保存配置"""
        with open(self.config_file, 'w', encoding='utf-8') as f:
            json.dump(self.config, f, indent=2, ensure_ascii=False)
    
    def get(self, key, default=None):
        """获取配置项"""
        return self.config.get(key, default)
    
    def set(self, key, value):
        """设置配置项"""
        self.config[key] = value
        self.save()
    
    def delete(self, key):
        """删除配置项"""
        if key in self.config:
            del self.config[key]
            self.save()

# 使用
config = ConfigManager()
config.set('app_name', 'MyApp')
config.set('version', '1.0.0')
print(config.get('app_name'))

# 练习4：文件备份工具
def backup_file(source, backup_dir='backups'):
    """备份文件"""
    from datetime import datetime
    import shutil
    
    # 创建备份目录
    backup_path = Path(backup_dir)
    backup_path.mkdir(exist_ok=True)
    
    # 生成备份文件名
    source_path = Path(source)
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_name = f"{source_path.stem}_{timestamp}{source_path.suffix}"
    backup_file = backup_path / backup_name
    
    # 复制文件
    shutil.copy2(source, backup_file)
    print(f"备份完成: {backup_file}")
    
    return backup_file

# 使用
backup_file('important.txt')

# 练习5：批量文件处理
def batch_rename(directory, pattern, replacement):
    """批量重命名文件"""
    path = Path(directory)
    renamed_count = 0
    
    for file in path.glob('*'):
        if file.is_file() and pattern in file.name:
            new_name = file.name.replace(pattern, replacement)
            new_path = file.parent / new_name
            file.rename(new_path)
            print(f"重命名: {file.name} -> {new_name}")
            renamed_count += 1
    
    print(f"共重命名 {renamed_count} 个文件")

# 使用
# batch_rename('.', 'old', 'new')

# 练习6：文件搜索工具
def search_in_files(directory, search_text, file_pattern='*.txt'):
    """在文件中搜索文本"""
    results = []
    path = Path(directory)
    
    for file in path.rglob(file_pattern):
        try:
            with open(file, 'r', encoding='utf-8') as f:
                for line_num, line in enumerate(f, 1):
                    if search_text in line:
                        results.append({
                            'file': str(file),
                            'line': line_num,
                            'content': line.strip()
                        })
        except Exception as e:
            print(f"无法读取 {file}: {e}")
    
    return results

# 使用
results = search_in_files('.', 'Python', '*.py')
for result in results:
    print(f"{result['file']}:{result['line']} - {result['content']}")
```

---

## 总结

本教程详细介绍了Python的进阶知识：

1. **函数**：函数定义、参数类型、装饰器、生成器等高级特性
2. **面向对象编程**：类与对象、继承、封装、多态、特殊方法等
3. **模块和包**：代码组织、标准库使用、包管理
4. **异常处理**：异常捕获、自定义异常、上下文管理器
5. **文件操作**：文件读写、目录操作、各种文件格式处理

### 学习建议

- **深入实践**：通过实际项目巩固所学知识
- **阅读源码**：学习优秀开源项目的代码
- **编写文档**：养成为代码编写文档的习惯
- **测试驱动**：学习单元测试和测试驱动开发
- **性能优化**：了解Python性能优化技巧

### 下一步学习方向

- **高级特性**：元类、描述符、上下文管理器
- **并发编程**：多线程、多进程、异步编程
- **网络编程**：Socket、HTTP、Web框架
- **数据处理**：NumPy、Pandas、数据分析
- **Web开发**：Django、Flask、FastAPI
- **机器学习**：Scikit-learn、TensorFlow、PyTorch

---

**继续学习，不断进步！🚀**

