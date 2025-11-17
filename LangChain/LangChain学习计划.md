# LangChain 全面学习计划

> 基于 LangChain 最新版本整理 | 更新时间：2025年1月

## 📚 目录
1. [LangChain 概述](#1-langchain-概述)
2. [核心架构](#2-核心架构)
3. [模型集成 (Models)](#3-模型集成-models)
4. [Agents 代理系统](#4-agents-代理系统)
5. [LangGraph 状态工作流](#5-langgraph-状态工作流)
6. [Memory 记忆系统](#6-memory-记忆系统)
7. [Tools 工具系统](#7-tools-工具系统)
8. [RAG 检索增强生成](#8-rag-检索增强生成)
9. [Streaming 流式处理](#9-streaming-流式处理)
10. [Middleware 中间件系统](#10-middleware-中间件系统)
11. [Structured Output 结构化输出](#11-structured-output-结构化输出)
12. [Prompt Engineering 提示工程](#12-prompt-engineering-提示工程)
13. [Observability 可观测性](#13-observability-可观测性)
14. [部署与生产](#14-部署与生产)
15. [最佳实践](#15-最佳实践)

---

## 1. LangChain 概述

### 1.1 什么是 LangChain？
LangChain 是一个用于开发由大语言模型（LLM）驱动的应用程序的框架。它简化了 LLM 应用生命周期的每个阶段，提供开源组件和第三方集成。

### 1.2 核心特性
- **组件化设计**：模块化的构建块，可灵活组合
- **链式调用**：将多个组件串联成复杂的工作流
- **Agent 系统**：智能代理可自主决策和使用工具
- **记忆管理**：短期和长期记忆系统
- **多模型支持**：统一接口支持各种 LLM 提供商
- **可观测性**：内置的追踪和监控能力

### 1.3 应用场景
- **问答系统**：基于文档的智能问答
- **聊天机器人**：多轮对话和上下文理解
- **文档分析**：自动摘要、分类和信息提取
- **代码助手**：代码生成、解释和调试
- **数据库查询**：自然语言转 SQL
- **工作流自动化**：多步骤任务编排
- **知识管理**：RAG 系统和知识库

### 1.4 安装

```bash
# 核心库
pip install langchain langchain-core

# 社区集成
pip install langchain-community

# 特定模型提供商
pip install langchain-openai      # OpenAI
pip install langchain-anthropic   # Anthropic
pip install langchain-google      # Google

# 文本分割和处理
pip install langchain-text-splitters

# LangGraph（状态工作流）
pip install langgraph

# LangSmith（监控和追踪）
pip install langsmith
```

---

## 2. 核心架构

### 2.1 整体架构层次

```
┌─────────────────────────────────────────────┐
│         应用层 (Applications)                │
│   聊天机器人、问答系统、自动化工具等            │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      LangGraph (状态工作流编排)               │
│   StateGraph、条件路由、并行执行               │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│       LangChain Agents (代理层)              │
│   create_agent、工具调用、决策逻辑             │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│       核心组件层 (Core Components)            │
│  Models | Prompts | Tools | Memory           │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      基础设施层 (Infrastructure)              │
│  LangSmith监控 | 向量存储 | 检查点             │
└─────────────────────────────────────────────┘
```

### 2.2 核心概念

#### 2.2.1 Runnable 接口
所有 LangChain 组件都实现 `Runnable` 接口，提供统一的调用方式：

```python
# 所有组件都支持这些方法
result = component.invoke(input)           # 单次同步调用
results = component.batch([input1, input2]) # 批量处理
for chunk in component.stream(input):      # 流式输出
    print(chunk)
```

**Runnable 的核心方法**：
- `invoke()`: 单次调用，返回完整结果
- `batch()`: 批量处理多个输入
- `stream()`: 流式输出，逐步返回结果
- `batch_as_completed()`: 异步批处理，按完成顺序返回

#### 2.2.2 LCEL (LangChain Expression Language)
使用 `|` 操作符链接组件，创建处理流水线：

```python
chain = prompt | model | output_parser
result = chain.invoke({"input": "用户输入"})
```

### 2.3 主要组件

| 组件 | 功能 | 示例 |
|-----|------|------|
| **Models** | LLM 和 Chat 模型 | ChatOpenAI, ChatAnthropic |
| **Prompts** | 提示模板 | PromptTemplate, ChatPromptTemplate |
| **Output Parsers** | 输出解析 | StrOutputParser, JsonOutputParser |
| **Retrievers** | 文档检索 | VectorStoreRetriever |
| **Tools** | 外部工具 | @tool 装饰器 |
| **Agents** | 智能代理 | create_agent() |
| **Memory** | 记忆系统 | Checkpointer, Store |

---

## 3. 模型集成 (Models)

### 3.1 初始化模型

#### 方式 1：字符串标识符（自动推断提供商）
```python
from langchain.agents import create_agent

agent = create_agent(
    "gpt-4o",  # 自动识别为 OpenAI 模型
    tools=tools
)
```

#### 方式 2：显式实例化
```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    max_tokens=2000,
    timeout=30
)
```

#### 方式 3：统一初始化接口
```python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    model="gpt-4o-mini",
    model_provider="openai",
    temperature=0.5
)
```

### 3.2 模型调用

#### 基本调用
```python
# 文本字符串
response = model.invoke("写一首关于春天的俳句")

# 消息列表
from langchain_core.messages import HumanMessage, SystemMessage

response = model.invoke([
    SystemMessage(content="你是一个有帮助的助手"),
    HumanMessage(content="解释量子计算")
])
```

#### 批量处理
```python
# 同步批处理
responses = model.batch([
    "为什么鹦鹉有彩色羽毛？",
    "飞机如何飞行？",
    "什么是量子计算？"
])

# 异步批处理（按完成顺序）
for response in model.batch_as_completed([...]):
    print(response)
```

### 3.3 工具绑定

```python
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """获取指定位置的天气信息"""
    return f"{location}的天气：晴天，22°C"

# 将工具绑定到模型
model_with_tools = model.bind_tools([get_weather])

# 模型可以决定是否调用工具
response = model_with_tools.invoke("纽约的天气怎么样？")
print(response.tool_calls)  # 查看工具调用
```

### 3.4 流式输出

```python
# 流式获取 token
for chunk in model.stream("讲一个长故事"):
    print(chunk.content, end="", flush=True)

# 流式工具调用
for chunk in model_with_tools.stream("波士顿和东京的天气？"):
    for tool_chunk in chunk.tool_call_chunks:
        if name := tool_chunk.get("name"):
            print(f"工具: {name}")
        if args := tool_chunk.get("args"):
            print(f"参数: {args}")
```

### 3.5 Token 使用追踪

```python
from langchain_core.callbacks import get_usage_metadata_callback

model_1 = init_chat_model("gpt-4o-mini")
model_2 = init_chat_model("claude-haiku-4-5-20251001")

with get_usage_metadata_callback() as cb:
    model_1.invoke("你好")
    model_2.invoke("你好")
    
    # 查看每个模型的 token 使用情况
    print(cb.usage_metadata)
    # {
    #   "gpt-4o-mini": {"input_tokens": 8, "output_tokens": 10, ...},
    #   "claude-haiku": {"input_tokens": 8, "output_tokens": 21, ...}
    # }
```

---

## 4. Agents 代理系统

### 4.1 什么是 Agent？

Agent 是能够使用工具、做出决策并执行多步骤任务的智能系统。

**Agent 的核心能力**：
- 🤔 **推理**：理解用户意图和任务需求
- 🔧 **工具使用**：选择和调用合适的工具
- 🔄 **迭代执行**：根据结果调整策略
- 💬 **对话管理**：维护上下文和多轮交互
- 📊 **结果综合**：整合多步骤结果

### 4.2 创建基础 Agent

```python
from langchain.agents import create_agent
from langchain.tools import tool

@tool
def search(query: str) -> str:
    """搜索信息"""
    return f"搜索结果：{query}"

@tool
def get_weather(location: str) -> str:
    """获取天气信息"""
    return f"{location}的天气：晴天，72°F"

# 创建 Agent
agent = create_agent(
    model="gpt-4o",
    tools=[search, get_weather],
    system_prompt="你是一个有帮助的助手，可以搜索信息和查询天气"
)

# 调用 Agent
result = agent.invoke({
    "messages": [{"role": "user", "content": "旧金山的天气如何？"}]
})

print(result["messages"][-1].content)
```

### 4.3 ReACT 工作模式

Agent 遵循 **Reasoning（推理）→ Action（行动）→ Observation（观察）** 循环：

```
用户输入: "搜索无线耳机并检查库存"

┌─────────────────────────────────────┐
│ 推理: 我需要先搜索产品              │
└─────────────────────────────────────┘
          ↓
┌─────────────────────────────────────┐
│ 行动: search_products("无线耳机")   │
└─────────────────────────────────────┘
          ↓
┌─────────────────────────────────────┐
│ 观察: "找到 Sony WH-1000XM5"        │
└─────────────────────────────────────┘
          ↓
┌─────────────────────────────────────┐
│ 推理: 现在检查这个产品的库存          │
└─────────────────────────────────────┘
          ↓
┌─────────────────────────────────────┐
│ 行动: check_inventory("WH-1000XM5") │
└─────────────────────────────────────┘
          ↓
┌──────────────────────────────────────┐
│ 观察: "库存 10 件"                   │
└──────────────────────────────────────┘
          ↓
┌──────────────────────────────────────┐
│ 最终回答: "找到 Sony WH-1000XM5，    │
│           当前库存 10 件"            │
└──────────────────────────────────────┘
```

### 4.4 Context 上下文系统

使用 Context 传递运行时信息：

```python
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime

@dataclass
class Context:
    user_id: str
    user_location: str

@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """获取用户位置"""
    return runtime.context.user_location

agent = create_agent(
    model="gpt-4o",
    tools=[get_user_location],
    context_schema=Context
)

# 调用时传递 Context
result = agent.invoke(
    {"messages": [{"role": "user", "content": "我在哪里？"}]},
    context=Context(user_id="user_123", user_location="旧金山")
)
```

---

## 5. LangGraph 状态工作流

### 5.1 什么是 LangGraph？

**LangGraph** 是一个低级编排框架，用于构建、管理和部署长期运行的有状态 Agent 和工作流。

**核心特性**：
- 🔄 **持久化执行**：状态保存和恢复
- 👤 **人机交互**：人工审核和干预
- 🧠 **全面记忆**：跨会话的状态管理
- 🚀 **生产就绪**：可扩展的部署能力

### 5.2 核心概念

#### State（状态）
```python
from typing_extensions import TypedDict
from typing import Annotated
import operator

class State(TypedDict):
    messages: Annotated[list, operator.add]  # 累加消息
    topic: str
    output: str
```

#### Node（节点）
```python
def generate_joke(state: State):
    """生成笑话"""
    msg = llm.invoke(f"写一个关于{state['topic']}的笑话")
    return {"joke": msg.content}
```

#### Edge（边）
- **普通边**：固定的节点转换
- **条件边**：基于状态的动态路由

### 5.3 简单链式工作流

```python
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    topic: str
    joke: str
    improved_joke: str

# 创建 Graph
workflow = StateGraph(State)

# 添加节点
workflow.add_node("generate_joke", generate_joke)
workflow.add_node("improve_joke", improve_joke)
workflow.add_node("polish_joke", polish_joke)

# 添加边
workflow.add_edge(START, "generate_joke")
workflow.add_edge("generate_joke", "improve_joke")
workflow.add_edge("improve_joke", "polish_joke")
workflow.add_edge("polish_joke", END)

# 编译并执行
chain = workflow.compile()
result = chain.invoke({"topic": "猫"})
```

### 5.4 并行执行工作流

```python
# 并行执行多个 LLM 调用
async def call_llm1(state: State):
    msg = await llm.invoke(f"写一个关于{state['topic']}的笑话")
    return {"joke": msg.content}

async def call_llm2(state: State):
    msg = await llm.invoke(f"写一个关于{state['topic']}的故事")
    return {"story": msg.content}

# 构建并行工作流
parallel_workflow = StateGraph(State)
parallel_workflow.add_node("llm1", call_llm1)
parallel_workflow.add_node("llm2", call_llm2)
parallel_workflow.add_node("aggregator", aggregator)

# 并行启动
parallel_workflow.add_edge(START, "llm1")
parallel_workflow.add_edge(START, "llm2")

# 汇聚到聚合器
parallel_workflow.add_edge("llm1", "aggregator")
parallel_workflow.add_edge("llm2", "aggregator")
parallel_workflow.add_edge("aggregator", END)
```

---

## 6. Memory 记忆系统

### 6.1 短期记忆（对话历史）

使用 **Checkpointer** 管理会话状态：

```python
from langchain.agents import create_agent
from langgraph.checkpoint.memory import InMemorySaver

agent = create_agent(
    "gpt-4o",
    tools=[],
    checkpointer=InMemorySaver()
)

# 第一轮对话
agent.invoke(
    {"messages": [{"role": "user", "content": "你好！我叫 Bob。"}]},
    {"configurable": {"thread_id": "1"}}
)

# 第二轮对话（记住上下文）
agent.invoke(
    {"messages": [{"role": "user", "content": "我叫什么名字？"}]},
    {"configurable": {"thread_id": "1"}}
)
# 输出: "你的名字是 Bob"
```

### 6.2 长期记忆（跨会话存储）

使用 **Store** 持久化数据：

```python
from langgraph.store.memory import InMemoryStore
from langchain.tools import tool, ToolRuntime
from dataclasses import dataclass

@dataclass
class Context:
    user_id: str

store = InMemoryStore()

# 存储用户信息
store.put(
    ("users",),
    "user_123",
    {"name": "John Smith", "language": "English"}
)

@tool
def get_user_info(runtime: ToolRuntime[Context]) -> str:
    """查询用户信息"""
    user_id = runtime.context.user_id
    user_info = runtime.store.get(("users",), user_id)
    return str(user_info.value) if user_info else "未知用户"

agent = create_agent(
    model="gpt-4o",
    tools=[get_user_info],
    store=store,
    context_schema=Context
)
```

### 6.3 记忆系统对比

| 记忆类型 | 实现方式 | 生命周期 | 使用场景 |
|---------|---------|---------|---------|
| **短期记忆** | Checkpointer | 单个会话 | 对话历史、上下文连贯性 |
| **长期记忆** | Store | 跨会话持久化 | 用户偏好、知识积累 |
| **工作记忆** | State | 单次执行 | 中间结果、临时数据 |

---

## 7. Tools 工具系统

### 7.1 定义工具

#### 基础工具
```python
from langchain.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """搜索客户数据库中匹配查询的记录。
    
    Args:
        query: 要查找的搜索词
        limit: 返回的最大结果数
    """
    return f"找到 {limit} 条关于 '{query}' 的结果"
```

#### 复杂输入模式（Pydantic）
```python
from pydantic import BaseModel, Field
from typing import Literal

class WeatherInput(BaseModel):
    location: str = Field(description="城市名称或坐标")
    units: Literal["celsius", "fahrenheit"] = Field(default="celsius")
    include_forecast: bool = Field(default=False)

@tool(args_schema=WeatherInput)
def get_weather(location: str, units: str = "celsius", 
                include_forecast: bool = False) -> str:
    """获取天气信息"""
    temp = 22 if units == "celsius" else 72
    result = f"{location}当前温度：{temp}°{units[0].upper()}"
    if include_forecast:
        result += "\n未来5天：晴天"
    return result
```

### 7.2 工具中的运行时信息

```python
@tool
def get_weather(city: str, runtime: ToolRuntime) -> str:
    """获取天气信息"""
    writer = runtime.stream_writer
    
    # 流式输出进度
    writer(f"正在查询城市：{city}")
    writer(f"已获取 {city} 的数据")
    
    return f"{city}永远是晴天！"
```

---

## 8. RAG 检索增强生成

### 8.1 什么是 RAG？

**RAG (Retrieval-Augmented Generation)** 通过检索外部知识来增强 LLM 的响应能力。

**核心流程**：
```
用户查询 → 向量检索 → 相关文档 → 注入上下文 → LLM 生成回答
```

### 8.2 基础 RAG 实现

```python
from langchain_core.vectorstores import InMemoryVectorStore
from langchain_community.document_loaders import WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter

# 1. 加载文档
loader = WebBaseLoader("https://example.com/blog")
docs = loader.load()

# 2. 分割文档
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(docs)

# 3. 创建向量存储
vector_store = InMemoryVectorStore.from_documents(
    documents=splits,
    embedding=embeddings
)

# 4. 创建检索器
retriever = vector_store.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)
```

### 8.3 RAG Agent

```python
from langchain.tools import tool

@tool(response_format="content_and_artifact")
def retrieve_context(query: str):
    """检索相关文档"""
    retrieved_docs = vector_store.similarity_search(query, k=2)
    serialized = "\n\n".join(
        f"来源: {doc.metadata}\n内容: {doc.page_content}"
        for doc in retrieved_docs
    )
    return serialized, retrieved_docs

# 创建 RAG Agent
tools = [retrieve_context]
prompt = "你可以使用检索工具来查找博客内容，以帮助回答用户问题。"

agent = create_agent(model, tools, system_prompt=prompt)

# 查询
result = agent.invoke({
    "messages": [{"role": "user", "content": "什么是任务分解？"}]
})
```

---

## 9. Streaming 流式处理

### 9.1 流式模式

#### 9.1.1 messages 模式（Token 流）
```python
agent = create_agent("gpt-4o", tools=[get_weather])

for token, metadata in agent.stream(
    {"messages": [{"role": "user", "content": "旧金山的天气？"}]},
    stream_mode="messages"
):
    print(f"节点: {metadata['langgraph_node']}")
    print(f"内容: {token.content}")
```

#### 9.1.2 updates 模式（步骤更新）
```python
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "旧金山的天气？"}]},
    stream_mode="updates"
):
    for step, data in chunk.items():
        print(f"步骤: {step}")
        print(f"消息: {data['messages'][-1].content}")
```

#### 9.1.3 custom 模式（自定义更新）
```python
from langgraph.config import get_stream_writer

@tool
def get_weather(city: str) -> str:
    """获取天气"""
    writer = get_stream_writer()
    writer(f"正在查询 {city}")
    writer(f"已获取 {city} 的数据")
    return f"{city}永远是晴天！"

for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "旧金山的天气？"}]},
    stream_mode="custom"
):
    print(chunk)
```

---

## 10. Middleware 中间件系统

### 10.1 什么是 Middleware？

Middleware 允许在 Agent 执行的各个阶段插入自定义逻辑：
- 🔄 **before_model**: 模型调用前
- 🔄 **after_model**: 模型调用后
- 🔧 **wrap_tool_call**: 工具调用包装
- 💬 **dynamic_prompt**: 动态提示词

### 10.2 动态提示词

```python
from langchain.agents.middleware import dynamic_prompt, ModelRequest

@dynamic_prompt
def user_role_prompt(request: ModelRequest) -> str:
    """基于用户角色生成提示词"""
    user_role = request.runtime.context.get("user_role", "user")
    base_prompt = "你是一个有帮助的助手。"
    
    if user_role == "expert":
        return f"{base_prompt} 提供详细的技术回答。"
    elif user_role == "beginner":
        return f"{base_prompt} 用简单的语言解释，避免术语。"
    
    return base_prompt

agent = create_agent(
    model="gpt-4o",
    tools=[],
    middleware=[user_role_prompt]
)
```

### 10.3 工具重试

```python
from langchain.agents.middleware import ToolRetryMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[search_tool, database_tool],
    middleware=[
        ToolRetryMiddleware(
            max_retries=3,
            backoff_factor=2.0,
            initial_delay=1.0
        )
    ]
)
```

### 10.4 工具调用限制

```python
from langchain.agents.middleware import ToolCallLimitMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[search_tool],
    middleware=[
        # 全局限制
        ToolCallLimitMiddleware(thread_limit=20, run_limit=10),
        # 工具特定限制
        ToolCallLimitMiddleware(
            tool_name="search",
            thread_limit=5,
            run_limit=3
        )
    ]
)
```

---

## 11. Structured Output 结构化输出

### 11.1 为什么需要结构化输出？

让 LLM 输出符合预定义模式的数据，便于后续处理和集成。

### 11.2 使用 Pydantic 模型

```python
from pydantic import BaseModel, Field

class Movie(BaseModel):
    """电影信息"""
    title: str = Field(description="电影标题")
    year: int = Field(description="上映年份")
    director: str = Field(description="导演")
    rating: float = Field(description="评分（满分10分）")

# 绑定结构化输出
model_with_structure = model.with_structured_output(Movie)

response = model_with_structure.invoke("提供电影《盗梦空间》的详细信息")
print(response)
# Movie(title="盗梦空间", year=2010, director="克里斯托弗·诺兰", rating=8.8)
```

### 11.3 在 Agent 中使用

```python
from langchain.agents.structured_output import ToolStrategy

class ContactInfo(BaseModel):
    name: str
    email: str
    phone: str

agent = create_agent(
    model="gpt-4o",
    tools=[search_tool],
    response_format=ToolStrategy(ContactInfo)
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "提取联系信息：John Doe, john@example.com, (555) 123-4567"}]
})

print(result["structured_response"])
# ContactInfo(name='John Doe', email='john@example.com', phone='(555) 123-4567')
```

---

## 12. Prompt Engineering 提示工程

### 12.1 系统提示词

```python
SYSTEM_PROMPT = """你是一个专业的 SQL 查询助手。
给定用户问题，创建语法正确的 {dialect} 查询。

规则：
1. 始终限制结果为 {top_k} 条
2. 先查看数据库中有哪些表
3. 查询最相关表的模式
4. 执行前仔细检查查询
5. 不要执行 DML 语句（INSERT、UPDATE、DELETE、DROP）
"""

agent = create_agent(
    model="gpt-4o",
    tools=sql_tools,
    system_prompt=SYSTEM_PROMPT.format(dialect="PostgreSQL", top_k=5)
)
```

### 12.2 动态上下文注入

```python
from langchain.agents.middleware import dynamic_prompt

@dynamic_prompt
def prompt_with_context(request: ModelRequest) -> str:
    """注入检索上下文"""
    last_query = request.state["messages"][-1].text
    retrieved_docs = vector_store.similarity_search(last_query)
    
    docs_content = "\n\n".join(doc.page_content for doc in retrieved_docs)
    
    return (
        "你是一个有帮助的助手。使用以下上下文回答问题："
        f"\n\n{docs_content}"
    )

agent = create_agent(model, tools=[], middleware=[prompt_with_context])
```

---

## 13. Observability 可观测性

### 13.1 LangSmith 集成

```bash
# 设置环境变量
export LANGSMITH_API_KEY="your-api-key"
export LANGSMITH_TRACING="true"
export LANGSMITH_PROJECT="my-project"
```

### 13.2 添加追踪元数据

```python
response = agent.invoke(
    {"messages": [{"role": "user", "content": "发送欢迎邮件"}]},
    config={
        "tags": ["production", "email-assistant", "v1.0"],
        "metadata": {
            "user_id": "user_123",
            "session_id": "session_456",
            "environment": "production"
        }
    }
)
```

### 13.3 选择性追踪

```python
import langsmith as ls

# 只追踪这段代码
with ls.tracing_context(enabled=True):
    agent.invoke({"messages": [{"role": "user", "content": "测试"}]})

# 这段不追踪
agent.invoke({"messages": [{"role": "user", "content": "另一个测试"}]})
```

---

## 14. 部署与生产

### 14.1 环境配置

```python
import os

# 生产环境配置
os.environ["LANGCHAIN_ENV"] = "production"
os.environ["LANGCHAIN_API_KEY"] = "your-api-key"

# 模型配置
model = ChatOpenAI(
    model="gpt-4o",
    temperature=0.1,  # 降低随机性
    max_retries=3,
    timeout=30
)
```

### 14.2 持久化 Checkpointer

```python
from langgraph.checkpoint.postgres import PostgresSaver

# 使用 PostgreSQL 持久化
checkpointer = PostgresSaver.from_conn_string(
    "postgresql://user:pass@localhost:5432/langchain_db"
)

agent = create_agent(
    model="gpt-4o",
    tools=tools,
    checkpointer=checkpointer
)
```

### 14.3 错误处理

```python
from langchain_core.exceptions import LangChainException

try:
    result = agent.invoke({"messages": [...]})
except LangChainException as e:
    logger.error(f"Agent 执行失败: {e}")
    # 降级策略
    result = fallback_response()
```

---

## 15. 最佳实践

### 15.1 设计原则

✅ **Do（推荐）**：
- 使用清晰的工具描述和类型提示
- 为 Agent 提供具体的系统提示词
- 使用持久化存储（生产环境）
- 添加追踪和监控
- 实施错误处理和重试机制
- 限制工具调用次数防止循环
- 使用结构化输出确保数据一致性

❌ **Don't（不推荐）**：
- 模糊的工具描述
- 过于复杂的单个工具
- 在生产环境使用内存存储
- 忽略错误处理
- 无限制的工具调用
- 硬编码 API 密钥

### 15.2 性能优化

```python
# 1. 使用批处理
responses = model.batch([query1, query2, query3])

# 2. 并行执行
from langgraph.graph import StateGraph
# 使用 add_edge 从 START 到多个节点实现并行

# 3. 缓存提示词
from langchain_anthropic.middleware import AnthropicPromptCachingMiddleware

agent = create_agent(
    model=ChatAnthropic(model="claude-sonnet-4-5-20250929"),
    system_prompt=LONG_PROMPT,
    middleware=[AnthropicPromptCachingMiddleware(ttl="5m")]
)
```

### 15.3 安全考虑

```python
# 1. 不要硬编码密钥
api_key = os.environ.get("OPENAI_API_KEY")

# 2. 验证工具输入
@tool
def execute_query(query: str) -> str:
    """执行数据库查询"""
    # 验证查询，防止注入
    if any(keyword in query.upper() for keyword in ["DROP", "DELETE", "UPDATE"]):
        return "不允许的操作"
    return db.execute(query)

# 3. 限制输出长度
model = ChatOpenAI(
    model="gpt-4o",
    max_tokens=2000  # 限制输出
)
```

### 15.4 测试策略

```python
# 1. 单元测试工具
def test_search_tool():
    result = search_database("test query", limit=5)
    assert "test query" in result

# 2. 集成测试 Agent
def test_agent_response():
    result = agent.invoke({
        "messages": [{"role": "user", "content": "测试查询"}]
    })
    assert result["messages"][-1].content

# 3. 使用 LLM 模拟器测试
from langchain.agents.middleware import LLMToolEmulator

test_agent = create_agent(
    model="gpt-4o",
    tools=[get_weather, search],
    middleware=[LLMToolEmulator()]  # 模拟工具调用
)
```

---

## 📖 学习资源

### 官方文档
- [LangChain Python 文档](https://python.langchain.com/)
- [LangChain JavaScript 文档](https://js.langchain.com/)
- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [LangSmith 文档](https://docs.smith.langchain.com/)

### 社区资源
- [GitHub 仓库](https://github.com/langchain-ai/langchain)
- [Discord 社区](https://discord.gg/langchain)
- [Twitter](https://twitter.com/LangChainAI)

### 示例项目
- RAG 问答系统
- SQL 查询 Agent
- 多 Agent 协作系统
- 文档分析工具
- 代码助手

---

## 🎯 学习路线建议

### 第一阶段：基础（1-2周）
1. 理解 LangChain 核心概念
2. 学习模型初始化和调用
3. 掌握基础工具定义
4. 创建简单的 Agent

### 第二阶段：进阶（2-3周）
1. 深入 LangGraph 工作流
2. 掌握记忆系统（短期+长期）
3. 实现 RAG 系统
4. 学习 Middleware 机制

### 第三阶段：高级（3-4周）
1. 多 Agent 协作
2. 结构化输出和数据验证
3. 流式处理和实时交互
4. 可观测性和监控

### 第四阶段：生产（持续）
1. 性能优化
2. 错误处理和容错
3. 安全性加固
4. 部署和运维

---

## 🚀 快速开始示例

```python
from langchain.agents import create_agent
from langchain.tools import tool
from langgraph.checkpoint.memory import InMemorySaver

# 定义工具
@tool
def get_weather(city: str) -> str:
    """获取城市天气"""
    return f"{city}的天气：晴天，22°C"

# 创建 Agent
agent = create_agent(
    model="gpt-4o",
    tools=[get_weather],
    system_prompt="你是一个友好的天气助手",
    checkpointer=InMemorySaver()
)

# 运行 Agent
result = agent.invoke(
    {"messages": [{"role": "user", "content": "北京的天气怎么样？"}]},
    {"configurable": {"thread_id": "session_1"}}
)

print(result["messages"][-1].content)
```

---

**🎉 恭喜你完成 LangChain 学习计划！现在开始构建你的 AI 应用吧！**

