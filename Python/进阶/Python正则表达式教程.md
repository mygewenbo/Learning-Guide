# Python正则表达式完全教程

> 本教程详细讲解Python正则表达式的核心概念、语法规则和实战应用，基于Python 3.13+编写

---

## 目录

1. [正则表达式基础](#1-正则表达式基础)
2. [元字符和特殊序列](#2-元字符和特殊序列)
3. [re模块详解](#3-re模块详解)
4. [分组和捕获](#4-分组和捕获)
5. [高级应用](#5-高级应用)
6. [实战案例](#6-实战案例)

---

## 1. 正则表达式基础

### 1.1 什么是正则表达式？

**正则表达式（Regular Expression，简称regex或regexp）**是一种用来描述字符串模式的强大工具。它就像是一个"搜索模式"，可以帮助我们：

- **查找**：在文本中找到符合特定模式的内容
- **匹配**：验证字符串是否符合某种格式（如邮箱、手机号）
- **替换**：批量修改文本中的内容
- **分割**：按照特定模式分割字符串

**生活中的类比：**
- 正则表达式就像"超级搜索器"，可以用通配符和规则来搜索
- 比如：找所有以"张"开头的姓名，找所有139开头的手机号
- 类似于Excel中的筛选功能，但更加强大和灵活

### 1.2 为什么需要正则表达式？

让我们先看一个例子，感受正则表达式的威力：

**任务：从一段文本中提取所有的电子邮件地址**

```python
text = """
联系我们：
销售部：sales@company.com
技术支持：support@company.com
人事部：hr@company.cn
"""

# 不使用正则表达式（笨方法）
emails = []
words = text.split()
for word in words:
    if '@' in word and '.' in word:
        emails.append(word)
print(emails)  # 可能包含不完整的邮箱


# 使用正则表达式（优雅方法）
import re
emails = re.findall(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', text)
print(emails)  # ['sales@company.com', 'support@company.com', 'hr@company.cn']
```

**正则表达式的优势：**
- ✅ 代码简洁：一行代码完成复杂的文本处理
- ✅ 准确性高：精确定义匹配规则
- ✅ 性能好：底层经过优化
- ✅ 通用性强：几乎所有编程语言都支持

### 1.3 Python中的正则表达式模块

Python通过内置的`re`模块提供正则表达式支持。

```python
import re  # 导入正则表达式模块

# 最简单的示例：查找是否包含某个词
text = "Python is powerful"
result = re.search('Python', text)
if result:
    print("找到了Python！")  # 输出：找到了Python！
```

### 1.4 第一个正则表达式

#### 普通字符匹配

最简单的正则表达式就是普通字符本身。

```python
import re

text = "Hello, World!"

# 查找"World"
match = re.search('World', text)
if match:
    print(f"找到了：{match.group()}")  # 输出：找到了：World
    print(f"位置：{match.span()}")     # 输出：位置：(7, 12)

# 查找"Python"
match = re.search('Python', text)
if match:
    print("找到了Python")
else:
    print("没找到Python")  # 输出：没找到Python
```

#### 原始字符串（Raw String）

**重要概念：**在Python中，正则表达式通常使用**原始字符串**（在字符串前加`r`）。

**为什么需要原始字符串？**

```python
# 不使用原始字符串
pattern1 = '\d+'  # \d会被Python解释器处理，可能不是你想要的
print(repr(pattern1))  # '\\d+'

# 使用原始字符串（推荐）
pattern2 = r'\d+'  # \d保持原样，作为正则表达式的一部分
print(repr(pattern2))  # '\\d+'

# 实际使用
text = "我有123个苹果"
print(re.findall('\d+', text))   # 可能有问题
print(re.findall(r'\d+', text))  # ['123'] 正确！
```

**规则：编写正则表达式时，始终使用原始字符串 `r''`**

### 1.5 匹配的基本概念

#### 匹配成功与失败

```python
import re

text = "今天是2024年11月19日"

# 匹配成功：返回Match对象
match1 = re.search(r'\d+', text)
print(match1)  # <re.Match object; span=(3, 7), match='2024'>
print(match1.group())  # 2024

# 匹配失败：返回None
match2 = re.search(r'xyz', text)
print(match2)  # None

# 安全的使用方式
if match2:
    print(match2.group())
else:
    print("没有匹配到")  # 输出：没有匹配到
```

#### 完整匹配 vs 部分匹配

```python
import re

pattern = r'cat'
text1 = "cat"
text2 = "category"
text3 = "dog"

# search：在字符串中查找（部分匹配）
print(re.search(pattern, text1))  # 匹配
print(re.search(pattern, text2))  # 也匹配（找到了cat）
print(re.search(pattern, text3))  # 不匹配

# fullmatch：整个字符串必须完全匹配
print(re.fullmatch(pattern, text1))  # 匹配
print(re.fullmatch(pattern, text2))  # 不匹配（有额外的字符）
print(re.fullmatch(pattern, text3))  # 不匹配
```

---

## 2. 元字符和特殊序列

### 2.1 点号（.）- 匹配任意字符

**点号`.`是最基本的元字符，它可以匹配除换行符外的任意单个字符。**

```python
import re

# . 匹配任意字符
text = "cat bat rat"
matches = re.findall(r'.at', text)
print(matches)  # ['cat', 'bat', 'rat']

# . 匹配数字、字母、符号
text2 = "h1t h@t h t"
matches2 = re.findall(r'h.t', text2)
print(matches2)  # ['h1t', 'h@t', 'h t']

# . 不匹配换行符（默认）
text3 = "hello\nworld"
match3 = re.search(r'hello.world', text3)
print(match3)  # None

# 使用DOTALL标志让.匹配换行符
match4 = re.search(r'hello.world', text3, re.DOTALL)
print(match4)  # <re.Match object...>
print(match4.group())  # hello\nworld
```

**记忆技巧：**`.`就像一个"占位符"，表示"这里有一个字符，但我不关心它是什么"

### 2.2 字符类 [] - 匹配一组字符

**方括号`[]`定义一个字符类，匹配其中的任意一个字符。**

```python
import re

# [abc] 匹配a、b或c
text = "apple banana cherry"
matches = re.findall(r'[abc]', text)
print(matches)  # ['a', 'a', 'b', 'a', 'a', 'a', 'c']

# [a-z] 匹配小写字母
matches2 = re.findall(r'[a-z]+', text)
print(matches2)  # ['apple', 'banana', 'cherry']

# [A-Z] 匹配大写字母
text2 = "Hello World"
matches3 = re.findall(r'[A-Z]', text2)
print(matches3)  # ['H', 'W']

# [0-9] 匹配数字
text3 = "Room 101, Floor 5"
matches4 = re.findall(r'[0-9]+', text3)
print(matches4)  # ['101', '5']

# 组合使用 [a-zA-Z0-9]
text4 = "User123"
matches5 = re.findall(r'[a-zA-Z0-9]+', text4)
print(matches5)  # ['User123']

# 特殊字符在[]中
text5 = "file-name_v2.txt"
matches6 = re.findall(r'[a-z_-]+', text5)
print(matches6)  # ['file-name_v']
```

**字符类的范围：**
- `[a-z]`：所有小写字母
- `[A-Z]`：所有大写字母
- `[0-9]`：所有数字
- `[a-zA-Z]`：所有字母
- `[a-zA-Z0-9]`：所有字母和数字

### 2.3 否定字符类 [^] - 匹配不在其中的字符

**`[^...]`匹配不在字符类中的任意字符。**

```python
import re

# [^0-9] 匹配非数字
text = "abc123def456"
matches = re.findall(r'[^0-9]+', text)
print(matches)  # ['abc', 'def']

# [^a-z] 匹配非小写字母
text2 = "Hello123World"
matches2 = re.findall(r'[^a-z]+', text2)
print(matches2)  # ['H', '123W']

# [^aeiou] 匹配非元音字母
text3 = "hello world"
matches3 = re.findall(r'[^aeiou]', text3)
print(matches3)  # ['h', 'l', 'l', ' ', 'w', 'r', 'l', 'd']
```

**注意：**`^`在`[]`内表示"非"，在`[]`外表示"开头"（后面会讲到）

### 2.4 预定义字符类 - 简写

为了方便，正则表达式提供了一些预定义的字符类：

| 简写 | 等价于 | 含义 |
|------|--------|------|
| `\d` | `[0-9]` | 数字 |
| `\D` | `[^0-9]` | 非数字 |
| `\w` | `[a-zA-Z0-9_]` | 单词字符（字母、数字、下划线） |
| `\W` | `[^a-zA-Z0-9_]` | 非单词字符 |
| `\s` | `[ \t\n\r\f\v]` | 空白字符（空格、制表符、换行符等） |
| `\S` | `[^ \t\n\r\f\v]` | 非空白字符 |

```python
import re

text = "Hello World 123! How are you?"

# \d 匹配数字
print(re.findall(r'\d', text))      # ['1', '2', '3']
print(re.findall(r'\d+', text))     # ['123']

# \D 匹配非数字
print(re.findall(r'\D+', text))     # ['Hello World ', '! How are you?']

# \w 匹配单词字符
print(re.findall(r'\w+', text))     # ['Hello', 'World', '123', 'How', 'are', 'you']

# \W 匹配非单词字符
print(re.findall(r'\W+', text))     # [' ', ' ', '! ', ' ', ' ', '?']

# \s 匹配空白字符
print(re.findall(r'\s+', text))     # [' ', ' ', ' ', ' ', ' ']

# \S 匹配非空白字符
print(re.findall(r'\S+', text))     # ['Hello', 'World', '123!', 'How', 'are', 'you?']
```

**记忆技巧：**
- 小写字母（`\d`, `\w`, `\s`）：匹配特定类型
- 大写字母（`\D`, `\W`, `\S`）：匹配相反类型（非...）

### 2.5 量词 - 指定匹配次数

**量词用来指定前面的字符或模式应该出现多少次。**

#### 基本量词

| 量词 | 含义 | 示例 |
|------|------|------|
| `*` | 0次或多次（任意次） | `a*` 匹配 "", "a", "aa", "aaa"... |
| `+` | 1次或多次（至少1次） | `a+` 匹配 "a", "aa", "aaa"...（不匹配""） |
| `?` | 0次或1次（可选） | `a?` 匹配 "" 或 "a" |
| `{n}` | 恰好n次 | `a{3}` 匹配 "aaa" |
| `{n,}` | 至少n次 | `a{2,}` 匹配 "aa", "aaa", "aaaa"... |
| `{n,m}` | n到m次 | `a{2,4}` 匹配 "aa", "aaa", "aaaa" |

```python
import re

text = "a aa aaa aaaa aaaaa"

# * 匹配0次或多次
print(re.findall(r'a*', text))      # 匹配所有（包括空字符串）
print(re.findall(r'ba*', text))     # [] 没有b开头的

# + 匹配1次或多次
print(re.findall(r'a+', text))      # ['a', 'aa', 'aaa', 'aaaa', 'aaaaa']

# ? 匹配0次或1次
print(re.findall(r'a?', text))      # 每个位置都匹配（0次或1次）

# {n} 恰好n次
print(re.findall(r'a{3}', text))    # ['aaa', 'aaa', 'aaa']

# {n,} 至少n次
print(re.findall(r'a{3,}', text))   # ['aaa', 'aaaa', 'aaaaa']

# {n,m} n到m次
print(re.findall(r'a{2,4}', text))  # ['aa', 'aaa', 'aaaa', 'aaaa']
```

#### 实际应用示例

```python
import re

# 匹配手机号（11位数字）
phone = "我的手机号是13812345678"
match = re.search(r'\d{11}', phone)
print(match.group() if match else "未找到")  # 13812345678

# 匹配邮政编码（6位数字）
zipcode = "地址：北京市 100000"
match = re.search(r'\d{6}', zipcode)
print(match.group() if match else "未找到")  # 100000

# 匹配HTML标签
html = "<div>内容</div>"
tags = re.findall(r'<\w+>', html)
print(tags)  # ['<div>', '</div>']  # 注意：这只是简单示例

# 匹配文件名
filename = "report_2024_final.pdf"
match = re.search(r'\w+\.\w+', filename)
print(match.group() if match else "未找到")  # report_2024_final.pdf
```

#### 贪婪匹配 vs 非贪婪匹配

**重要概念：**量词默认是**贪婪的**（greedy），会尽可能多地匹配字符。

```python
import re

text = "<div>内容1</div><div>内容2</div>"

# 贪婪匹配：尽可能多地匹配
greedy = re.findall(r'<div>.*</div>', text)
print(greedy)  # ['<div>内容1</div><div>内容2</div>']
# .* 匹配了从第一个<div>到最后一个</div>之间的所有内容

# 非贪婪匹配：尽可能少地匹配（在量词后加?）
non_greedy = re.findall(r'<div>.*?</div>', text)
print(non_greedy)  # ['<div>内容1</div>', '<div>内容2</div>']
# .*? 匹配最少的字符，直到遇到第一个</div>


# 更多非贪婪示例
text2 = "aaaa"
print(re.findall(r'a+', text2))   # ['aaaa'] 贪婪
print(re.findall(r'a+?', text2))  # ['a', 'a', 'a', 'a'] 非贪婪

text3 = "123456"
print(re.findall(r'\d{2,5}', text3))   # ['12345', '6'] 贪婪
print(re.findall(r'\d{2,5}?', text3))  # ['12', '34', '56'] 非贪婪
```

**非贪婪量词表：**

| 贪婪 | 非贪婪 | 含义 |
|------|--------|------|
| `*` | `*?` | 0次或多次，非贪婪 |
| `+` | `+?` | 1次或多次，非贪婪 |
| `?` | `??` | 0次或1次，非贪婪 |
| `{n,m}` | `{n,m}?` | n到m次，非贪婪 |

### 2.6 锚点 - 指定位置

**锚点用来匹配字符串的特定位置，而不是字符本身。**

| 锚点 | 含义 | 说明 |
|------|------|------|
| `^` | 字符串开头 | 必须从开头开始匹配 |
| `$` | 字符串结尾 | 必须匹配到结尾 |
| `\b` | 单词边界 | 单词的开始或结束位置 |
| `\B` | 非单词边界 | 不是单词边界的位置 |

```python
import re

text = "hello world"

# ^ 匹配开头
print(re.search(r'^hello', text))    # 匹配
print(re.search(r'^world', text))    # 不匹配（world不在开头）

# $ 匹配结尾
print(re.search(r'world$', text))    # 匹配
print(re.search(r'hello$', text))    # 不匹配（hello不在结尾）

# ^和$一起使用：整个字符串必须匹配
print(re.fullmatch(r'^hello world$', text))  # 匹配
print(re.match(r'^hello world$', text))      # 匹配


# \b 单词边界
text2 = "The cat is in the cathedral"
print(re.findall(r'\bcat\b', text2))      # ['cat'] 只匹配单词cat
print(re.findall(r'cat', text2))          # ['cat', 'cat'] 匹配所有cat

# 更多单词边界示例
text3 = "hello_world hello-world hello world"
print(re.findall(r'\bhello\b', text3))    # ['hello', 'hello'] 
# hello_world中的hello不算独立单词（_是单词字符）
# hello-world中的hello算独立单词（-不是单词字符）


# \B 非单词边界
text4 = "please enter"
print(re.findall(r'\Bea', text4))         # ['ea'] pl(ea)se中的ea
print(re.findall(r'ea\B', text4))         # ['ea'] pl(ea)se中的ea
```

#### 锚点的实际应用

```python
import re

# 验证用户名（只允许字母和数字，3-16位）
def validate_username(username):
    pattern = r'^[a-zA-Z0-9]{3,16}$'
    return bool(re.match(pattern, username))

print(validate_username("john123"))    # True
print(validate_username("jo"))         # False（太短）
print(validate_username("john_123"))   # False（包含下划线）
print(validate_username("john 123"))   # False（包含空格）


# 提取整个单词
text = "The price is 100 dollars, not 1000 or 10000"
# 只提取完整的数字（不包括数字的一部分）
numbers = re.findall(r'\b\d+\b', text)
print(numbers)  # ['100', '1000', '10000']


# 匹配行首的#（Markdown标题）
markdown = """
# 标题1
## 标题2
这是内容#不是标题
### 标题3
"""
titles = re.findall(r'^#+\s+.+$', markdown, re.MULTILINE)
print(titles)  # ['# 标题1', '## 标题2', '### 标题3']
```

### 2.7 选择和分组

#### 或运算符 |

**`|`表示"或"，匹配左边或右边的模式。**

```python
import re

# 基本用法
text = "I like cat and dog"
matches = re.findall(r'cat|dog', text)
print(matches)  # ['cat', 'dog']

# 匹配多个可能性
text2 = "Color: red, Color: blue, Color: green"
colors = re.findall(r'red|blue|green|yellow', text2)
print(colors)  # ['red', 'blue', 'green']

# 配合其他模式
text3 = "I have a cat and a dog"
# 注意优先级
print(re.findall(r'cat|dog', text3))    # ['cat', 'dog']
print(re.findall(r'a cat|a dog', text3)) # ['a cat', 'a dog']
```

#### 分组 ()

**圆括号`()`用于分组，可以：**
1. 改变优先级
2. 提取特定部分（捕获组）
3. 应用量词到整个组

```python
import re

# 1. 改变优先级
text = "I like apple and banana"
# 不使用分组
print(re.findall(r'apple|banana', text))  # ['apple', 'banana']

# 使用分组
text2 = "repeat repeat"
print(re.findall(r'(re)+', text2))         # ['re', 're']
print(re.findall(r'(peat)+', text2))       # ['peat', 'peat']


# 2. 对整个组应用量词
phone = "Phone: 010-12345678 or 021-87654321"
# 匹配区号-号码的格式
matches = re.findall(r'\d{3,4}-\d{7,8}', phone)
print(matches)  # ['010-12345678', '021-87654321']

# 使用分组使模式更清晰
pattern = r'(\d{3,4})-(\d{7,8})'
matches = re.findall(pattern, phone)
print(matches)  # [('010', '12345678'), ('021', '87654321')]


# 3. 重复整个模式
text3 = "ha haha hahaha"
print(re.findall(r'(ha)+', text3))  # ['ha', 'ha', 'ha']
print(re.findall(r'(ha){2}', text3))  # ['ha', 'ha']
print(re.findall(r'(ha){2,3}', text3))  # ['ha', 'ha']


# 实际应用：匹配日期
date_text = "今天是2024-11-19，明天是2024-11-20"
dates = re.findall(r'(\d{4})-(\d{2})-(\d{2})', date_text)
print(dates)  # [('2024', '11', '19'), ('2024', '11', '20')]

for year, month, day in dates:
    print(f"年：{year}，月：{month}，日：{day}")
```

---

## 3. re模块详解

### 3.1 re模块的主要函数

Python的`re`模块提供了多个函数来处理正则表达式。

#### 3.1.1 re.search() - 查找第一个匹配

**在字符串中搜索第一个匹配的位置。**

```python
import re

text = "The price is 100 dollars and 200 euros"

# 查找第一个数字
match = re.search(r'\d+', text)
if match:
    print(f"找到：{match.group()}")    # 找到：100
    print(f"位置：{match.span()}")     # 位置：(13, 16)
    print(f"开始：{match.start()}")    # 开始：13
    print(f"结束：{match.end()}")      # 结束：16

# 未找到返回None
match2 = re.search(r'xyz', text)
print(match2)  # None
```

#### 3.1.2 re.match() - 从开头匹配

**从字符串的开头开始匹配（相当于模式前加了^）。**

```python
import re

text = "Python is powerful"

# 从开头匹配
match1 = re.match(r'Python', text)
print(match1)  # <re.Match object...>

# 不从开头则失败
match2 = re.match(r'powerful', text)
print(match2)  # None

# 但search可以找到
match3 = re.search(r'powerful', text)
print(match3)  # <re.Match object...>
```

**使用场景：**验证字符串格式（如用户输入）

```python
import re

def validate_email(email):
    """验证邮箱格式（简单版本）"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

print(validate_email("user@example.com"))    # True
print(validate_email("invalid.email"))       # False
print(validate_email("@example.com"))        # False
```

#### 3.1.3 re.fullmatch() - 完全匹配

**整个字符串必须完全匹配模式。**

```python
import re

pattern = r'\d{3}'

print(re.match(pattern, '123'))      # 匹配
print(re.match(pattern, '123456'))   # 也匹配（从开头匹配了123）

print(re.fullmatch(pattern, '123'))     # 匹配
print(re.fullmatch(pattern, '123456'))  # 不匹配（有额外字符）

# 实际应用：严格验证
def validate_phone(phone):
    """验证手机号（必须恰好11位数字）"""
    return bool(re.fullmatch(r'\d{11}', phone))

print(validate_phone('13812345678'))   # True
print(validate_phone('138123456789'))  # False（12位）
print(validate_phone('1381234567'))    # False（10位）
```

#### 3.1.4 re.findall() - 查找所有匹配

**返回所有匹配的列表。**

```python
import re

text = "我的手机号是13812345678，座机是010-12345678"

# 查找所有数字
numbers = re.findall(r'\d+', text)
print(numbers)  # ['13812345678', '010', '12345678']

# 查找所有单词
text2 = "Hello World, Python is great!"
words = re.findall(r'\w+', text2)
print(words)  # ['Hello', 'World', 'Python', 'is', 'great']

# 使用分组时返回元组列表
text3 = "Price: $100, Tax: $20, Total: $120"
prices = re.findall(r'\$(\d+)', text3)
print(prices)  # ['100', '20', '120'] 只返回括号内的内容

# 多个分组
text4 = "Name: John, Age: 25; Name: Mary, Age: 30"
info = re.findall(r'Name: (\w+), Age: (\d+)', text4)
print(info)  # [('John', '25'), ('Mary', '30')]
```

#### 3.1.5 re.finditer() - 返回迭代器

**返回匹配对象的迭代器，比findall()更节省内存。**

```python
import re

text = "联系方式：13812345678，座机：010-12345678"

# 使用finditer
for match in re.finditer(r'\d+', text):
    print(f"找到：{match.group()}，位置：{match.span()}")
# 输出:
# 找到：13812345678，位置：(5, 16)
# 找到：010，位置：(20, 23)
# 找到：12345678，位置：(24, 32)

# 适合处理大文件
def process_large_log(filename):
    """处理大日志文件"""
    pattern = r'ERROR: (.+)'
    with open(filename, 'r') as f:
        content = f.read()
        for match in re.finditer(pattern, content):
            print(f"错误信息：{match.group(1)}")
```

#### 3.1.6 re.sub() - 替换匹配的内容

**替换字符串中匹配的部分。**

```python
import re

# 基本替换
text = "我的手机号是13812345678"
new_text = re.sub(r'\d', '*', text)
print(new_text)  # 我的手机号是***********

# 限制替换次数
text2 = "apple apple apple"
new_text2 = re.sub(r'apple', 'orange', text2, count=2)
print(new_text2)  # orange orange apple

# 使用分组引用
text3 = "张三的邮箱是zhangsan@example.com，李四的邮箱是lisi@example.com"
new_text3 = re.sub(r'(\w+)@example\.com', r'\1@newdomain.com', text3)
print(new_text3)  # 张三的邮箱是zhangsan@newdomain.com，李四的邮箱是lisi@newdomain.com

# 使用函数进行替换
def mask_phone(match):
    """隐藏手机号中间4位"""
    phone = match.group()
    return phone[:3] + '****' + phone[7:]

text4 = "联系电话：13812345678"
new_text4 = re.sub(r'\d{11}', mask_phone, text4)
print(new_text4)  # 联系电话：138****5678
```

**实际应用：文本清理**

```python
import re

# 移除HTML标签
html = "<p>这是<strong>加粗</strong>的文本</p>"
clean_text = re.sub(r'<[^>]+>', '', html)
print(clean_text)  # 这是加粗的文本

# 标准化空白字符
text = "Hello    World\t\nPython"
normalized = re.sub(r'\s+', ' ', text)
print(normalized)  # Hello World Python

# 移除特殊字符
text2 = "价格：¥100.00！优惠价：¥80.00！"
clean_price = re.sub(r'[¥！]', '', text2)
print(clean_price)  # 价格：100.00优惠价：80.00
```

#### 3.1.7 re.split() - 按模式分割字符串

**按照正则表达式匹配的分隔符分割字符串。**

```python
import re

# 按空白字符分割
text = "apple   banana\tcherry\ndate"
words = re.split(r'\s+', text)
print(words)  # ['apple', 'banana', 'cherry', 'date']

# 按多种分隔符分割
text2 = "apple,banana;cherry|date"
fruits = re.split(r'[,;|]', text2)
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

# 限制分割次数
text3 = "a-b-c-d-e"
parts = re.split(r'-', text3, maxsplit=2)
print(parts)  # ['a', 'b', 'c-d-e']

# 保留分隔符（使用捕获组）
text4 = "apple123banana456cherry"
parts = re.split(r'(\d+)', text4)
print(parts)  # ['apple', '123', 'banana', '456', 'cherry']
```

#### 3.1.8 re.compile() - 编译正则表达式

**将正则表达式编译成Pattern对象，可以重复使用，提高效率。**

```python
import re

# 不编译（每次都要重新解析）
text = "test"
for _ in range(1000):
    re.search(r'\w+', text)

# 编译后使用（效率更高）
pattern = re.compile(r'\w+')
text = "test"
for _ in range(1000):
    pattern.search(text)

# 实际使用示例
phone_pattern = re.compile(r'1[3-9]\d{9}')
email_pattern = re.compile(r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}')

text = "联系方式：13812345678，邮箱：user@example.com"
phone = phone_pattern.search(text)
email = email_pattern.search(text)

print(f"手机号：{phone.group() if phone else '未找到'}")
print(f"邮箱：{email.group() if email else '未找到'}")
```

### 3.2 Match对象的方法

当匹配成功时，会返回一个Match对象，它包含了匹配的详细信息。

```python
import re

text = "Python version 3.13.0 was released in 2024"
pattern = r'(\w+) (\d+)\.(\d+)\.(\d+)'
match = re.search(pattern, text)

if match:
    # group() - 获取匹配的字符串
    print(match.group())      # Python version 3.13.0
    print(match.group(0))     # Python version 3.13.0 (完整匹配)
    print(match.group(1))     # Python (第1个分组)
    print(match.group(2))     # 3 (第2个分组)
    print(match.group(3))     # 13 (第3个分组)
    print(match.group(4))     # 0 (第4个分组)
    
    # groups() - 获取所有分组的元组
    print(match.groups())     # ('Python', '3', '13', '0')
    
    # span() - 获取匹配的位置
    print(match.span())       # (0, 21)
    print(match.span(1))      # (0, 6) 第1个分组的位置
    
    # start(), end() - 获取开始和结束位置
    print(match.start())      # 0
    print(match.end())        # 21
    
    # groupdict() - 如果有命名分组，返回字典
```

### 3.3 编译标志（Flags）

标志可以修改正则表达式的行为。

| 标志 | 简写 | 说明 |
|------|------|------|
| `re.IGNORECASE` | `re.I` | 忽略大小写 |
| `re.MULTILINE` | `re.M` | 多行模式，^和$匹配每行的开头和结尾 |
| `re.DOTALL` | `re.S` | .匹配包括换行符在内的所有字符 |
| `re.VERBOSE` | `re.X` | 详细模式，可以添加注释和空白 |
| `re.ASCII` | `re.A` | ASCII模式，\w等只匹配ASCII字符 |

```python
import re

# IGNORECASE - 忽略大小写
text = "Python PYTHON python"
matches = re.findall(r'python', text, re.IGNORECASE)
print(matches)  # ['Python', 'PYTHON', 'python']

# MULTILINE - 多行模式
text2 = """第一行
第二行
第三行"""
# 不使用MULTILINE
print(re.findall(r'^第', text2))           # ['第'] 只匹配第一个
# 使用MULTILINE
print(re.findall(r'^第', text2, re.MULTILINE))  # ['第', '第', '第']

# DOTALL - .匹配换行符
text3 = "hello\nworld"
print(re.search(r'hello.world', text3))           # None
print(re.search(r'hello.world', text3, re.DOTALL))  # 匹配

# VERBOSE - 详细模式（增强可读性）
pattern = re.compile(r"""
    \d{3,4}    # 区号（3-4位数字）
    -          # 连字符
    \d{7,8}    # 电话号码（7-8位数字）
""", re.VERBOSE)

phone = "010-12345678"
print(pattern.search(phone))  # 匹配

# 组合多个标志
pattern2 = re.compile(r'python', re.IGNORECASE | re.MULTILINE)
```

---

## 4. 分组和捕获

### 4.1 捕获组

**用圆括号`()`定义的组会被捕获，可以通过索引或名称获取。**

```python
import re

# 基本捕获组
text = "Born: 1990-05-15"
match = re.search(r'(\d{4})-(\d{2})-(\d{2})', text)
if match:
    year, month, day = match.groups()
    print(f"年：{year}，月：{month}，日：{day}")
    # 输出：年：1990，月：05，日：15
```

### 4.2 命名组

**使用`(?P<name>...)`给分组命名，通过名称访问更清晰。**

```python
import re

# 命名组
text = "姓名：张三，年龄：25岁"
pattern = r'姓名：(?P<name>\w+)，年龄：(?P<age>\d+)岁'
match = re.search(pattern, text)

if match:
    print(match.group('name'))  # 张三
    print(match.group('age'))   # 25
    print(match.groupdict())    # {'name': '张三', 'age': '25'}

# 实际应用：解析日志
log = "2024-11-19 10:30:00 ERROR Database connection failed"
pattern = r'(?P<date>\d{4}-\d{2}-\d{2}) (?P<time>\d{2}:\d{2}:\d{2}) (?P<level>\w+) (?P<message>.+)'
match = re.search(pattern, log)

if match:
    log_dict = match.groupdict()
    print(f"日期：{log_dict['date']}")
    print(f"时间：{log_dict['time']}")
    print(f"级别：{log_dict['level']}")
    print(f"信息：{log_dict['message']}")
```

### 4.3 非捕获组

**使用`(?:...)`创建非捕获组，不会被保存，只用于分组。**

```python
import re

text = "http://www.example.com"

# 捕获组
pattern1 = r'(https?)://(www\.)?(\w+\.\w+)'
match1 = re.search(pattern1, text)
print(match1.groups())  # ('http', 'www.', 'example.com')

# 非捕获组（www.部分）
pattern2 = r'(https?)://(?:www\.)?(\w+\.\w+)'
match2 = re.search(pattern2, text)
print(match2.groups())  # ('http', 'example.com')
# www.部分不被捕获，groups()中没有它
```

**为什么使用非捕获组？**
- 提高性能（不需要保存）
- 简化结果（减少不需要的分组）
- 保持分组编号简洁

### 4.4 反向引用

**在正则表达式中引用前面的分组。**

```python
import re

# 匹配重复的单词
text = "the the cat cat sat on the mat"
duplicates = re.findall(r'\b(\w+)\s+\1\b', text)
print(duplicates)  # ['the', 'cat']

# 匹配HTML标签对
html = "<div>内容</div><span>文本</span>"
tags = re.findall(r'<(\w+)>.*?</\1>', html)
print(tags)  # ['div', 'span']

# 匹配引号内的内容（单引号或双引号配对）
text2 = '''He said "Hello" and she said 'World' '''
quotes = re.findall(r'(["\']).*?\1', text2)
print(quotes)  # ['"', "'"]
```

### 4.5 前瞻和后顾断言

**断言不消耗字符，只判断位置是否符合条件。**

#### 正向前瞻 (?=...)

**匹配后面跟着特定模式的位置。**

```python
import re

# 匹配后面跟着数字的单词
text = "file1 file2 document test3"
matches = re.findall(r'\w+(?=\d)', text)
print(matches)  # ['file', 'file', 'test']

# 密码强度：至少包含一个数字
def has_digit(password):
    return bool(re.search(r'(?=.*\d)', password))

print(has_digit("abc123"))  # True
print(has_digit("abcdef"))  # False
```

#### 负向前瞻 (?!...)

**匹配后面不跟着特定模式的位置。**

```python
import re

# 匹配后面不跟数字的单词
text = "file1 file2 document test3"
matches = re.findall(r'\w+(?!\d)', text)
print(matches)  # ['fil', 'fil', 'document', 'tes']

# 更精确的版本
matches = re.findall(r'\b\w+\b(?!\d)', text)
print(matches)  # ['document']
```

#### 正向后顾 (?<=...)

**匹配前面是特定模式的位置。**

```python
import re

# 提取货币符号后的数字
text = "价格：$100, ￥200, €50"
prices = re.findall(r'(?<=[$￥€])\d+', text)
print(prices)  # ['100', '200', '50']

# 匹配@后面的用户名
text2 = "Follow @alice and @bob on Twitter"
users = re.findall(r'(?<=@)\w+', text2)
print(users)  # ['alice', 'bob']
```

#### 负向后顾 (?<!...)

**匹配前面不是特定模式的位置。**

```python
import re

# 匹配不以$开头的数字
text = "$100 200 $300 400"
numbers = re.findall(r'(?<!\$)\b\d+\b', text)
print(numbers)  # ['200', '400']
```

---

## 5. 高级应用

### 5.1 常用正则表达式模式

#### 邮箱地址

```python
import re

def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

emails = ["user@example.com", "invalid.email", "test@domain.co.uk", "@example.com"]
for email in emails:
    print(f"{email}: {validate_email(email)}")
```

#### 手机号

```python
import re

def validate_chinese_phone(phone):
    """验证中国大陆手机号"""
    pattern = r'^1[3-9]\d{9}$'
    return bool(re.match(pattern, phone))

phones = ["13812345678", "15912345678", "12812345678", "1381234567"]
for phone in phones:
    print(f"{phone}: {validate_chinese_phone(phone)}")
```

#### URL

```python
import re

def extract_urls(text):
    pattern = r'https?://(?:www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b(?:[-a-zA-Z0-9()@:%_\+.~#?&/=]*)'
    return re.findall(pattern, text)

text = "访问 https://www.example.com 或 http://test.org/path?query=1"
urls = extract_urls(text)
print(urls)
```

#### IP地址

```python
import re

def validate_ipv4(ip):
    pattern = r'^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$'
    return bool(re.match(pattern, ip))

ips = ["192.168.1.1", "255.255.255.255", "256.1.1.1", "192.168.1"]
for ip in ips:
    print(f"{ip}: {validate_ipv4(ip)}")
```

### 5.2 性能优化技巧

```python
import re
import time

# 1. 使用re.compile()预编译
pattern = re.compile(r'\d+')
text = "test 123 test"

# 2. 使用非捕获组
# 慢：re.findall(r'(https?)://(www\.)?(\w+)', text)
# 快：re.findall(r'(?:https?)://(?:www\.)?(\w+)', text)

# 3. 尽早使用锚点
# 慢：re.search(r'test\d+', long_text)
# 快：re.search(r'^test\d+', long_text)  # 如果确定在开头

# 4. 使用原子组避免回溯（高级）
# 慢：re.search(r'a+b+c', 'aaaaaaaaaaaaaaaaaaaaax')
# 快：re.search(r'(?>a+)(?>b+)c', 'aaaaaaaaaaaaaaaaaaaaax')
```

---

## 6. 实战案例

### 案例1：提取网页中的所有链接

```python
import re

html = '''
<a href="https://www.example.com">Example</a>
<a href="/path/to/page">Internal</a>
<a href="https://test.org/page?id=123">Test</a>
'''

# 提取所有href属性的值
links = re.findall(r'href=["\']([^"\']+)["\']', html)
print("所有链接:")
for link in links:
    print(f"  {link}")
```

### 案例2：解析日志文件

```python
import re

log_entry = '''
2024-11-19 10:30:15 INFO User login successful: user_id=12345
2024-11-19 10:31:20 ERROR Database connection failed: timeout
2024-11-19 10:32:00 WARNING Memory usage high: 85%
'''

pattern = r'(?P<date>\d{4}-\d{2}-\d{2}) (?P<time>\d{2}:\d{2}:\d{2}) (?P<level>\w+) (?P<message>.+)'

for match in re.finditer(pattern, log_entry):
    log_dict = match.groupdict()
    if log_dict['level'] == 'ERROR':
        print(f"错误日志：{log_dict['message']}")
```

### 案例3：数据清洗

```python
import re

# 清理用户输入的手机号
def clean_phone(phone):
    # 移除所有非数字字符
    cleaned = re.sub(r'[^\d]', '', phone)
    return cleaned

phones = ["138-1234-5678", "(138)12345678", "138 1234 5678"]
for phone in phones:
    print(f"{phone} -> {clean_phone(phone)}")
```

### 案例4：Markdown解析

```python
import re

markdown = '''
# 标题1
## 标题2
这是**加粗**的文本
这是*斜体*的文本
'''

# 提取标题
titles = re.findall(r'^(#+)\s+(.+)$', markdown, re.MULTILINE)
for hashes, title in titles:
    level = len(hashes)
    print(f"H{level}: {title}")

# 提取加粗文本
bold = re.findall(r'\*\*(.+?)\*\*', markdown)
print(f"加粗文本: {bold}")

# 提取斜体文本
italic = re.findall(r'\*(.+?)\*', markdown)
print(f"斜体文本: {italic}")
```

---

## 总结

### 核心概念回顾

1. **基础元字符**
   - `.` 任意字符
   - `[]` 字符类
   - `\d \w \s` 预定义字符类

2. **量词**
   - `* + ? {n} {n,} {n,m}`
   - 贪婪 vs 非贪婪

3. **锚点**
   - `^ $ \b \B`
   - 位置匹配

4. **分组和捕获**
   - `()` 捕获组
   - `(?P<name>)` 命名组
   - `(?:)` 非捕获组
   - `\1 \2` 反向引用

5. **re模块函数**
   - `search()` 查找第一个
   - `match()` 从开头匹配
   - `findall()` 查找所有
   - `sub()` 替换
   - `split()` 分割

### 学习建议

1. **多练习**：从简单模式开始，逐步增加复杂度
2. **使用工具**：regex101.com 等在线工具可以可视化调试
3. **阅读文档**：Python官方re模块文档
4. **避免过度优化**：可读性优先于性能
5. **测试边界情况**：确保正则表达式健壮

### 最佳实践

✅ **推荐做法**
- 使用原始字符串 `r''`
- 预编译频繁使用的模式
- 使用命名组提高可读性
- 添加注释（VERBOSE模式）
- 充分测试边界情况

❌ **避免做法**
- 过度复杂的正则表达式
- 用正则表达式解析HTML/XML（使用专门的解析器）
- 忽略性能问题
- 没有处理匹配失败的情况

---

**祝你掌握正则表达式这个强大的文本处理工具！🎯**

