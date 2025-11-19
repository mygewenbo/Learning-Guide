# Python函数进阶教程

> 本教程深入讲解Python函数的高级特性，包括参数传递、作用域、闭包、装饰器、高阶函数、生成器等核心概念，基于Python 3.13+编写

---

## 目录

1. [函数参数的深入理解](#1-函数参数的深入理解)
2. [作用域与闭包](#2-作用域与闭包)
3. [装饰器详解](#3-装饰器详解)
4. [高阶函数与Lambda表达式](#4-高阶函数与lambda表达式)
5. [生成器与迭代器](#5-生成器与迭代器)
6. [递归与高级技巧](#6-递归与高级技巧)

---

## 1. 函数参数的深入理解

### 1.1 参数类型全解析

在Python中，函数参数有多种类型，理解它们的区别对于编写灵活、易用的函数至关重要。

#### 1.1.1 位置参数（Positional Arguments）

**什么是位置参数？**

位置参数是最基本的参数类型，调用函数时必须按照定义的顺序传递参数。参数的位置决定了它们对应的形参。

```python
def greet(name, age):
    """根据名字和年龄打招呼"""
    print(f"你好，{name}！你今年{age}岁了。")

# 调用时必须按顺序传递
greet("张三", 25)  # 输出: 你好，张三！你今年25岁了。

# 位置错误会导致混乱
greet(25, "张三")  # 输出: 你好，25！你今年张三岁了。（逻辑错误）
```

#### 1.1.2 关键字参数（Keyword Arguments）

**什么是关键字参数？**

关键字参数允许在调用函数时指定参数名称，这样就可以不按顺序传递参数。这提高了代码的可读性。

```python
def create_user(username, email, age, city):
    """创建用户信息"""
    return {
        'username': username,
        'email': email,
        'age': age,
        'city': city
    }

# 使用关键字参数，顺序可以任意
user = create_user(
    age=28,
    city="北京",
    username="zhangsan",
    email="zhangsan@example.com"
)
print(user)
# 输出: {'username': 'zhangsan', 'email': 'zhangsan@example.com', 'age': 28, 'city': '北京'}

# 也可以混合使用位置参数和关键字参数（但位置参数必须在前）
user2 = create_user("lisi", "lisi@example.com", age=30, city="上海")
```

#### 1.1.3 默认参数（Default Arguments）

**什么是默认参数？**

默认参数允许在定义函数时为参数指定默认值。如果调用时没有提供该参数，则使用默认值。

```python
def connect_database(host, port=3306, timeout=30, retry=3):
    """连接数据库"""
    print(f"连接到 {host}:{port}")
    print(f"超时时间: {timeout}秒, 重试次数: {retry}")
    return True

# 只传必需的参数
connect_database("localhost")
# 输出:
# 连接到 localhost:3306
# 超时时间: 30秒, 重试次数: 3

# 覆盖部分默认参数
connect_database("192.168.1.100", port=5432, timeout=60)
# 输出:
# 连接到 192.168.1.100:5432
# 超时时间: 60秒, 重试次数: 3
```

**⚠️ 默认参数的陷阱：可变对象**

这是Python中一个常见的坑！默认参数在函数定义时就被创建，如果默认值是可变对象（如列表、字典），所有函数调用都会共享同一个对象。

```python
# ❌ 错误示例：使用可变对象作为默认参数
def add_student_wrong(name, students=[]):
    """错误的做法"""
    students.append(name)
    return students

class1 = add_student_wrong("张三")
print(class1)  # ['张三']

class2 = add_student_wrong("李四")
print(class2)  # ['张三', '李四']  # 意外！李四加到了同一个列表

print(class1)  # ['张三', '李四']  # class1也被修改了！


# ✅ 正确示例：使用None作为默认值
def add_student_correct(name, students=None):
    """正确的做法"""
    if students is None:
        students = []  # 每次调用都创建新列表
    students.append(name)
    return students

class1 = add_student_correct("张三")
print(class1)  # ['张三']

class2 = add_student_correct("李四")
print(class2)  # ['李四']  # 正确！

print(class1)  # ['张三']  # class1未被影响
```

#### 1.1.4 可变参数：*args 和 **kwargs

**什么是可变参数？**

有时候我们不知道函数会接收多少个参数，这时就需要可变参数。Python提供了两种可变参数：
- `*args`: 接收任意数量的位置参数，打包成元组
- `**kwargs`: 接收任意数量的关键字参数，打包成字典

##### *args - 可变位置参数

```python
def calculate_sum(*numbers):
    """计算任意数量数字的和"""
    print(f"接收到的参数类型: {type(numbers)}")  # <class 'tuple'>
    print(f"参数内容: {numbers}")
    
    total = 0
    for num in numbers:
        total += num
    return total

# 可以传递任意数量的参数
print(calculate_sum(1, 2, 3))           # 输出: 6
print(calculate_sum(10, 20, 30, 40))    # 输出: 100
print(calculate_sum(5))                 # 输出: 5
print(calculate_sum())                  # 输出: 0


def print_info(name, *hobbies):
    """打印姓名和爱好"""
    print(f"姓名: {name}")
    if hobbies:
        print("爱好:")
        for hobby in hobbies:
            print(f"  - {hobby}")
    else:
        print("没有提供爱好信息")

print_info("张三", "篮球", "游泳", "阅读")
# 输出:
# 姓名: 张三
# 爱好:
#   - 篮球
#   - 游泳
#   - 阅读
```

##### **kwargs - 可变关键字参数

```python
def create_profile(**info):
    """创建用户档案"""
    print(f"接收到的参数类型: {type(info)}")  # <class 'dict'>
    print(f"参数内容: {info}")
    
    print("\n用户档案:")
    for key, value in info.items():
        print(f"  {key}: {value}")

create_profile(name="张三", age=25, city="北京", job="工程师")
# 输出:
# 接收到的参数类型: <class 'dict'>
# 参数内容: {'name': '张三', 'age': 25, 'city': '北京', 'job': '工程师'}
# 
# 用户档案:
#   name: 张三
#   age: 25
#   city: 北京
#   job: 工程师


# 实际应用：配置函数
def setup_server(host, port, **options):
    """设置服务器配置"""
    print(f"服务器地址: {host}:{port}")
    
    # 处理可选配置
    max_connections = options.get('max_connections', 100)
    timeout = options.get('timeout', 30)
    ssl_enabled = options.get('ssl_enabled', False)
    
    print(f"最大连接数: {max_connections}")
    print(f"超时时间: {timeout}秒")
    print(f"SSL加密: {'启用' if ssl_enabled else '禁用'}")
    
    # 显示其他未知配置
    known_keys = {'max_connections', 'timeout', 'ssl_enabled'}
    other_options = {k: v for k, v in options.items() if k not in known_keys}
    if other_options:
        print("其他配置:", other_options)

setup_server("localhost", 8080, 
             max_connections=200, 
             ssl_enabled=True, 
             debug_mode=True,
             log_level="INFO")
```

##### 组合使用 *args 和 **kwargs

```python
def advanced_function(required, *args, optional=None, **kwargs):
    """展示所有参数类型的组合"""
    print(f"必需参数: {required}")
    print(f"位置参数(*args): {args}")
    print(f"可选关键字参数: {optional}")
    print(f"关键字参数(**kwargs): {kwargs}")

advanced_function(
    "必需的",           # required
    "额外1", "额外2",    # *args
    optional="可选值",   # optional
    key1="值1",         # **kwargs
    key2="值2"
)
# 输出:
# 必需参数: 必需的
# 位置参数(*args): ('额外1', '额外2')
# 可选关键字参数: 可选值
# 关键字参数(**kwargs): {'key1': '值1', 'key2': '值2'}
```

#### 1.1.5 仅限位置参数和仅限关键字参数

Python 3.8+ 引入了明确的语法来限制参数的传递方式。

##### 仅限位置参数（Positional-Only Parameters）

使用 `/` 来标记，`/` 之前的参数只能通过位置传递，不能使用关键字。

```python
def divide(dividend, divisor, /):
    """除法函数，参数只能按位置传递"""
    return dividend / divisor

# ✅ 正确：使用位置参数
print(divide(10, 2))  # 输出: 5.0

# ❌ 错误：不能使用关键字参数
# print(divide(dividend=10, divisor=2))  # TypeError
```

**为什么需要仅限位置参数？**

1. 提高性能（避免关键字参数的开销）
2. 允许将来修改参数名而不影响API
3. 防止用户依赖参数名

```python
def process_data(data, /, *, format='json'):
    """
    处理数据
    data: 只能按位置传递
    format: 只能通过关键字传递
    """
    print(f"处理数据: {data}")
    print(f"格式: {format}")

# ✅ 正确用法
process_data([1, 2, 3], format='xml')

# ❌ 错误用法
# process_data(data=[1, 2, 3], format='xml')  # TypeError: data不能用关键字
# process_data([1, 2, 3], 'xml')  # TypeError: format必须用关键字
```

##### 仅限关键字参数（Keyword-Only Parameters）

使用 `*` 来标记，`*` 之后的参数只能通过关键字传递。

```python
def send_email(recipient, *, subject, body, cc=None):
    """
    发送邮件
    recipient: 可以按位置或关键字传递
    subject, body: 必须使用关键字传递
    cc: 可选的关键字参数
    """
    print(f"收件人: {recipient}")
    print(f"主题: {subject}")
    print(f"正文: {body}")
    if cc:
        print(f"抄送: {cc}")

# ✅ 正确用法
send_email("user@example.com", subject="问候", body="你好！")
send_email("user@example.com", subject="通知", body="会议", cc="boss@example.com")

# ❌ 错误用法
# send_email("user@example.com", "问候", "你好！")  # TypeError
```

**实际应用场景：**

```python
def create_connection(host, port=3306, /, *, 
                     username, password, 
                     database=None, 
                     charset='utf8mb4',
                     **options):
    """
    创建数据库连接
    
    host, port: 仅限位置参数
    username, password: 必需的关键字参数
    database, charset: 可选的关键字参数
    **options: 其他配置选项
    """
    print(f"连接到: {host}:{port}")
    print(f"用户: {username}")
    print(f"数据库: {database or '未指定'}")
    print(f"字符集: {charset}")
    if options:
        print(f"其他选项: {options}")

# ✅ 正确的调用方式
create_connection("localhost", 3306, 
                 username="root", 
                 password="secret",
                 database="mydb",
                 timeout=30,
                 autocommit=True)
```

### 1.2 参数解包

**什么是参数解包？**

参数解包允许我们将序列（如列表、元组）或字典解包为函数的参数。这在处理动态参数时非常有用。

#### 使用 * 解包序列

```python
def calculate_average(a, b, c):
    """计算三个数的平均值"""
    return (a + b + c) / 3

# 有一个包含三个数字的列表
scores = [85, 90, 78]

# ❌ 错误：直接传递列表
# print(calculate_average(scores))  # TypeError

# ✅ 正确：解包列表
print(calculate_average(*scores))  # 输出: 84.33333333333333

# 相当于: calculate_average(85, 90, 78)


# 实际应用：合并多个列表
list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]

combined = [*list1, *list2, *list3]
print(combined)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 使用 ** 解包字典

```python
def register_user(username, email, age, city):
    """注册用户"""
    print(f"用户名: {username}")
    print(f"邮箱: {email}")
    print(f"年龄: {age}")
    print(f"城市: {city}")

# 用户数据存储在字典中
user_data = {
    'username': 'zhangsan',
    'email': 'zhangsan@example.com',
    'age': 25,
    'city': '北京'
}

# ✅ 解包字典
register_user(**user_data)


# 实际应用：合并多个字典
default_config = {'host': 'localhost', 'port': 8080, 'debug': False}
user_config = {'port': 9000, 'debug': True}

# Python 3.9+ 支持字典合并操作符 |
final_config = default_config | user_config
print(final_config)  # {'host': 'localhost', 'port': 9000, 'debug': True}

# 使用解包合并（适用于所有Python 3版本）
final_config = {**default_config, **user_config}
print(final_config)  # {'host': 'localhost', 'port': 9000, 'debug': True}
```

#### 高级应用：函数参数转发

```python
def log_function_call(func):
    """记录函数调用的装饰器（后面会详细讲解装饰器）"""
    def wrapper(*args, **kwargs):
        print(f"调用函数: {func.__name__}")
        print(f"  位置参数: {args}")
        print(f"  关键字参数: {kwargs}")
        # 转发所有参数给原函数
        result = func(*args, **kwargs)
        print(f"  返回值: {result}")
        return result
    return wrapper

@log_function_call
def create_user(name, age, city="未知"):
    return f"用户{name}已创建"

create_user("张三", 25, city="北京")
# 输出:
# 调用函数: create_user
#   位置参数: ('张三', 25)
#   关键字参数: {'city': '北京'}
#   返回值: 用户张三已创建
```

---

## 2. 作用域与闭包

### 2.1 作用域规则（LEGB规则）

**什么是作用域？**

作用域是程序中变量可以被访问的区域。Python使用LEGB规则来查找变量：

- **L (Local)**: 本地作用域 - 函数内部
- **E (Enclosing)**: 嵌套作用域 - 外层函数的作用域
- **G (Global)**: 全局作用域 - 模块级别
- **B (Built-in)**: 内置作用域 - Python内置函数和异常

Python按照这个顺序查找变量：L → E → G → B

```python
# B (Built-in) - 内置作用域
# len, print, max 等都是内置函数

# G (Global) - 全局作用域
global_var = "我是全局变量"

def outer_function():
    # E (Enclosing) - 嵌套作用域
    enclosing_var = "我是外层函数变量"
    
    def inner_function():
        # L (Local) - 本地作用域
        local_var = "我是本地变量"
        
        # 可以访问所有作用域的变量
        print(f"Local: {local_var}")
        print(f"Enclosing: {enclosing_var}")
        print(f"Global: {global_var}")
        print(f"Built-in len函数: {len('hello')}")
    
    inner_function()

outer_function()
# 输出:
# Local: 我是本地变量
# Enclosing: 我是外层函数变量
# Global: 我是全局变量
# Built-in len函数: 5
```

#### 变量查找示例

```python
x = "全局x"

def outer():
    x = "outer的x"
    
    def inner():
        x = "inner的x"
        print(f"在inner中查找x: {x}")  # 找到本地的x
    
    inner()
    print(f"在outer中查找x: {x}")  # 找到outer的x

outer()
print(f"在全局查找x: {x}")  # 找到全局的x

# 输出:
# 在inner中查找x: inner的x
# 在outer中查找x: outer的x
# 在全局查找x: 全局x
```

### 2.2 global 和 nonlocal 关键字

#### global - 修改全局变量

默认情况下，在函数内部给变量赋值会创建一个新的本地变量，而不会修改全局变量。

```python
counter = 0  # 全局变量

def increment_wrong():
    """错误示例：创建了新的本地变量"""
    counter = counter + 1  # UnboundLocalError!

# increment_wrong()  # 报错！


def increment_correct():
    """正确示例：使用global关键字"""
    global counter
    counter = counter + 1

print(f"初始值: {counter}")
increment_correct()
print(f"调用后: {counter}")
increment_correct()
print(f"再次调用后: {counter}")
# 输出:
# 初始值: 0
# 调用后: 1
# 再次调用后: 2
```

**实际应用：配置管理**

```python
# 全局配置
APP_CONFIG = {
    'debug': False,
    'version': '1.0.0',
    'max_users': 100
}

def update_config(key, value):
    """更新全局配置"""
    global APP_CONFIG
    APP_CONFIG[key] = value
    print(f"配置已更新: {key} = {value}")

def get_config(key):
    """获取配置值"""
    return APP_CONFIG.get(key)

print(f"初始调试模式: {get_config('debug')}")
update_config('debug', True)
print(f"更新后调试模式: {get_config('debug')}")
```

#### nonlocal - 修改嵌套作用域的变量

`nonlocal`关键字用于在嵌套函数中修改外层（但非全局）函数的变量。

```python
def create_counter():
    """创建一个计数器函数"""
    count = 0  # 外层函数的变量
    
    def increment():
        nonlocal count  # 声明要修改外层的count
        count += 1
        return count
    
    def decrement():
        nonlocal count
        count -= 1
        return count
    
    def get_count():
        return count
    
    # 返回三个内部函数
    return increment, decrement, get_count

# 创建计数器
inc, dec, get = create_counter()

print(inc())  # 1
print(inc())  # 2
print(inc())  # 3
print(dec())  # 2
print(get())  # 2
```

**nonlocal vs global 对比**

```python
x = "全局"

def demo_scopes():
    x = "外层函数"
    
    def use_global():
        global x
        x = "修改全局"
        print(f"use_global中: {x}")
    
    def use_nonlocal():
        nonlocal x
        x = "修改外层"
        print(f"use_nonlocal中: {x}")
    
    def use_local():
        x = "本地"
        print(f"use_local中: {x}")
    
    print(f"初始外层: {x}")
    use_local()
    print(f"use_local后外层: {x}")  # 不变
    
    use_nonlocal()
    print(f"use_nonlocal后外层: {x}")  # 改变
    
    use_global()
    print(f"use_global后外层: {x}")  # 不变（修改的是全局）

demo_scopes()
print(f"全局变量: {x}")  # 被use_global修改了
```

### 2.3 闭包（Closures）

**什么是闭包？**

闭包是指一个函数能够记住并访问其定义时所在作用域的变量，即使这个函数在其作用域之外被调用。简单来说，闭包 = 函数 + 环境变量。

#### 闭包的基本示例

```python
def make_multiplier(factor):
    """创建一个乘法器函数"""
    def multiplier(number):
        return number * factor  # factor来自外层函数
    return multiplier

# 创建不同的乘法器
times_2 = make_multiplier(2)
times_3 = make_multiplier(3)
times_10 = make_multiplier(10)

print(times_2(5))   # 10
print(times_3(5))   # 15
print(times_10(5))  # 50

# 每个函数都"记住"了自己的factor值
```

**闭包的内部机制**

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

closure = outer(10)

# 查看闭包的自由变量
print(closure.__closure__)  # (<cell at 0x...: int object at 0x...>,)
print(closure.__closure__[0].cell_contents)  # 10

print(closure(5))  # 15
```

#### 闭包的实际应用

**1. 数据封装和隐私**

```python
def create_bank_account(initial_balance):
    """创建银行账户，余额是私有的"""
    balance = initial_balance  # 私有变量
    
    def deposit(amount):
        """存款"""
        nonlocal balance
        if amount > 0:
            balance += amount
            return f"存入{amount}元,当前余额:{balance}元"
        return "存款金额必须大于0"
    
    def withdraw(amount):
        """取款"""
        nonlocal balance
        if 0 < amount <= balance:
            balance -= amount
            return f"取出{amount}元,当前余额:{balance}元"
        return "余额不足或金额无效"
    
    def get_balance():
        """查询余额"""
        return f"当前余额:{balance}元"
    
    # 返回操作方法，但不直接暴露balance
    return deposit, withdraw, get_balance

# 创建账户
deposit, withdraw, get_balance = create_bank_account(1000)

print(get_balance())       # 当前余额:1000元
print(deposit(500))        # 存入500元,当前余额:1500元
print(withdraw(300))       # 取出300元,当前余额:1200元
print(get_balance())       # 当前余额:1200元

# 无法直接访问balance变量
# print(balance)  # NameError: name 'balance' is not defined
```

**2. 函数工厂**

```python
def make_power_function(exponent):
    """创建幂函数"""
    def power(base):
        return base ** exponent
    return power

# 创建不同的幂函数
square = make_power_function(2)      # 平方
cube = make_power_function(3)        # 立方
fourth_power = make_power_function(4)  # 四次方

print(square(5))        # 25
print(cube(3))          # 27
print(fourth_power(2))  # 16


def make_greeting(greeting_word):
    """创建问候函数"""
    def greet(name):
        return f"{greeting_word}, {name}!"
    return greet

# 创建不同语言的问候函数
hello_en = make_greeting("Hello")
hello_zh = make_greeting("你好")
hello_fr = make_greeting("Bonjour")

print(hello_en("Alice"))   # Hello, Alice!
print(hello_zh("小明"))     # 你好, 小明!
print(hello_fr("Marie"))   # Bonjour, Marie!
```

**3. 延迟绑定问题与解决**

```python
# ❌ 常见错误：循环中的闭包
def create_multipliers_wrong():
    """错误示例：所有函数共享同一个变量"""
    multipliers = []
    for i in range(5):
        def multiplier(x):
            return x * i  # i在循环结束后是4
        multipliers.append(multiplier)
    return multipliers

funcs = create_multipliers_wrong()
print([func(2) for func in funcs])  # [8, 8, 8, 8, 8] 而不是 [0, 2, 4, 6, 8]


# ✅ 解决方案1：使用默认参数
def create_multipliers_correct1():
    multipliers = []
    for i in range(5):
        def multiplier(x, factor=i):  # 默认参数在定义时绑定
            return x * factor
        multipliers.append(multiplier)
    return multipliers

funcs = create_multipliers_correct1()
print([func(2) for func in funcs])  # [0, 2, 4, 6, 8] 正确！


# ✅ 解决方案2：使用函数工厂
def create_multipliers_correct2():
    def make_multiplier(factor):
        def multiplier(x):
            return x * factor
        return multiplier
    
    return [make_multiplier(i) for i in range(5)]

funcs = create_multipliers_correct2()
print([func(2) for func in funcs])  # [0, 2, 4, 6, 8] 正确！
```

---

## 3. 装饰器详解

### 3.1 装饰器的本质

**什么是装饰器？**

装饰器是Python中一种强大的设计模式，它允许我们在不修改原函数代码的情况下，为函数添加新的功能。装饰器本质上是一个**接收函数作为参数并返回一个新函数的高阶函数**。

#### 从简单函数开始理解

```python
def simple_function():
    """一个简单的函数"""
    print("执行简单函数")

# 直接调用
simple_function()  # 输出: 执行简单函数


# 现在我们想在函数执行前后添加日志
def simple_function_with_log():
    """添加了日志的函数"""
    print("=== 开始执行 ===")
    print("执行简单函数")
    print("=== 执行完毕 ===")

simple_function_with_log()
# 输出:
# === 开始执行 ===
# 执行简单函数
# === 执行完毕 ===
```

但是如果有很多函数都需要添加日志呢？一个个修改太麻烦了！这就是装饰器的用武之地。

#### 手动装饰

```python
def add_logging(func):
    """装饰器函数：添加日志功能"""
    def wrapper():
        print("=== 开始执行 ===")
        func()  # 调用原函数
        print("=== 执行完毕 ===")
    return wrapper

def my_function():
    print("执行我的函数")

# 手动装饰
my_function = add_logging(my_function)
my_function()
# 输出:
# === 开始执行 ===
# 执行我的函数
# === 执行完毕 ===
```

#### 使用@语法糖

```python
def add_logging(func):
    """装饰器函数"""
    def wrapper():
        print("=== 开始执行 ===")
        func()
        print("=== 执行完毕 ===")
    return wrapper

# 使用@语法糖，等价于: my_function = add_logging(my_function)
@add_logging
def my_function():
    print("执行我的函数")

my_function()
# 输出:
# === 开始执行 ===
# 执行我的函数
# === 执行完毕 ===
```

### 3.2 带参数的装饰器

上面的装饰器只能装饰无参数的函数。如果被装饰的函数有参数怎么办？

#### 处理任意参数

```python
def add_logging(func):
    """通用装饰器：支持任意参数"""
    def wrapper(*args, **kwargs):
        print(f"=== 调用 {func.__name__} ===")
        print(f"  位置参数: {args}")
        print(f"  关键字参数: {kwargs}")
        result = func(*args, **kwargs)
        print(f"  返回值: {result}")
        print("=== 执行完毕 ===")
        return result
    return wrapper

@add_logging
def add(a, b):
    """加法函数"""
    return a + b

@add_logging
def greet(name, greeting="Hello"):
    """问候函数"""
    return f"{greeting}, {name}!"

print(add(3, 5))
print()
print(greet("Alice", greeting="Hi"))
# 输出:
# === 调用 add ===
#   位置参数: (3, 5)
#   关键字参数: {}
#   返回值: 8
# === 执行完毕 ===
# 8
# 
# === 调用 greet ===
#   位置参数: ('Alice',)
#   关键字参数: {'greeting': 'Hi'}
#   返回值: Hi, Alice!
# === 执行完毕 ===
# Hi, Alice!
```

#### 保留函数元数据

装饰后的函数会丢失原函数的元数据（名称、文档字符串等）：

```python
def simple_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@simple_decorator
def my_function():
    """这是我的函数"""
    pass

print(my_function.__name__)  # wrapper (错误！)
print(my_function.__doc__)   # None (文档丢失！)
```

**解决方案：使用functools.wraps**

```python
from functools import wraps

def better_decorator(func):
    @wraps(func)  # 保留原函数的元数据
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@better_decorator
def my_function():
    """这是我的函数"""
    pass

print(my_function.__name__)  # my_function (正确！)
print(my_function.__doc__)   # 这是我的函数 (正确！)
```

### 3.3 装饰器的实际应用

#### 1. 计时装饰器

```python
import time
from functools import wraps

def timer(func):
    """计算函数执行时间"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} 执行时间: {end_time - start_time:.4f}秒")
        return result
    return wrapper

@timer
def slow_function():
    """模拟耗时操作"""
    time.sleep(1)
    return "完成"

@timer
def calculate_sum(n):
    """计算1到n的和"""
    return sum(range(n + 1))

result = slow_function()
print(f"结果: {result}\n")

result = calculate_sum(1000000)
print(f"结果: {result}")
```

#### 2. 缓存装饰器

```python
from functools import wraps

def cache(func):
    """简单的缓存装饰器"""
    cached_results = {}
    
    @wraps(func)
    def wrapper(*args):
        if args in cached_results:
            print(f"从缓存获取结果: {args}")
            return cached_results[args]
        
        print(f"计算结果: {args}")
        result = func(*args)
        cached_results[args] = result
        return result
    
    return wrapper

@cache
def fibonacci(n):
    """计算斐波那契数列"""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))  # 第一次计算
print(fibonacci(10))  # 从缓存获取
print(fibonacci(5))   # 部分从缓存获取


# Python内置的缓存装饰器
from functools import lru_cache

@lru_cache(maxsize=128)  # 最多缓存128个结果
def fibonacci_cached(n):
    """使用内置缓存的斐波那契"""
    if n < 2:
        return n
    return fibonacci_cached(n - 1) + fibonacci_cached(n - 2)

print(fibonacci_cached(100))  # 很快就能计算出来
```

#### 3. 权限检查装饰器

```python
from functools import wraps

# 模拟当前用户
current_user = {'username': 'alice', 'role': 'admin'}

def require_auth(required_role):
    """检查用户权限的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if current_user.get('role') != required_role:
                return f"权限不足! 需要 {required_role} 角色"
            return func(*args, **kwargs)
        return wrapper
    return decorator

@require_auth('admin')
def delete_user(username):
    """删除用户（需要admin权限）"""
    return f"用户 {username} 已删除"

@require_auth('user')
def view_profile():
    """查看个人资料（普通用户权限）"""
    return "个人资料信息"

print(delete_user('bob'))   # 用户 bob 已删除
current_user['role'] = 'user'
print(delete_user('bob'))   # 权限不足! 需要 admin 角色
print(view_profile())       # 个人资料信息
```

#### 4. 重试装饰器

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    """重试装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    print(f"第 {attempt} 次尝试...")
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"失败: {e}")
                    if attempt == max_attempts:
                        print("达到最大重试次数，放弃")
                        raise
                    print(f"等待 {delay} 秒后重试...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=2)
def unreliable_function():
    """模拟不稳定的函数"""
    import random
    if random.random() < 0.7:  # 70%的概率失败
        raise Exception("随机错误")
    return "成功!"

# 多次运行可能看到不同的重试次数
try:
    result = unreliable_function()
    print(f"最终结果: {result}")
except Exception as e:
    print(f"最终失败: {e}")
```

### 3.4 类装饰器

除了函数，我们也可以用类来实现装饰器。

```python
from functools import wraps

class CountCalls:
    """统计函数调用次数的类装饰器"""
    def __init__(self, func):
        wraps(func)(self)  # 保留元数据
        self.func = func
        self.count = 0
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"{self.func.__name__} 被调用了 {self.count} 次")
        return self.func(*args, **kwargs)

@CountCalls
def say_hello(name):
    return f"Hello, {name}!"

print(say_hello("Alice"))
print(say_hello("Bob"))
print(say_hello("Charlie"))
# 输出:
# say_hello 被调用了 1 次
# Hello, Alice!
# say_hello 被调用了 2 次
# Hello, Bob!
# say_hello 被调用了 3 次
# Hello, Charlie!


# 查看调用次数
print(f"\n总共调用了 {say_hello.count} 次")
```

### 3.5 多个装饰器的叠加

可以对同一个函数应用多个装饰器。

```python
from functools import wraps
import time

def bold(func):
    """加粗装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    """斜体装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

def timer(func):
    """计时装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"执行时间: {end - start:.4f}秒")
        return result
    return wrapper

# 多个装饰器从下往上应用
# 相当于: greet = timer(bold(italic(greet)))
@timer
@bold
@italic
def greet(name):
    """问候函数"""
    return f"Hello, {name}"

result = greet("Alice")
print(result)
# 输出:
# 执行时间: 0.0000秒
# <b><i>Hello, Alice</i></b>
```

---

## 4. 高阶函数与Lambda表达式

### 4.1 高阶函数的概念

**什么是高阶函数？**

高阶函数是指满足以下至少一个条件的函数：
1. 接受一个或多个函数作为参数
2. 返回一个函数作为结果

高阶函数是函数式编程的核心概念，它让代码更加灵活和可复用。

#### 函数作为参数

```python
def apply_operation(x, y, operation):
    """接受一个操作函数作为参数"""
    return operation(x, y)

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

# 传入不同的函数
print(apply_operation(5, 3, add))       # 8
print(apply_operation(5, 3, multiply))  # 15


# 实际应用：排序
students = [
    {'name': '张三', 'score': 85},
    {'name': '李四', 'score': 92},
    {'name': '王五', 'score': 78}
]

# 按成绩排序
sorted_by_score = sorted(students, key=lambda s: s['score'])
print(sorted_by_score)

# 按姓名排序
sorted_by_name = sorted(students, key=lambda s: s['name'])
print(sorted_by_name)
```

#### 函数作为返回值

```python
def create_adder(x):
    """返回一个加法函数"""
    def adder(y):
        return x + y
    return adder

# 创建不同的加法器
add_5 = create_adder(5)
add_10 = create_adder(10)

print(add_5(3))   # 8
print(add_10(3))  # 13
```

### 4.2 内置高阶函数

Python提供了几个非常有用的内置高阶函数。

#### map() - 映射

`map()`函数对可迭代对象的每个元素应用一个函数。

```python
# 基本用法
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]


# 实际应用：数据转换
prices_str = ['10.5', '20.3', '15.8', '30.2']
prices_float = list(map(float, prices_str))
print(prices_float)  # [10.5, 20.3, 15.8, 30.2]


# 多个可迭代对象
numbers1 = [1, 2, 3, 4]
numbers2 = [10, 20, 30, 40]
result = list(map(lambda x, y: x + y, numbers1, numbers2))
print(result)  # [11, 22, 33, 44]


# 字符串处理
names = ['alice', 'bob', 'charlie']
capitalized = list(map(str.capitalize, names))
print(capitalized)  # ['Alice', 'Bob', 'Charlie']
```

#### filter() - 过滤

`filter()`函数根据条件过滤可迭代对象的元素。

```python
# 基本用法
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # [2, 4, 6, 8, 10]


# 实际应用：数据筛选
students = [
    {'name': '张三', 'score': 85},
    {'name': '李四', 'score': 92},
    {'name': '王五', 'score': 78},
    {'name': '赵六', 'score': 95}
]

# 筛选出成绩大于80分的学生
high_scorers = list(filter(lambda s: s['score'] > 80, students))
print(high_scorers)


# 过滤空字符串和None
data = ['apple', '', 'banana', None, 'cherry', '', 'date']
filtered = list(filter(None, data))  # None会过滤掉假值
print(filtered)  # ['apple', 'banana', 'cherry', 'date']
```

#### reduce() - 归约

`reduce()`函数将一个函数累积地应用到序列的元素上，从左到右，将序列归约为单个值。

```python
from functools import reduce

# 基本用法：计算累加和
numbers = [1, 2, 3, 4, 5]
sum_result = reduce(lambda x, y: x + y, numbers)
print(sum_result)  # 15

# 等价于: ((((1 + 2) + 3) + 4) + 5)


# 计算累乘积
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120


# 带初始值
sum_with_initial = reduce(lambda x, y: x + y, numbers, 100)
print(sum_with_initial)  # 115 (100 + 15)


# 实际应用：找最大值
numbers = [45, 23, 78, 12, 89, 34]
max_value = reduce(lambda x, y: x if x > y else y, numbers)
print(max_value)  # 89


# 合并字典列表
dict_list = [
    {'a': 1, 'b': 2},
    {'c': 3, 'd': 4},
    {'e': 5}
]
merged = reduce(lambda x, y: {**x, **y}, dict_list)
print(merged)  # {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5}
```

#### sorted() 与 key参数

```python
# 复杂排序示例
data = [
    ('apple', 5, 10.5),
    ('banana', 3, 8.2),
    ('cherry', 8, 15.3),
    ('date', 2, 12.1)
]

# 按数量排序（第二个元素）
sorted_by_quantity = sorted(data, key=lambda x: x[1])
print(sorted_by_quantity)

# 按价格降序排序（第三个元素）
sorted_by_price = sorted(data, key=lambda x: x[2], reverse=True)
print(sorted_by_price)

# 多级排序：先按数量，再按价格
sorted_multi = sorted(data, key=lambda x: (x[1], x[2]))
print(sorted_multi)
```

### 4.3 Lambda表达式

**什么是Lambda表达式？**

Lambda是创建匿名函数的简洁方式。它适合用于简单的、一次性使用的函数。

语法：`lambda 参数: 表达式`

#### Lambda的基本用法

```python
# 普通函数
def add(x, y):
    return x + y

# 等价的lambda表达式
add_lambda = lambda x, y: x + y

print(add(3, 5))        # 8
print(add_lambda(3, 5)) # 8


# 单参数lambda
square = lambda x: x ** 2
print(square(5))  # 25


# 无参数lambda
get_pi = lambda: 3.14159
print(get_pi())  # 3.14159
```

#### Lambda的实际应用

```python
# 1. 在排序中使用
points = [(1, 2), (3, 1), (5, 0), (2, 4)]
sorted_points = sorted(points, key=lambda p: p[1])
print(sorted_points)  # [(5, 0), (3, 1), (1, 2), (2, 4)]


# 2. 在map中使用
celsius = [0, 10, 20, 30, 40]
fahrenheit = list(map(lambda c: c * 9/5 + 32, celsius))
print(fahrenheit)  # [32.0, 50.0, 68.0, 86.0, 104.0]


# 3. 在filter中使用
words = ['apple', 'banana', 'cherry', 'date', 'elderberry']
short_words = list(filter(lambda w: len(w) <= 5, words))
print(short_words)  # ['apple', 'date']


# 4. 条件表达式
max_func = lambda x, y: x if x > y else y
print(max_func(10, 20))  # 20


# 5. 立即执行的lambda（IIFE）
result = (lambda x, y: x + y)(5, 3)
print(result)  # 8
```

#### Lambda vs 普通函数

```python
# ✅ Lambda适合的场景：简单、一次性使用
numbers = [1, 2, 3, 4, 5]
doubled = list(map(lambda x: x * 2, numbers))


# ❌ Lambda不适合的场景：复杂逻辑
# 错误示例：lambda中包含复杂逻辑（不推荐）
# complex_lambda = lambda x: print(x) if x > 0 else print(-x) # 语法错误！

# ✅ 应该使用普通函数
def process_number(x):
    """处理数字的复杂逻辑"""
    if x > 0:
        result = x ** 2
        print(f"正数的平方: {result}")
        return result
    else:
        result = abs(x)
        print(f"负数的绝对值: {result}")
        return result
```

### 4.4 函数式编程实例

#### 链式调用

```python
from functools import reduce

# 数据处理管道
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 找出所有偶数，平方后求和
result = reduce(
    lambda x, y: x + y,                    # 求和
    map(lambda x: x ** 2,                  # 平方
        filter(lambda x: x % 2 == 0,       # 过滤偶数
               numbers)))
print(result)  # 220 (4 + 16 + 36 + 64 + 100)


# 更易读的写法：分步骤
even_numbers = filter(lambda x: x % 2 == 0, numbers)
squared = map(lambda x: x ** 2, even_numbers)
total = reduce(lambda x, y: x + y, squared)
print(total)  # 220
```

#### 函数组合

```python
def compose(*functions):
    """组合多个函数"""
    def inner(arg):
        result = arg
        for func in reversed(functions):
            result = func(result)
        return result
    return inner

# 定义一些简单函数
def add_10(x):
    return x + 10

def multiply_2(x):
    return x * 2

def subtract_5(x):
    return x - 5

# 组合函数
combined = compose(add_10, multiply_2, subtract_5)
print(combined(5))  # ((5 - 5) * 2) + 10 = 10


# 实际应用：文本处理管道
def remove_spaces(text):
    return text.replace(' ', '')

def to_uppercase(text):
    return text.upper()

def add_exclamation(text):
    return text + '!!!'

process_text = compose(add_exclamation, to_uppercase, remove_spaces)
result = process_text("hello world")
print(result)  # HELLOWORLD!!!
```

---

## 5. 生成器与迭代器

### 5.1 迭代器协议

**什么是迭代器？**

迭代器是一个可以记住遍历位置的对象。它实现了两个方法：
- `__iter__()`: 返回迭代器对象自身
- `__next__()`: 返回下一个值，没有更多元素时抛出`StopIteration`

```python
# 手动实现一个迭代器
class CountDown:
    """倒数计数器"""
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# 使用迭代器
counter = CountDown(5)
for num in counter:
    print(num)  # 5, 4, 3, 2, 1


# 手动迭代
counter = CountDown(3)
print(next(counter))  # 3
print(next(counter))  # 2
print(next(counter))  # 1
# print(next(counter))  # StopIteration异常
```

**可迭代对象vs迭代器**

```python
# 列表是可迭代对象，但不是迭代器
my_list = [1, 2, 3]
print(hasattr(my_list, '__iter__'))  # True（可迭代）
print(hasattr(my_list, '__next__'))  # False（不是迭代器）

# 获取迭代器
iterator = iter(my_list)
print(hasattr(iterator, '__iter__'))  # True
print(hasattr(iterator, '__next__'))  # True

print(next(iterator))  # 1
print(next(iterator))  # 2
print(next(iterator))  # 3
```

### 5.2 生成器（Generators）

**什么是生成器？**

生成器是创建迭代器的简单而强大的工具。它使用`yield`关键字，每次调用时返回一个值，并保存函数的状态。

#### 生成器函数

```python
def simple_generator():
    """简单的生成器示例"""
    print("开始")
    yield 1
    print("继续")
    yield 2
    print("再继续")
    yield 3
    print("结束")

# 创建生成器对象
gen = simple_generator()
print(type(gen))  # <class 'generator'>

# 逐个获取值
print(next(gen))  # 开始 \n 1
print(next(gen))  # 继续 \n 2
print(next(gen))  # 再继续 \n 3
# print(next(gen))  # 结束 \n StopIteration
```

#### 实用的生成器示例

```python
def fibonacci(n):
    """生成斐波那契数列的前n个数"""
    a, b = 0, 1
    count = 0
    while count < n:
        yield a
        a, b = b, a + b
        count += 1

# 使用生成器
for num in fibonacci(10):
    print(num, end=' ')  # 0 1 1 2 3 5 8 13 21 34
print()


def read_large_file(file_path):
    """逐行读取大文件（避免一次性加载到内存）"""
    with open(file_path, 'r', encoding='utf-8') as f:
        for line in f:
            yield line.strip()

# 使用
# for line in read_large_file('huge_file.txt'):
#     process(line)  # 每次只处理一行


def infinite_sequence():
    """无限序列生成器"""
    num = 0
    while True:
        yield num
        num += 1

# 使用（配合其他机制终止）
gen = infinite_sequence()
for i in range(5):
    print(next(gen))  # 0, 1, 2, 3, 4
```

#### 生成器表达式

生成器表达式类似列表推导式，但使用圆括号，并且是惰性求值。

```python
# 列表推导式：立即创建整个列表
list_comp = [x ** 2 for x in range(10)]
print(type(list_comp))  # <class 'list'>
print(list_comp)        # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 生成器表达式：惰性求值
gen_exp = (x ** 2 for x in range(10))
print(type(gen_exp))    # <class 'generator'>
print(list(gen_exp))    # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]


# 内存效率对比
import sys

list_obj = [x for x in range(10000)]
gen_obj = (x for x in range(10000))

print(f"列表大小: {sys.getsizeof(list_obj)} 字节")      # 约 87616 字节
print(f"生成器大小: {sys.getsizeof(gen_obj)} 字节")    # 约 128 字节


# 实际应用：处理大数据
def process_large_dataset():
    """处理大数据集"""
    # 不好的做法：一次性加载
    # data = [expensive_operation(i) for i in range(1000000)]
    
    # 好的做法：使用生成器
    data = (expensive_operation(i) for i in range(1000000))
    return data

def expensive_operation(x):
    return x * 2
```

### 5.3 yield的高级用法

#### yield from

`yield from`用于委托给另一个生成器。

```python
def generator1():
    yield 1
    yield 2

def generator2():
    yield 3
    yield 4

def combined_wrong():
    """错误示例"""
    yield generator1()  # 这会yield生成器对象本身
    yield generator2()

def combined_correct():
    """正确示例"""
    yield from generator1()
    yield from generator2()

print(list(combined_correct()))  # [1, 2, 3, 4]


# 实际应用：树的遍历
def flatten(nested_list):
    """展平嵌套列表"""
    for item in nested_list:
        if isinstance(item, list):
            yield from flatten(item)  # 递归展平
        else:
            yield item

nested = [1, [2, 3, [4, 5]], 6, [7, [8, 9]]]
flat = list(flatten(nested))
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 生成器的send()方法

生成器不仅可以产生值，还可以接收值。

```python
def echo_generator():
    """回声生成器"""
    while True:
        received = yield
        print(f"收到: {received}")

gen = echo_generator()
next(gen)  # 启动生成器
gen.send("Hello")   # 收到: Hello
gen.send("World")   # 收到: World


# 更复杂的示例：可控制的计数器
def controllable_counter(start=0):
    """可控制的计数器"""
    current = start
    while True:
        change = yield current
        if change is not None:
            current += change
        else:
            current += 1

counter = controllable_counter(0)
print(next(counter))          # 0
print(counter.send(5))        # 5
print(counter.send(10))       # 15
print(next(counter))          # 16
```

---

## 6. 递归与高级技巧

### 6.1 递归函数

**什么是递归？**

递归是函数调用自身的过程。递归函数必须有：
1. **基准条件（Base Case）**：停止递归的条件
2. **递归条件（Recursive Case）**：函数调用自身

#### 递归的基本示例

```python
def countdown(n):
    """递归倒数"""
    if n <= 0:  # 基准条件
        print("发射！")
    else:
        print(n)
        countdown(n - 1)  # 递归调用

countdown(5)
# 输出:
# 5
# 4
# 3
# 2
# 1
# 发射！


# 经典示例：阶乘
def factorial(n):
    """计算n的阶乘"""
    if n <= 1:  # 基准条件
        return 1
    return n * factorial(n - 1)  # 递归条件

print(factorial(5))  # 120 (5 * 4 * 3 * 2 * 1)


# 斐波那契数列（递归实现）
def fibonacci_recursive(n):
    """递归计算斐波那契数"""
    if n <= 1:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)

print(fibonacci_recursive(10))  # 55
# 注意：这个实现效率很低，会重复计算很多值
```

#### 递归的可视化理解

```python
def factorial_verbose(n, depth=0):
    """带可视化的阶乘函数"""
    indent = "  " * depth
    print(f"{indent}factorial({n}) 被调用")
    
    if n <= 1:
        print(f"{indent}返回 1")
        return 1
    
    result = n * factorial_verbose(n - 1, depth + 1)
    print(f"{indent}返回 {n} * factorial({n-1}) = {result}")
    return result

factorial_verbose(4)
# 输出展示了递归调用的层次结构
```

### 6.2 递归的实际应用

#### 1. 树形结构遍历

```python
# 文件系统遍历示例
def list_files(directory, prefix=""):
    """递归列出目录下的所有文件"""
    import os
    
    try:
        items = os.listdir(directory)
        for item in items:
            path = os.path.join(directory, item)
            print(f"{prefix}{item}")
            
            if os.path.isdir(path):
                list_files(path, prefix + "  ")  # 递归遍历子目录
    except PermissionError:
        print(f"{prefix}[权限不足]")

# list_files("./my_project")


# JSON数据遍历
def print_json(data, indent=0):
    """递归打印JSON结构"""
    prefix = "  " * indent
    
    if isinstance(data, dict):
        for key, value in data.items():
            print(f"{prefix}{key}:")
            print_json(value, indent + 1)
    elif isinstance(data, list):
        for i, item in enumerate(data):
            print(f"{prefix}[{i}]:")
            print_json(item, indent + 1)
    else:
        print(f"{prefix}{data}")

json_data = {
    'name': '张三',
    'age': 25,
    'skills': ['Python', 'JavaScript'],
    'address': {
        'city': '北京',
        'district': '海淀'
    }
}

print_json(json_data)
```

#### 2. 分治算法

```python
def quick_sort(arr):
    """快速排序（递归实现）"""
    if len(arr) <= 1:  # 基准条件
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    # 递归排序左右两部分
    return quick_sort(left) + middle + quick_sort(right)

numbers = [64, 34, 25, 12, 22, 11, 90]
sorted_numbers = quick_sort(numbers)
print(sorted_numbers)  # [11, 12, 22, 25, 34, 64, 90]


# 归并排序
def merge_sort(arr):
    """归并排序（递归实现）"""
    if len(arr) <= 1:
        return arr
    
    # 分割
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    # 合并
    return merge(left, right)

def merge(left, right):
    """合并两个有序数组"""
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result

numbers = [64, 34, 25, 12, 22, 11, 90]
sorted_numbers = merge_sort(numbers)
print(sorted_numbers)  # [11, 12, 22, 25, 34, 64, 90]
```

#### 3. 动态规划问题

```python
# 使用记忆化避免重复计算
def fibonacci_memo(n, memo={}):
    """带记忆化的斐波那契"""
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    
    memo[n] = fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo)
    return memo[n]

print(fibonacci_memo(100))  # 很快就能计算出来


# 使用装饰器实现记忆化
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci_cached(n):
    """使用lru_cache的斐波那契"""
    if n <= 1:
        return n
    return fibonacci_cached(n - 1) + fibonacci_cached(n - 2)

print(fibonacci_cached(100))  # 354224848179261915075
```

### 6.3 尾递归优化

**什么是尾递归？**

尾递归是指递归调用是函数的最后一个操作。Python不支持尾递归优化，但我们可以手动改写。

```python
# 普通递归（非尾递归）
def factorial_normal(n):
    if n <= 1:
        return 1
    return n * factorial_normal(n - 1)  # 递归后还有乘法操作


# 尾递归版本
def factorial_tail(n, accumulator=1):
    """尾递归版本的阶乘"""
    if n <= 1:
        return accumulator
    return factorial_tail(n - 1, n * accumulator)  # 递归调用是最后的操作

print(factorial_tail(5))  # 120


# 将递归转换为循环（推荐）
def factorial_iterative(n):
    """迭代版本的阶乘"""
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result

print(factorial_iterative(5))  # 120
```

### 6.4 函数的高级特性

#### 1. 函数注解（Type Hints）

Python 3.5+支持类型注解，提高代码可读性和IDE支持。

```python
def greet(name: str, age: int) -> str:
    """
    问候函数
    
    参数:
        name: 用户名
        age: 用户年龄
    
    返回:
        问候语字符串
    """
    return f"Hello, {name}! You are {age} years old."

# 类型注解不会强制类型检查，但IDE会给出提示
result = greet("Alice", 25)
print(result)


# 复杂类型注解
from typing import List, Dict, Tuple, Optional, Union, Callable

def process_data(
    items: List[int],
    config: Dict[str, str],
    callback: Callable[[int], int]
) -> Tuple[List[int], int]:
    """处理数据"""
    processed = [callback(item) for item in items]
    total = sum(processed)
    return processed, total


# Optional表示可以是None
def find_user(user_id: int) -> Optional[Dict[str, str]]:
    """查找用户，可能返回None"""
    if user_id > 0:
        return {'id': str(user_id), 'name': 'User'}
    return None


# Union表示多种类型
def format_value(value: Union[int, float, str]) -> str:
    """格式化值"""
    return str(value)
```

#### 2. 函数的内省

```python
import inspect

def example_function(a: int, b: str = "default", *args, **kwargs) -> str:
    """示例函数"""
    return f"{a} {b}"

# 获取函数签名
sig = inspect.signature(example_function)
print(f"函数签名: {sig}")

# 获取参数信息
for param_name, param in sig.parameters.items():
    print(f"参数: {param_name}")
    print(f"  类型注解: {param.annotation}")
    print(f"  默认值: {param.default}")
    print(f"  类型: {param.kind}")

# 获取返回值类型
print(f"返回值类型: {sig.return_annotation}")


# 获取函数的源代码
source = inspect.getsource(example_function)
print("源代码:")
print(source)
```

#### 3. 偏函数（Partial Functions）

```python
from functools import partial

# 原始函数
def power(base, exponent):
    """计算幂"""
    return base ** exponent

# 创建偏函数：固定指数为2（平方）
square = partial(power, exponent=2)
print(square(5))  # 25

# 创建偏函数：固定指数为3（立方）
cube = partial(power, exponent=3)
print(cube(5))  # 125


# 实际应用：日志记录
import logging

def log_message(message, level='INFO'):
    """记录日志"""
    print(f"[{level}] {message}")

# 创建不同级别的日志函数
log_info = partial(log_message, level='INFO')
log_error = partial(log_message, level='ERROR')
log_warning = partial(log_message, level='WARNING')

log_info("系统启动")
log_error("发生错误")
log_warning("警告信息")
```

#### 4. 单分派泛型函数

```python
from functools import singledispatch

@singledispatch
def process(data):
    """默认处理函数"""
    return f"处理未知类型: {type(data)}"

@process.register(int)
def _(data):
    """处理整数"""
    return f"整数: {data * 2}"

@process.register(str)
def _(data):
    """处理字符串"""
    return f"字符串: {data.upper()}"

@process.register(list)
def _(data):
    """处理列表"""
    return f"列表长度: {len(data)}"

@process.register(dict)
def _(data):
    """处理字典"""
    return f"字典键: {list(data.keys())}"

# 根据参数类型自动调用对应的实现
print(process(42))              # 整数: 84
print(process("hello"))         # 字符串: HELLO
print(process([1, 2, 3]))       # 列表长度: 3
print(process({'a': 1}))        # 字典键: ['a']
print(process(3.14))            # 处理未知类型: <class 'float'>
```

### 6.5 性能优化技巧

#### 1. 使用生成器代替列表

```python
import sys

# 不好的做法：创建大列表
def get_squares_list(n):
    return [x ** 2 for x in range(n)]

# 好的做法：使用生成器
def get_squares_generator(n):
    return (x ** 2 for x in range(n))

# 内存对比
n = 1000000
list_obj = get_squares_list(n)
gen_obj = get_squares_generator(n)

print(f"列表内存: {sys.getsizeof(list_obj) / 1024 / 1024:.2f} MB")
print(f"生成器内存: {sys.getsizeof(gen_obj) / 1024:.2f} KB")
```

#### 2. 避免重复计算

```python
import time

# 不好的做法：重复计算
def process_bad(items):
    result = []
    for item in items:
        if len(items) > 100:  # len(items)被重复计算
            result.append(item * 2)
    return result

# 好的做法：缓存结果
def process_good(items):
    result = []
    items_length = len(items)  # 只计算一次
    for item in items:
        if items_length > 100:
            result.append(item * 2)
    return result
```

#### 3. 使用内置函数

```python
# 不好的做法：手动实现
def sum_list_bad(numbers):
    total = 0
    for num in numbers:
        total += num
    return total

# 好的做法：使用内置函数
def sum_list_good(numbers):
    return sum(numbers)

# 内置函数通常用C实现，速度更快
```

---

## 总结

本教程深入讲解了Python函数的高级特性：

### 核心概念回顾

1. **函数参数**
   - 位置参数、关键字参数、默认参数
   - `*args`和`**kwargs`可变参数
   - 仅限位置/关键字参数
   - 参数解包

2. **作用域与闭包**
   - LEGB规则（Local, Enclosing, Global, Built-in）
   - `global`和`nonlocal`关键字
   - 闭包的概念和应用

3. **装饰器**
   - 装饰器的本质：函数包装函数
   - 带参数的装饰器
   - 类装饰器
   - 实际应用：日志、缓存、权限检查等

4. **高阶函数与Lambda**
   - 函数作为一等公民
   - `map()`、`filter()`、`reduce()`
   - Lambda表达式的适用场景
   - 函数式编程思想

5. **生成器与迭代器**
   - 迭代器协议
   - `yield`关键字
   - 生成器表达式
   - `yield from`和`send()`

6. **递归与高级技巧**
   - 递归的基准条件和递归条件
   - 分治算法
   - 记忆化优化
   - 类型注解、函数内省、偏函数

### 最佳实践

✅ **推荐做法**
- 使用类型注解提高代码可读性
- 使用装饰器复用横切关注点（日志、缓存等）
- 用生成器处理大数据集
- 避免可变对象作为默认参数
- 使用`functools.wraps`保留函数元数据
- 递归时考虑栈溢出风险

❌ **避免做法**
- 过度使用Lambda（复杂逻辑应使用普通函数）
- 深层递归没有优化（考虑改用循环）
- 忽略函数的副作用
- 全局变量的滥用

### 进阶学习建议

1. 深入学习函数式编程思想
2. 了解异步函数（`async/await`）
3. 学习更多设计模式
4. 研究CPython源码，了解函数的底层实现
5. 实践项目中灵活运用这些技巧

---

**祝你在Python编程之路上越走越远！🚀**

