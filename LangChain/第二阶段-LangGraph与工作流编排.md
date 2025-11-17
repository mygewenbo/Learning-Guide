# 第二阶段：LangGraph 与工作流编排（详细讲解）

> **学习目标**：深入理解 LangGraph、掌握 StateGraph 状态管理、学会条件路由和并行执行、实现复杂工作流
> 
> **预计学习时间**：2-3周
> 
> **前置要求**：完成第一阶段学习、理解 Agent 基础、熟悉 Python 类型系统

---

## 📚 第二阶段学习大纲

1. [LangGraph 深入理解](#第一部分langgraph-深入理解)
   - 什么是 LangGraph？
   - 为什么需要 LangGraph？
   - LangGraph vs Agent
   - 核心概念详解

2. [StateGraph 状态管理](#第二部分stategraph-状态管理)
   - State 定义和类型
   - Annotated 和 Reducer
   - 状态更新机制
   - 最佳实践

3. [节点与边的艺术](#第三部分节点与边的艺术)
   - Node 节点详解
   - Edge 边的类型
   - 控制流设计
   - 图的可视化

4. [条件路由系统](#第四部分条件路由系统)
   - 条件边基础
   - 路由函数设计
   - 多路径路由
   - Command 动态路由

5. [并行执行策略](#第五部分并行执行策略)
   - 并行节点设计
   - Send API 详解
   - 编排者-工作者模式
   - 性能优化

6. [Checkpointer 持久化](#第六部分checkpointer-持久化)
   - 内存检查点
   - 数据库持久化
   - 状态恢复机制
   - 时间旅行调试

---

## 第一部分：LangGraph 深入理解

### 1.1 什么是 LangGraph？

**LangGraph** 是一个用于构建**有状态、多步骤应用程序**的低级编排框架。它是 LangChain 生态系统的一部分，专注于**复杂工作流**的构建。

#### 核心定位

```
简单任务              LangChain Agent
  ↓                        ↓
单步工具调用         create_agent()
快速原型                 简单配置

────────────────────────────────────

复杂任务              LangGraph
  ↓                        ↓
多步骤工作流         StateGraph
精细控制               显式编排
```

#### LangGraph 的独特之处

| 特性 | LangChain Agent | LangGraph |
|-----|----------------|-----------|
| **抽象级别** | 高级 | 低级 |
| **控制粒度** | 自动决策 | 显式编排 |
| **状态管理** | 隐式 | 显式 |
| **适用场景** | 简单任务 | 复杂工作流 |
| **学习曲线** | 平缓 | 陡峭 |

### 1.2 为什么需要 LangGraph？

#### 问题 1：Agent 的局限性

```python
# ❌ 使用 Agent 处理复杂多步骤任务的问题

from langchain.agents import create_agent
from langchain.tools import tool

@tool
def research(topic: str) -> str:
    """研究主题"""
    return f"关于 {topic} 的研究结果"

@tool
def write_draft(research_result: str) -> str:
    """撰写草稿"""
    return f"基于研究的草稿"

@tool
def review(draft: str) -> str:
    """审核草稿"""
    return f"审核意见"

@tool
def revise(draft: str, feedback: str) -> str:
    """修订草稿"""
    return f"修订后的草稿"

agent = create_agent(
    model="gpt-4",
    tools=[research, write_draft, review, revise]
)

# 问题：
# 1. Agent 可能跳过某些步骤
# 2. 步骤顺序不可控
# 3. 难以实现复杂的条件逻辑
# 4. 无法并行执行某些步骤
# 5. 状态管理不透明

result = agent.invoke({
    "messages": [{"role": "user", "content": "写一篇关于量子计算的文章"}]
})
```

#### LangGraph 的解决方案

```python
# ✅ 使用 LangGraph 精确控制工作流

from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

# 1. 明确定义状态
class State(TypedDict):
    topic: str
    research_result: str
    draft: str
    feedback: str
    final_article: str
    iteration: int

# 2. 定义每个步骤为节点
def research_node(state: State):
    """研究节点"""
    result = f"关于 {state['topic']} 的深入研究"
    return {"research_result": result}

def write_node(state: State):
    """撰写节点"""
    draft = f"基于研究的文章草稿：{state['research_result']}"
    return {"draft": draft}

def review_node(state: State):
    """审核节点"""
    feedback = f"对草稿的审核意见"
    return {"feedback": feedback}

def revise_node(state: State):
    """修订节点"""
    revised = f"根据 {state['feedback']} 修订后的草稿"
    return {"draft": revised, "iteration": state.get("iteration", 0) + 1}

def finalize_node(state: State):
    """定稿节点"""
    return {"final_article": state["draft"]}

# 3. 定义条件路由
def should_revise(state: State) -> str:
    """决定是否需要修订"""
    if state.get("iteration", 0) < 2:  # 最多修订2次
        return "revise"
    return "finalize"

# 4. 构建工作流
workflow = StateGraph(State)

# 添加节点
workflow.add_node("research", research_node)
workflow.add_node("write", write_node)
workflow.add_node("review", review_node)
workflow.add_node("revise", revise_node)
workflow.add_node("finalize", finalize_node)

# 定义流程（完全可控）
workflow.add_edge(START, "research")
workflow.add_edge("research", "write")
workflow.add_edge("write", "review")
workflow.add_conditional_edges(
    "review",
    should_revise,
    {
        "revise": "revise",
        "finalize": "finalize"
    }
)
workflow.add_edge("revise", "review")  # 形成循环
workflow.add_edge("finalize", END)

# 编译并执行
app = workflow.compile()
result = app.invoke({"topic": "量子计算", "iteration": 0})

print(result["final_article"])

# 优势：
# ✅ 步骤顺序完全可控
# ✅ 可以实现复杂的条件逻辑
# ✅ 状态透明可追踪
# ✅ 可以形成循环和分支
# ✅ 易于调试和优化
```

### 1.3 LangGraph vs Agent 对比

#### 使用 Agent 的场景

```python
# 适合 Agent：简单的工具调用任务

from langchain.agents import create_agent
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """获取天气"""
    return f"{city}的天气：晴天"

@tool
def send_email(to: str, content: str) -> str:
    """发送邮件"""
    return f"已发送邮件到 {to}"

# 创建简单的 Agent
agent = create_agent(
    model="gpt-4",
    tools=[get_weather, send_email],
    system_prompt="你是一个助手"
)

# 简单任务，Agent 可以自主决策
result = agent.invoke({
    "messages": [{"role": "user", "content": "查询北京天气并发邮件给 alice@example.com"}]
})
```

#### 使用 LangGraph 的场景

```python
# 适合 LangGraph：复杂的多步骤工作流

from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict
import operator
from typing import Annotated

class ReportState(TypedDict):
    topic: str
    outline: list[str]
    sections: Annotated[list, operator.add]  # 累加器
    final_report: str

def plan_outline(state: ReportState):
    """规划大纲"""
    outline = ["引言", "主体", "结论"]
    return {"outline": outline}

def write_section(state: ReportState):
    """并行撰写各个章节"""
    # 这里会被多次并行调用
    return {"sections": ["章节内容"]}

def merge_report(state: ReportState):
    """合并报告"""
    report = "\n\n".join(state["sections"])
    return {"final_report": report}

# 构建复杂工作流
workflow = StateGraph(ReportState)
workflow.add_node("plan", plan_outline)
workflow.add_node("write", write_section)
workflow.add_node("merge", merge_report)

workflow.add_edge(START, "plan")
workflow.add_edge("plan", "write")
workflow.add_edge("write", "merge")
workflow.add_edge("merge", END)

app = workflow.compile()
```

#### 选择指南

```python
# 决策树

def choose_framework(task):
    """
    选择合适的框架
    """
    if task.has_clear_steps and task.needs_precise_control:
        return "LangGraph"
    
    if task.is_simple and task.can_auto_decide:
        return "Agent"
    
    if task.has_loops or task.has_complex_conditions:
        return "LangGraph"
    
    if task.needs_parallel_execution:
        return "LangGraph"
    
    if task.is_prototype:
        return "Agent"  # 快速开发
    
    if task.is_production:
        return "LangGraph"  # 更好的控制
    
    return "Agent"  # 默认从简单开始
```

### 1.4 核心概念详解

#### 概念 1：Graph（图）

图是一个有向无环图（DAG）或有向有环图：

```python
from langgraph.graph import StateGraph, START, END

# 图的组成：
# 1. START：起始节点（虚拟节点）
# 2. END：结束节点（虚拟节点）
# 3. 自定义节点：执行具体任务
# 4. 边：连接节点，定义执行顺序

workflow = StateGraph(State)

# START 和 END 是特殊的虚拟节点
# - START: 图的入口，不执行任何操作
# - END: 图的出口，标记执行完成
```

#### 概念 2：State（状态）

状态是在节点之间传递的数据：

```python
from typing_extensions import TypedDict
from typing import Annotated
import operator

# 基础状态
class BasicState(TypedDict):
    name: str
    age: int

# 带累加器的状态
class AccumulatorState(TypedDict):
    messages: Annotated[list, operator.add]  # 自动累加
    count: int

# 状态传递流程：
# Input State → Node 1 → Updated State → Node 2 → Final State
```

#### 概念 3：Node（节点）

节点是执行具体任务的函数：

```python
def my_node(state: State) -> dict:
    """
    节点函数规范：
    1. 接收当前状态作为参数
    2. 执行任务
    3. 返回状态更新（字典）
    """
    # 处理逻辑
    result = process(state)
    
    # 返回状态更新
    return {"field_name": result}

# 节点会自动合并返回的字典到状态中
```

#### 概念 4：Edge（边）

边定义节点之间的连接：

```python
# 1. 普通边（固定路径）
workflow.add_edge("node_a", "node_b")

# 2. 条件边（动态路径）
def route(state: State) -> str:
    if state["value"] > 10:
        return "node_b"
    return "node_c"

workflow.add_conditional_edges("node_a", route)

# 3. 从 START 开始
workflow.add_edge(START, "first_node")

# 4. 到 END 结束
workflow.add_edge("last_node", END)
```

### 1.5 第一个 LangGraph 程序

让我们创建一个完整的 LangGraph 应用：

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

# === 第1步：定义状态 ===
class GreetingState(TypedDict):
    name: str
    greeting: str
    farewell: str

# === 第2步：定义节点 ===
def greet_node(state: GreetingState):
    """问候节点"""
    name = state["name"]
    greeting = f"你好，{name}！欢迎使用 LangGraph。"
    return {"greeting": greeting}

def farewell_node(state: GreetingState):
    """告别节点"""
    name = state["name"]
    farewell = f"再见，{name}！期待下次见面。"
    return {"farewell": farewell}

# === 第3步：构建图 ===
workflow = StateGraph(GreetingState)

# 添加节点
workflow.add_node("greet", greet_node)
workflow.add_node("farewell", farewell_node)

# 添加边（定义执行顺序）
workflow.add_edge(START, "greet")
workflow.add_edge("greet", "farewell")
workflow.add_edge("farewell", END)

# === 第4步：编译图 ===
app = workflow.compile()

# === 第5步：执行图 ===
initial_state = {"name": "小明"}
result = app.invoke(initial_state)

print(result)
# 输出：
# {
#     'name': '小明',
#     'greeting': '你好，小明！欢迎使用 LangGraph。',
#     'farewell': '再见，小明！期待下次见面。'
# }
```

#### 执行流程分析

```
初始状态: {"name": "小明"}
    ↓
[START 节点]
    ↓
[greet 节点]
    输入: {"name": "小明"}
    处理: 生成问候语
    返回: {"greeting": "你好，小明！..."}
    当前状态: {"name": "小明", "greeting": "你好，小明！..."}
    ↓
[farewell 节点]
    输入: {"name": "小明", "greeting": "你好，小明！..."}
    处理: 生成告别语
    返回: {"farewell": "再见，小明！..."}
    当前状态: {
        "name": "小明",
        "greeting": "你好，小明！...",
        "farewell": "再见，小明！..."
    }
    ↓
[END 节点]
    ↓
最终状态: 完整的状态字典
```

### 1.6 可视化图结构

LangGraph 提供了可视化功能：

```python
from IPython.display import Image, display

# 方法 1：生成 Mermaid 图
mermaid_png = app.get_graph().draw_mermaid_png()
display(Image(mermaid_png))

# 方法 2：打印 ASCII 图
print(app.get_graph().draw_ascii())

# 方法 3：获取图的结构信息
graph_dict = app.get_graph().to_json()
print(graph_dict)
```

输出示例（ASCII）：
```
      +-----------+
      |  __start__|
      +-----------+
            *
            *
            *
      +-----------+
      |   greet   |
      +-----------+
            *
            *
            *
      +-----------+
      | farewell  |
      +-----------+
            *
            *
            *
      +-----------+
      |   __end__ |
      +-----------+
```

### 1.7 调试技巧

#### 技巧 1：打印中间状态

```python
def debug_node(state: State):
    """调试节点"""
    print(f"当前状态: {state}")
    return {}  # 不修改状态

# 在关键位置插入调试节点
workflow.add_node("debug_1", debug_node)
workflow.add_edge("some_node", "debug_1")
workflow.add_edge("debug_1", "next_node")
```

#### 技巧 2：使用流式输出

```python
# 流式查看每个步骤
for step in app.stream(initial_state, stream_mode="updates"):
    print("=" * 50)
    print(f"步骤更新: {step}")
```

#### 技巧 3：检查图结构

```python
# 查看所有节点
print("节点:", app.get_graph().nodes)

# 查看所有边
print("边:", app.get_graph().edges)

# 查看入口节点
print("入口:", app.get_graph().entry_point)
```

---

## 第二部分：StateGraph 状态管理

### 2.1 State 定义和类型

#### 基础 State 定义

```python
from typing_extensions import TypedDict

# 最简单的状态定义
class SimpleState(TypedDict):
    name: str
    age: int
    email: str

# 状态字段说明：
# - name: 必需字段，字符串类型
# - age: 必需字段，整数类型
# - email: 必需字段，字符串类型
```

#### 可选字段

```python
from typing_extensions import TypedDict, NotRequired

class StateWithOptional(TypedDict):
    name: str  # 必需
    age: NotRequired[int]  # 可选
    email: NotRequired[str]  # 可选

# 使用示例
state1 = {"name": "Alice"}  # ✅ 有效
state2 = {"name": "Bob", "age": 30}  # ✅ 有效
state3 = {"age": 25}  # ❌ 错误：缺少 name
```

#### 嵌套状态

```python
from typing import List

class Address(TypedDict):
    street: str
    city: str
    zipcode: str

class UserState(TypedDict):
    name: str
    addresses: List[Address]
    metadata: dict

# 使用示例
user_state = {
    "name": "Alice",
    "addresses": [
        {"street": "123 Main St", "city": "Beijing", "zipcode": "100000"}
    ],
    "metadata": {"created_at": "2024-01-01"}
}
```

### 2.2 Annotated 和 Reducer

**Reducer（归约器）** 定义了如何合并状态更新。这是 LangGraph 状态管理的核心机制。

#### 默认行为（覆盖）

```python
from typing_extensions import TypedDict

class State(TypedDict):
    count: int

def node1(state: State):
    return {"count": 5}

def node2(state: State):
    return {"count": 10}

# 执行流程：
# 初始: {"count": 0}
# node1: {"count": 5}  ← 覆盖
# node2: {"count": 10} ← 再次覆盖
```

#### 使用 Reducer（累加）

```python
from typing import Annotated
import operator

class State(TypedDict):
    # 使用 operator.add 作为 reducer
    messages: Annotated[list, operator.add]

def node1(state: State):
    return {"messages": ["Hello"]}

def node2(state: State):
    return {"messages": ["World"]}

# 执行流程：
# 初始: {"messages": []}
# node1: {"messages": ["Hello"]}      ← 添加
# node2: {"messages": ["Hello", "World"]} ← 累加
```

#### 常用 Reducer

```python
import operator
from typing import Annotated

# 1. 列表累加
class State1(TypedDict):
    items: Annotated[list, operator.add]
    # ["A"] + ["B"] = ["A", "B"]

# 2. 数值相加
class State2(TypedDict):
    total: Annotated[int, operator.add]
    # 5 + 3 = 8

# 3. 字符串拼接
class State3(TypedDict):
    text: Annotated[str, operator.add]
    # "Hello" + " World" = "Hello World"

# 4. 集合合并
def merge_sets(left: set, right: set) -> set:
    return left | right

class State4(TypedDict):
    tags: Annotated[set, merge_sets]
    # {1, 2} | {2, 3} = {1, 2, 3}
```

#### 自定义 Reducer

```python
from typing import Annotated, List

def merge_unique_items(existing: List[str], new: List[str]) -> List[str]:
    """合并列表，保持唯一性"""
    combined = existing + new
    return list(set(combined))

def merge_with_limit(existing: List[str], new: List[str], limit: int = 10) -> List[str]:
    """合并列表，限制总数"""
    combined = existing + new
    return combined[-limit:]  # 只保留最后 limit 个

class SmartState(TypedDict):
    # 使用自定义 reducer
    unique_items: Annotated[List[str], merge_unique_items]
    recent_items: Annotated[List[str], lambda x, y: (x + y)[-10:]]

# 使用示例
def node1(state: SmartState):
    return {"unique_items": ["apple", "banana"]}

def node2(state: SmartState):
    return {"unique_items": ["banana", "orange"]}  # banana 不会重复

# 结果：unique_items = ["apple", "banana", "orange"]
```

### 2.3 状态更新机制

#### 部分更新

```python
class FullState(TypedDict):
    name: str
    age: int
    email: str
    address: str

def update_node(state: FullState):
    """节点只需返回要更新的字段"""
    # 只更新 age，其他字段保持不变
    return {"age": state["age"] + 1}

# 执行前: {"name": "Alice", "age": 25, "email": "alice@example.com", "address": "..."}
# 执行后: {"name": "Alice", "age": 26, "email": "alice@example.com", "address": "..."}
```

#### 条件更新

```python
def conditional_update(state: State):
    """根据条件决定更新什么"""
    updates = {}
    
    if state["age"] < 18:
        updates["category"] = "minor"
    else:
        updates["category"] = "adult"
    
    if state.get("email"):
        updates["contact_verified"] = True
    
    return updates
```

#### 批量更新

```python
def batch_update(state: State):
    """一次更新多个字段"""
    return {
        "name": state["name"].upper(),
        "age": state["age"] + 1,
        "last_updated": "2024-01-01",
        "status": "active"
    }
```

### 2.4 实战：构建聊天历史管理

```python
from typing import Annotated, List
from typing_extensions import TypedDict
import operator
from datetime import datetime

# 消息类型
class Message(TypedDict):
    role: str  # "user" 或 "assistant"
    content: str
    timestamp: str

# 聊天状态
class ChatState(TypedDict):
    # 使用 operator.add 累加消息
    messages: Annotated[List[Message], operator.add]
    user_id: str
    conversation_id: str

def user_input_node(state: ChatState):
    """处理用户输入"""
    user_message = {
        "role": "user",
        "content": "你好！",
        "timestamp": datetime.now().isoformat()
    }
    return {"messages": [user_message]}

def ai_response_node(state: ChatState):
    """AI 回复"""
    # 获取最后一条用户消息
    last_message = state["messages"][-1]
    
    ai_message = {
        "role": "assistant",
        "content": f"你好！你刚才说：{last_message['content']}",
        "timestamp": datetime.now().isoformat()
    }
    return {"messages": [ai_message]}

# 构建聊天工作流
from langgraph.graph import StateGraph, START, END

chat_workflow = StateGraph(ChatState)
chat_workflow.add_node("user_input", user_input_node)
chat_workflow.add_node("ai_response", ai_response_node)

chat_workflow.add_edge(START, "user_input")
chat_workflow.add_edge("user_input", "ai_response")
chat_workflow.add_edge("ai_response", END)

chat_app = chat_workflow.compile()

# 运行
result = chat_app.invoke({
    "user_id": "user_123",
    "conversation_id": "conv_456",
    "messages": []
})

print("聊天历史:")
for msg in result["messages"]:
    print(f"[{msg['role']}] {msg['content']}")
```

---

## 第三部分：节点与边的艺术

### 3.1 Node 节点详解

#### 节点函数签名

```python
from typing_extensions import TypedDict

class State(TypedDict):
    value: int

# 标准节点函数
def standard_node(state: State) -> dict:
    """
    标准节点函数必须：
    1. 接收 state 参数（类型为定义的 State）
    2. 返回字典（包含状态更新）
    """
    new_value = state["value"] + 1
    return {"value": new_value}

# 也可以返回部分字段
def partial_node(state: State) -> dict:
    return {"value": 100}  # 只更新 value

# 可以返回空字典（不更新状态）
def observer_node(state: State) -> dict:
    print(f"当前值: {state['value']}")
    return {}  # 观察但不修改
```

#### 异步节点

```python
import asyncio

async def async_node(state: State) -> dict:
    """异步节点用于 I/O 密集型任务"""
    # 模拟异步 API 调用
    await asyncio.sleep(1)
    result = await fetch_data_async(state["value"])
    return {"value": result}

# 在异步图中使用
async def run_async_graph():
    result = await app.ainvoke(initial_state)
    return result
```

#### 节点中的错误处理

```python
def safe_node(state: State) -> dict:
    """带错误处理的节点"""
    try:
        result = risky_operation(state["value"])
        return {"value": result, "error": None}
    except Exception as e:
        # 捕获错误并保存到状态
        return {
            "error": str(e),
            "status": "failed"
        }

# 后续节点可以检查错误
def check_error_node(state: State) -> dict:
    if state.get("error"):
        return {"should_retry": True}
    return {"should_retry": False}
```

### 3.2 Edge 边的类型

#### 类型 1：普通边（固定路径）

```python
from langgraph.graph import StateGraph, START, END

workflow = StateGraph(State)

# 固定的节点转换
workflow.add_edge(START, "node_a")  # 从 START 到 node_a
workflow.add_edge("node_a", "node_b")  # 从 node_a 到 node_b
workflow.add_edge("node_b", END)  # 从 node_b 到 END

# 执行顺序：START → node_a → node_b → END
```

#### 类型 2：条件边（动态路径）

```python
from typing import Literal

def route_function(state: State) -> Literal["path_a", "path_b"]:
    """路由函数决定下一个节点"""
    if state["value"] > 10:
        return "path_a"
    return "path_b"

# 添加条件边
workflow.add_conditional_edges(
    "decision_node",  # 源节点
    route_function,   # 路由函数
    {
        "path_a": "node_a",  # 路由结果 → 目标节点
        "path_b": "node_b"
    }
)

# 执行流程：
# decision_node 执行后 → 调用 route_function → 根据结果选择路径
```

#### 类型 3：多路径路由

```python
from typing import Sequence

def multi_route(state: State) -> Sequence[str]:
    """返回多个目标节点（并行执行）"""
    paths = []
    
    if state["needs_validation"]:
        paths.append("validate")
    
    if state["needs_logging"]:
        paths.append("log")
    
    if state["needs_notification"]:
        paths.append("notify")
    
    return paths if paths else ["default"]

# 添加多路径条件边
workflow.add_conditional_edges(
    "processor",
    multi_route
)
```

### 3.3 控制流设计

#### 模式 1：线性流

```python
# 简单的顺序执行
workflow.add_edge(START, "step1")
workflow.add_edge("step1", "step2")
workflow.add_edge("step2", "step3")
workflow.add_edge("step3", END)

# 流程图：
# START → step1 → step2 → step3 → END
```

#### 模式 2：分支流

```python
def branch_route(state: State) -> str:
    if state["type"] == "A":
        return "process_a"
    elif state["type"] == "B":
        return "process_b"
    else:
        return "process_default"

workflow.add_edge(START, "classifier")
workflow.add_conditional_edges(
    "classifier",
    branch_route,
    {
        "process_a": "process_a",
        "process_b": "process_b",
        "process_default": "process_default"
    }
)
workflow.add_edge("process_a", END)
workflow.add_edge("process_b", END)
workflow.add_edge("process_default", END)

# 流程图：
#          ┌─→ process_a → END
#          │
# START → classifier ─→ process_b → END
#          │
#          └─→ process_default → END
```

#### 模式 3：循环流

```python
def should_continue(state: State) -> Literal["continue", "end"]:
    """判断是否继续循环"""
    if state["iteration"] < state["max_iterations"]:
        return "continue"
    return "end"

workflow.add_edge(START, "initialize")
workflow.add_edge("initialize", "process")
workflow.add_conditional_edges(
    "process",
    should_continue,
    {
        "continue": "process",  # 循环回自己
        "end": END
    }
)

# 流程图：
#             ┌─────┐
#             ↓     │
# START → initialize → process ⟲
#                      │
#                      ↓ (when done)
#                     END
```

#### 模式 4：汇聚流

```python
# 多个分支汇聚到一个节点
workflow.add_edge(START, "branch_a")
workflow.add_edge(START, "branch_b")
workflow.add_edge(START, "branch_c")

# 三个分支都流向 merge 节点
workflow.add_edge("branch_a", "merge")
workflow.add_edge("branch_b", "merge")
workflow.add_edge("branch_c", "merge")

workflow.add_edge("merge", END)

# 流程图：
# START ──→ branch_a ──┐
#       ├─→ branch_b ──┼─→ merge → END
#       └─→ branch_c ──┘
```

### 3.4 图的可视化

#### 生成可视化图

```python
from langgraph.graph import StateGraph, START, END
from IPython.display import Image, display

# 创建工作流
workflow = StateGraph(State)
workflow.add_node("node_a", node_a)
workflow.add_node("node_b", node_b)
workflow.add_edge(START, "node_a")
workflow.add_edge("node_a", "node_b")
workflow.add_edge("node_b", END)

app = workflow.compile()

# 方法 1：Mermaid 图（推荐）
try:
    mermaid_png = app.get_graph().draw_mermaid_png()
    display(Image(mermaid_png))
except Exception as e:
    print(f"无法生成图片: {e}")

# 方法 2：ASCII 图
ascii_graph = app.get_graph().draw_ascii()
print(ascii_graph)

# 方法 3：导出为 JSON
graph_json = app.get_graph().to_json()
print(graph_json)
```

#### 保存可视化

```python
# 保存为文件
with open("workflow.png", "wb") as f:
    f.write(mermaid_png)

# 或使用 graphviz（需要安装）
try:
    import graphviz
    dot = app.get_graph().to_graphviz()
    dot.render("workflow", format="png", cleanup=True)
except ImportError:
    print("需要安装 graphviz: pip install graphviz")
```

---

## 第四部分：条件路由系统

### 4.1 条件边基础

条件边是 LangGraph 最强大的特性之一，允许根据状态动态选择执行路径。

#### 基础条件路由

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict
from typing import Literal

class State(TypedDict):
    age: int
    category: str

def classify_node(state: State):
    """分类节点"""
    return {"age": state["age"]}

def child_process(state: State):
    """儿童处理流程"""
    return {"category": "child"}

def adult_process(state: State):
    """成人处理流程"""
    return {"category": "adult"}

# 路由函数
def age_router(state: State) -> Literal["child", "adult"]:
    """根据年龄路由"""
    if state["age"] < 18:
        return "child"
    return "adult"

# 构建工作流
workflow = StateGraph(State)
workflow.add_node("classify", classify_node)
workflow.add_node("child_process", child_process)
workflow.add_node("adult_process", adult_process)

workflow.add_edge(START, "classify")
workflow.add_conditional_edges(
    "classify",
    age_router,
    {
        "child": "child_process",
        "adult": "adult_process"
    }
)
workflow.add_edge("child_process", END)
workflow.add_edge("adult_process", END)

app = workflow.compile()

# 测试
result1 = app.invoke({"age": 15})  # → child_process
result2 = app.invoke({"age": 25})  # → adult_process
```

### 4.2 路由函数设计模式

#### 模式 1：基于阈值的路由

```python
def threshold_router(state: State) -> str:
    """基于阈值路由"""
    value = state["score"]
    
    if value >= 90:
        return "excellent"
    elif value >= 70:
        return "good"
    elif value >= 60:
        return "pass"
    else:
        return "fail"

workflow.add_conditional_edges(
    "evaluate",
    threshold_router,
    {
        "excellent": "reward",
        "good": "pass",
        "pass": "pass",
        "fail": "retry"
    }
)
```

#### 模式 2：基于类型的路由

```python
def type_router(state: State) -> str:
    """基于数据类型路由"""
    data_type = state.get("data_type")
    
    routing_map = {
        "text": "text_processor",
        "image": "image_processor",
        "video": "video_processor",
        "audio": "audio_processor"
    }
    
    return routing_map.get(data_type, "default_processor")
```

#### 模式 3：多条件组合路由

```python
def complex_router(state: State) -> str:
    """复杂条件组合路由"""
    is_premium = state.get("is_premium", False)
    urgency = state.get("urgency", "normal")
    
    # 组合条件
    if is_premium and urgency == "high":
        return "priority_process"
    elif is_premium:
        return "premium_process"
    elif urgency == "high":
        return "urgent_process"
    else:
        return "standard_process"
```

### 4.3 多路径路由

一个节点可以同时路由到多个节点（并行执行）：

```python
from typing import Sequence

def parallel_router(state: State) -> Sequence[str]:
    """返回多个路径"""
    paths = []
    
    # 根据不同条件添加路径
    if state.get("needs_translation"):
        paths.append("translator")
    
    if state.get("needs_summary"):
        paths.append("summarizer")
    
    if state.get("needs_analysis"):
        paths.append("analyzer")
    
    # 至少执行一个默认路径
    return paths if paths else ["default"]

class ProcessState(TypedDict):
    text: str
    needs_translation: bool
    needs_summary: bool
    needs_analysis: bool
    results: Annotated[list, operator.add]

def translator(state: ProcessState):
    return {"results": ["翻译完成"]}

def summarizer(state: ProcessState):
    return {"results": ["摘要完成"]}

def analyzer(state: ProcessState):
    return {"results": ["分析完成"]}

def merge_results(state: ProcessState):
    return {"final": ", ".join(state["results"])}

workflow = StateGraph(ProcessState)
workflow.add_node("router", lambda s: s)  # 路由节点
workflow.add_node("translator", translator)
workflow.add_node("summarizer", summarizer)
workflow.add_node("analyzer", analyzer)
workflow.add_node("merge", merge_results)

workflow.add_edge(START, "router")
workflow.add_conditional_edges(
    "router",
    parallel_router
)
workflow.add_edge("translator", "merge")
workflow.add_edge("summarizer", "merge")
workflow.add_edge("analyzer", "merge")
workflow.add_edge("merge", END)
```

### 4.4 Command 动态路由

使用 `Command` 对象可以在节点内部直接控制路由：

```python
from langgraph.types import Command
from typing import Literal

def smart_node(state: State) -> Command[Literal["node_b", "node_c"]]:
    """使用 Command 进行动态路由"""
    
    # 执行业务逻辑
    processed_value = state["value"] * 2
    
    # 决定路由并同时更新状态
    if processed_value > 100:
        return Command(
            update={"value": processed_value, "status": "high"},
            goto="node_b"  # 路由到 node_b
        )
    else:
        return Command(
            update={"value": processed_value, "status": "normal"},
            goto="node_c"  # 路由到 node_c
        )

# 无需定义条件边，Command 自动处理路由
workflow.add_node("smart", smart_node)
workflow.add_node("node_b", lambda s: {"result": "高值处理"})
workflow.add_node("node_c", lambda s: {"result": "普通处理"})

workflow.add_edge(START, "smart")
# smart 节点会自动路由，无需显式添加条件边
workflow.add_edge("node_b", END)
workflow.add_edge("node_c", END)
```

### 4.5 实战：智能客服路由系统

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict
from typing import Literal, Annotated
import operator

# 状态定义
class CustomerServiceState(TypedDict):
    user_message: str
    intent: str
    priority: str
    response: str
    history: Annotated[list, operator.add]

# 节点定义
def classify_intent(state: CustomerServiceState):
    """分类用户意图"""
    message = state["user_message"].lower()
    
    if "退款" in message or "refund" in message:
        intent = "refund"
        priority = "high"
    elif "投诉" in message or "complaint" in message:
        intent = "complaint"
        priority = "high"
    elif "查询" in message or "query" in message:
        intent = "query"
        priority = "normal"
    elif "建议" in message or "suggestion" in message:
        intent = "suggestion"
        priority = "low"
    else:
        intent = "general"
        priority = "normal"
    
    return {
        "intent": intent,
        "priority": priority,
        "history": [f"分类完成: {intent}"]
    }

def handle_refund(state: CustomerServiceState):
    """处理退款"""
    return {
        "response": "您的退款申请已提交，我们会在24小时内处理。",
        "history": ["退款流程已启动"]
    }

def handle_complaint(state: CustomerServiceState):
    """处理投诉"""
    return {
        "response": "非常抱歉给您带来不便，我们会立即调查并在48小时内回复。",
        "history": ["投诉已记录，转至专员"]
    }

def handle_query(state: CustomerServiceState):
    """处理查询"""
    return {
        "response": "感谢您的查询，相关信息如下：...",
        "history": ["查询已处理"]
    }

def handle_suggestion(state: CustomerServiceState):
    """处理建议"""
    return {
        "response": "感谢您的宝贵建议，我们会认真考虑！",
        "history": ["建议已收录"]
    }

def handle_general(state: CustomerServiceState):
    """通用处理"""
    return {
        "response": "您好，请问有什么可以帮您的？",
        "history": ["通用响应"]
    }

# 路由函数
def intent_router(state: CustomerServiceState) -> Literal[
    "refund", "complaint", "query", "suggestion", "general"
]:
    """根据意图路由"""
    return state["intent"]

# 构建客服系统
customer_service = StateGraph(CustomerServiceState)

# 添加节点
customer_service.add_node("classify", classify_intent)
customer_service.add_node("handle_refund", handle_refund)
customer_service.add_node("handle_complaint", handle_complaint)
customer_service.add_node("handle_query", handle_query)
customer_service.add_node("handle_suggestion", handle_suggestion)
customer_service.add_node("handle_general", handle_general)

# 构建路由
customer_service.add_edge(START, "classify")
customer_service.add_conditional_edges(
    "classify",
    intent_router,
    {
        "refund": "handle_refund",
        "complaint": "handle_complaint",
        "query": "handle_query",
        "suggestion": "handle_suggestion",
        "general": "handle_general"
    }
)

# 所有处理节点都连接到 END
for handler in ["handle_refund", "handle_complaint", "handle_query", 
                "handle_suggestion", "handle_general"]:
    customer_service.add_edge(handler, END)

# 编译
cs_app = customer_service.compile()

# 测试不同类型的消息
test_messages = [
    "我要申请退款",
    "你们的服务太差了，我要投诉",
    "请问我的订单什么时候到？",
    "建议你们增加更多支付方式",
    "你好"
]

for msg in test_messages:
    print(f"\n用户: {msg}")
    result = cs_app.invoke({"user_message": msg})
    print(f"意图: {result['intent']} | 优先级: {result['priority']}")
    print(f"回复: {result['response']}")
    print(f"历史: {result['history']}")
```

---

## 第五部分：并行执行策略

### 5.1 并行节点设计

LangGraph 支持真正的并行执行，提高处理效率。

#### 基础并行执行

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict
from typing import Annotated
import operator

class ParallelState(TypedDict):
    input_data: str
    results: Annotated[list, operator.add]

def process_a(state: ParallelState):
    """处理 A"""
    import time
    time.sleep(1)  # 模拟耗时操作
    return {"results": ["A完成"]}

def process_b(state: ParallelState):
    """处理 B"""
    import time
    time.sleep(1)
    return {"results": ["B完成"]}

def process_c(state: ParallelState):
    """处理 C"""
    import time
    time.sleep(1)
    return {"results": ["C完成"]}

def merge_results(state: ParallelState):
    """合并结果"""
    return {"final": ", ".join(state["results"])}

# 构建并行工作流
workflow = StateGraph(ParallelState)
workflow.add_node("process_a", process_a)
workflow.add_node("process_b", process_b)
workflow.add_node("process_c", process_c)
workflow.add_node("merge", merge_results)

# 从 START 并行启动三个节点
workflow.add_edge(START, "process_a")
workflow.add_edge(START, "process_b")
workflow.add_edge(START, "process_c")

# 三个节点都连接到 merge
workflow.add_edge("process_a", "merge")
workflow.add_edge("process_b", "merge")
workflow.add_edge("process_c", "merge")

workflow.add_edge("merge", END)

app = workflow.compile()

# 执行（三个process会并行运行）
import time
start = time.time()
result = app.invoke({"input_data": "test", "results": []})
end = time.time()

print(f"结果: {result['results']}")
print(f"总耗时: {end - start:.2f}秒")  # 约1秒而不是3秒
```

### 5.2 Send API 详解

**Send API** 是动态创建并行工作节点的强大工具。

#### 基础 Send 用法

```python
from langgraph.types import Send

def fan_out_node(state: State):
    """扇出节点 - 动态创建多个并行任务"""
    items = state["items"]
    
    # 为每个 item 创建一个并行任务
    return [Send("worker", {"item": item}) for item in items]

# worker 节点会被并行调用多次
def worker(state: WorkerState):
    """工作节点 - 处理单个 item"""
    result = process_item(state["item"])
    return {"results": [result]}
```

### 5.3 编排者-工作者模式

这是 Send API 最常用的模式：一个编排者节点分配任务给多个并行工作者。

```python
from langgraph.graph import StateGraph, START, END
from langgraph.types import Send
from typing_extensions import TypedDict
from typing import Annotated, List
import operator

# 主状态
class ReportState(TypedDict):
    topic: str
    sections: List[str]
    completed_sections: Annotated[List[str], operator.add]
    final_report: str

# 工作者状态
class WorkerState(TypedDict):
    section_title: str
    content: str

def orchestrator(state: ReportState):
    """编排者：规划并分配任务"""
    topic = state["topic"]
    
    # 规划报告章节
    sections = [
        "引言",
        "背景介绍",
        "技术细节",
        "应用案例",
        "未来展望",
        "结论"
    ]
    
    # 使用 Send 为每个章节创建工作者
    return [
        Send("worker", {"section_title": section, "topic": topic})
        for section in sections
    ]

def worker(state: WorkerState):
    """工作者：撰写单个章节"""
    section = state["section_title"]
    
    # 模拟章节撰写
    content = f"## {section}\n\n这是关于{section}的内容..."
    
    return {"completed_sections": [content]}

def synthesizer(state: ReportState):
    """合成器：合并所有章节"""
    report = "\n\n".join(state["completed_sections"])
    return {"final_report": report}

# 构建工作流
report_workflow = StateGraph(ReportState)

report_workflow.add_node("orchestrator", orchestrator)
report_workflow.add_node("worker", worker)
report_workflow.add_node("synthesizer", synthesizer)

# 编排流程
report_workflow.add_edge(START, "orchestrator")
report_workflow.add_conditional_edges(
    "orchestrator",
    lambda x: x,  # orchestrator 返回 Send 列表
    ["worker"]
)
report_workflow.add_edge("worker", "synthesizer")
report_workflow.add_edge("synthesizer", END)

report_app = report_workflow.compile()

# 执行
result = report_app.invoke({
    "topic": "量子计算",
    "completed_sections": []
})

print(result["final_report"])
```

### 5.4 控制并发数

```python
# 限制最大并发数
result = app.invoke(
    initial_state,
    config={
        "configurable": {
            "max_concurrency": 5  # 最多同时执行5个节点
        }
    }
)
```

### 5.5 性能优化技巧

#### 技巧 1：异步并行

```python
import asyncio

async def async_worker(state: WorkerState):
    """异步工作节点"""
    await asyncio.sleep(0.1)  # 异步 I/O
    return {"result": "done"}

# 使用 ainvoke 执行异步图
result = await app.ainvoke(initial_state)
```

#### 技巧 2：批处理优化

```python
def batch_worker(state: State):
    """批量处理多个项目"""
    items = state["batch_items"]
    
    # 一次性处理多个 item（更高效）
    results = process_batch(items)
    
    return {"results": results}
```

---

## 第六部分：Checkpointer 持久化

### 6.1 为什么需要 Checkpointer？

Checkpointer 提供状态持久化能力：
- 💾 保存对话历史
- 🔄 支持会话恢复
- 🐛 便于调试和回溯
- ⏱️  实现时间旅行

### 6.2 内存检查点

#### 使用 InMemorySaver

```python
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.graph import StateGraph, START

checkpointer = InMemorySaver()

workflow = StateGraph(State)
# ... 添加节点和边 ...

# 编译时传入 checkpointer
app = workflow.compile(checkpointer=checkpointer)

# 使用 thread_id 标识会话
config = {"configurable": {"thread_id": "conversation_1"}}

# 第一次调用
result1 = app.invoke(
    {"messages": [{"role": "user", "content": "你好"}]},
    config
)

# 第二次调用 - 会记住之前的状态
result2 = app.invoke(
    {"messages": [{"role": "user", "content": "我刚才说了什么？"}]},
    config
)
```

### 6.3 数据库持久化

#### PostgreSQL 持久化

```python
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://user:password@localhost:5432/langchain_db"

with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    # 首次使用需要设置数据库表
    checkpointer.setup()
    
    app = workflow.compile(checkpointer=checkpointer)
    
    config = {"configurable": {"thread_id": "user_123"}}
    
    result = app.invoke(initial_state, config)
```

#### 异步 PostgreSQL

```python
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

async with AsyncPostgresSaver.from_conn_string(DB_URI) as checkpointer:
    app = workflow.compile(checkpointer=checkpointer)
    
    config = {"configurable": {"thread_id": "user_123"}}
    
    async for chunk in app.astream(initial_state, config):
        print(chunk)
```

### 6.4 查看和管理检查点

#### 获取状态快照

```python
# 获取最新状态
config = {"configurable": {"thread_id": "conversation_1"}}
state = app.get_state(config)

print("当前状态:", state.values)
print("检查点ID:", state.config["configurable"]["checkpoint_id"])

# 获取特定检查点的状态
config_with_checkpoint = {
    "configurable": {
        "thread_id": "conversation_1",
        "checkpoint_id": "1ef663ba-28fe-6528-8002-5a559208592c"
    }
}
historical_state = app.get_state(config_with_checkpoint)
```

#### 时间旅行调试

```python
# 更新历史状态
new_config = app.update_state(
    config,
    values={"some_field": "new_value"}
)

print("新检查点ID:", new_config["configurable"]["checkpoint_id"])
```

---

## 🎓 第二阶段完成！

恭喜你完成了 LangGraph 与工作流编排的学习！

### 你已经掌握：

✅ **LangGraph 核心概念**
- LangGraph vs Agent 的区别
- Graph、State、Node、Edge 的理解
- 图的构建和编译

✅ **StateGraph 状态管理**
- State 定义和类型系统
- Annotated 和 Reducer 机制
- 状态更新和传递

✅ **节点与边**
- 节点函数编写
- 普通边和条件边
- 控制流设计模式

✅ **条件路由**
- 路由函数设计
- 多路径路由
- Command 动态路由

✅ **并行执行**
- 并行节点设计
- Send API 使用
- 编排者-工作者模式

✅ **Checkpointer 持久化**
- 内存和数据库持久化
- 状态管理和恢复
- 时间旅行调试

### 📌 下一阶段预告

**第三阶段：高级特性与实战项目**

你将学习：
1. 🧰 Tools 工具系统深入
2. 🔍 RAG 检索增强生成
3. 📨 Streaming 流式处理
4. 🎛️  Middleware 中间件系统
5. 📝 Structured Output 结构化输出
6. 🚀 完整的生产级项目

准备好进入下一阶段了吗？

