# 第一阶段：LangChain 基础入门（详细讲解）

> **学习目标**：理解 LangChain 核心概念、掌握模型初始化和调用、学会定义工具、创建简单的 Agent
> 
> **预计学习时间**：1-2周
> 
> **前置要求**：Python 基础、了解 API 调用、基本的异步编程概念

---

## 📚 第一阶段学习大纲

1. [LangChain 概述](#第一部分langchain-概述)
   - 什么是 LangChain？
   - 为什么需要 LangChain？
   - LangChain 的核心价值
   - 应用场景详解
   
2. [安装与环境配置](#第二部分安装与环境配置)
   - 基础安装
   - 各模型提供商集成
   - 环境变量配置
   - 验证安装

3. [核心架构深入理解](#第三部分核心架构深入理解)
   - Runnable 接口详解
   - LCEL 表达式语言
   - 链式调用机制
   - 组件化设计思想

4. [模型集成完全指南](#第四部分模型集成完全指南)
   - 初始化模型的三种方式
   - 各主流模型提供商配置
   - 模型调用详解
   - 参数调优

5. [创建第一个 Agent](#第五部分创建第一个-agent)
   - Agent 基本概念
   - 简单 Agent 创建
   - Agent 执行流程
   - 实战案例

---

## 第一部分：LangChain 概述

### 1.1 什么是 LangChain？

**LangChain** 是一个专为开发 **大语言模型（LLM）驱动的应用程序** 而设计的框架。它不是一个 LLM，而是一个**编排框架**，帮助开发者将 LLM 与其他工具、数据源和服务集成在一起。

#### 核心理念

```
传统应用              LLM 应用（使用 LangChain）
────────────         ──────────────────────────
用户输入              用户输入
  ↓                     ↓
固定逻辑             智能推理 (LLM)
  ↓                     ↓
数据库查询           工具调用 (Tools)
  ↓                     ↓
返回结果             外部数据 (RAG)
                        ↓
                     综合回答
```

**传统开发 vs LangChain 开发**：

| 传统方式 | LangChain 方式 |
|---------|---------------|
| 硬编码业务逻辑 | LLM 动态推理 |
| 固定的 if-else | Agent 自主决策 |
| 单一数据源 | 多数据源集成（RAG）|
| 无上下文 | 完整的记忆系统 |
| 难以维护 | 模块化可扩展 |

### 1.2 为什么需要 LangChain？

#### 问题 1：LLM 单独使用的局限性

```python
# ❌ 直接使用 OpenAI API 的问题
import openai

# 问题 1: 没有记忆，每次都是新对话
response1 = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "我叫 Alice"}]
)

# 下一次调用，模型不记得用户是 Alice
response2 = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "我叫什么名字？"}]
)
# 回答：我不知道你的名字

# 问题 2: 无法访问外部数据
response3 = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "2024年11月的新闻有哪些？"}]
)
# 回答：我的知识截止到 2023 年...

# 问题 3: 无法调用工具
response4 = openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "帮我发送一封邮件"}]
)
# 回答：我是 AI，无法发送邮件（只能提供建议）
```

#### LangChain 的解决方案

```python
# ✅ 使用 LangChain 解决这些问题
from langchain.agents import create_agent
from langchain.tools import tool
from langgraph.checkpoint.memory import InMemorySaver

# 解决问题 1: 添加记忆系统
checkpointer = InMemorySaver()

# 解决问题 2: 定义检索工具（访问外部数据）
@tool
def search_news(query: str) -> str:
    """搜索最新新闻"""
    # 调用新闻 API
    return "2024年11月最新新闻：..."

# 解决问题 3: 定义功能工具
@tool
def send_email(to: str, subject: str, body: str) -> str:
    """发送邮件"""
    # 调用邮件服务 API
    return f"邮件已发送到 {to}"

# 创建具备所有能力的 Agent
agent = create_agent(
    model="gpt-4",
    tools=[search_news, send_email],
    checkpointer=checkpointer  # 记忆系统
)

# 第一次对话
agent.invoke(
    {"messages": [{"role": "user", "content": "我叫 Alice"}]},
    {"configurable": {"thread_id": "conversation_1"}}
)

# 第二次对话 - Agent 会记住 Alice
agent.invoke(
    {"messages": [{"role": "user", "content": "我叫什么名字？"}]},
    {"configurable": {"thread_id": "conversation_1"}}
)
# 回答：你的名字是 Alice

# Agent 可以搜索新闻
agent.invoke(
    {"messages": [{"role": "user", "content": "2024年11月的新闻"}]},
    {"configurable": {"thread_id": "conversation_1"}}
)
# Agent 会调用 search_news 工具

# Agent 可以发送邮件
agent.invoke(
    {"messages": [{"role": "user", "content": "发邮件给 bob@example.com"}]},
    {"configurable": {"thread_id": "conversation_1"}}
)
# Agent 会调用 send_email 工具
```

### 1.3 LangChain 的核心价值

#### 价值 1：统一的抽象层

LangChain 提供统一的接口来使用不同的 LLM 提供商：

```python
from langchain.chat_models import init_chat_model

# 使用 OpenAI
model = init_chat_model("gpt-4")
response = model.invoke("你好")

# 切换到 Anthropic Claude（代码完全相同）
model = init_chat_model("claude-sonnet-4-5-20250929")
response = model.invoke("你好")

# 切换到 Google Gemini（代码完全相同）
model = init_chat_model("google_genai:gemini-2.5-flash-lite")
response = model.invoke("你好")

# 💡 关键优势：无需修改代码即可切换模型
```

#### 价值 2：组件化和可复用

```python
# 定义可复用的组件
from langchain.tools import tool

# 组件 1: 天气工具
@tool
def get_weather(city: str) -> str:
    """获取城市天气"""
    return f"{city}的天气：晴天 22°C"

# 组件 2: 翻译工具
@tool
def translate(text: str, target_lang: str) -> str:
    """翻译文本"""
    return f"翻译后的文本：{text} -> {target_lang}"

# 可以在不同的 Agent 中复用这些工具
agent1 = create_agent(
    model="gpt-4",
    tools=[get_weather]  # 只有天气功能
)

agent2 = create_agent(
    model="gpt-4",
    tools=[get_weather, translate]  # 天气 + 翻译
)

# 💡 关键优势：工具可以在多个 Agent 之间共享和复用
```

#### 价值 3：强大的编排能力

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

# 定义工作流状态
class State(TypedDict):
    user_input: str
    weather: str
    translation: str
    final_output: str

# 定义工作流节点
def get_weather_node(state: State):
    weather = f"{state['user_input']} 的天气：晴天"
    return {"weather": weather}

def translate_node(state: State):
    translation = f"翻译：{state['weather']}"
    return {"translation": translation}

def format_output_node(state: State):
    output = f"最终结果：{state['translation']}"
    return {"final_output": output}

# 构建工作流
workflow = StateGraph(State)
workflow.add_node("get_weather", get_weather_node)
workflow.add_node("translate", translate_node)
workflow.add_node("format", format_output_node)

# 定义执行顺序
workflow.add_edge(START, "get_weather")
workflow.add_edge("get_weather", "translate")
workflow.add_edge("translate", "format")
workflow.add_edge("format", END)

# 编译并执行
app = workflow.compile()
result = app.invoke({"user_input": "北京"})
print(result["final_output"])

# 💡 关键优势：可以编排复杂的多步骤工作流
```

### 1.4 应用场景详解

#### 场景 1：智能客服系统

```python
from langchain.agents import create_agent
from langchain.tools import tool
from langgraph.checkpoint.memory import InMemorySaver

# 客服工具
@tool
def query_order(order_id: str) -> str:
    """查询订单状态"""
    return f"订单 {order_id}：已发货，预计明天送达"

@tool
def cancel_order(order_id: str) -> str:
    """取消订单"""
    return f"订单 {order_id} 已取消"

@tool
def search_faq(question: str) -> str:
    """搜索常见问题"""
    return "根据您的问题，建议查看退货政策..."

# 创建客服 Agent
customer_service_agent = create_agent(
    model="gpt-4",
    tools=[query_order, cancel_order, search_faq],
    system_prompt="""你是一个专业的客服助手。
    - 友好、耐心、专业
    - 先理解用户问题，再决定使用哪个工具
    - 提供准确的订单信息
    """,
    checkpointer=InMemorySaver()
)

# 模拟客服对话
response = customer_service_agent.invoke(
    {"messages": [{"role": "user", "content": "我的订单 12345 什么时候到？"}]},
    {"configurable": {"thread_id": "customer_001"}}
)

# Agent 会：
# 1. 理解用户想查询订单状态
# 2. 调用 query_order("12345")
# 3. 返回友好的答案
```

#### 场景 2：代码助手

```python
@tool
def execute_code(code: str) -> str:
    """执行 Python 代码"""
    try:
        exec_result = exec(code)
        return f"执行成功：{exec_result}"
    except Exception as e:
        return f"执行错误：{e}"

@tool
def search_documentation(library: str, topic: str) -> str:
    """搜索库文档"""
    return f"{library} 的 {topic} 文档：..."

code_assistant = create_agent(
    model="gpt-4",
    tools=[execute_code, search_documentation],
    system_prompt="你是一个 Python 编程助手，帮助用户编写和调试代码"
)

# 用户：帮我写一个快速排序
response = code_assistant.invoke({
    "messages": [{"role": "user", "content": "帮我写一个快速排序算法"}]
})
# Agent 会生成代码并可以执行验证
```

#### 场景 3：文档分析（RAG）

```python
from langchain_core.vectorstores import InMemoryVectorStore
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader

# 加载文档
loader = PyPDFLoader("company_handbook.pdf")
documents = loader.load()

# 分割文档
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(documents)

# 创建向量存储
vector_store = InMemoryVectorStore.from_documents(
    documents=splits,
    embedding=embeddings
)

# 定义检索工具
@tool
def search_handbook(query: str) -> str:
    """搜索公司手册"""
    docs = vector_store.similarity_search(query, k=3)
    return "\n\n".join([doc.page_content for doc in docs])

# 创建文档问答 Agent
handbook_qa = create_agent(
    model="gpt-4",
    tools=[search_handbook],
    system_prompt="你是公司手册助手，基于检索到的内容准确回答问题"
)

# 用户提问
response = handbook_qa.invoke({
    "messages": [{"role": "user", "content": "公司的休假政策是什么？"}]
})
# Agent 会：
# 1. 调用 search_handbook 检索相关内容
# 2. 基于检索结果生成回答
```

#### 场景 4：数据分析助手

```python
import pandas as pd

@tool
def load_dataset(file_path: str) -> str:
    """加载数据集"""
    df = pd.read_csv(file_path)
    return f"数据集加载成功，共 {len(df)} 行，{len(df.columns)} 列"

@tool
def analyze_data(analysis_type: str, column: str) -> str:
    """分析数据"""
    # 这里简化处理
    return f"对 {column} 列进行 {analysis_type} 分析的结果：..."

@tool
def generate_visualization(chart_type: str, x_column: str, y_column: str) -> str:
    """生成可视化"""
    return f"已生成 {chart_type} 图表：{x_column} vs {y_column}"

data_analyst = create_agent(
    model="gpt-4",
    tools=[load_dataset, analyze_data, generate_visualization],
    system_prompt="你是数据分析助手，帮助用户分析数据和生成可视化"
)

# 用户：分析销售数据
response = data_analyst.invoke({
    "messages": [{"role": "user", "content": "加载 sales.csv 并分析销售趋势"}]
})
# Agent 会按步骤：
# 1. load_dataset("sales.csv")
# 2. analyze_data("趋势分析", "销售额")
# 3. generate_visualization("折线图", "日期", "销售额")
```

### 1.5 LangChain 生态系统

```
┌─────────────────────────────────────────┐
│         LangChain 生态系统               │
├─────────────────────────────────────────┤
│                                         │
│  ┌────────────┐  ┌────────────┐        │
│  │ LangChain  │  │ LangGraph  │        │
│  │ 核心组件    │  │ 工作流编排  │        │
│  └────────────┘  └────────────┘        │
│                                         │
│  ┌────────────┐  ┌────────────┐        │
│  │ LangSmith  │  │ LangServe  │        │
│  │ 监控追踪    │  │ 部署服务    │        │
│  └────────────┘  └────────────┘        │
│                                         │
│  ┌────────────┐  ┌────────────┐        │
│  │ Community  │  │ Integrations│        │
│  │ 社区贡献    │  │ 第三方集成   │        │
│  └────────────┘  └────────────┘        │
└─────────────────────────────────────────┘
```

**核心库说明**：

| 库名 | 用途 | 何时使用 |
|-----|------|---------|
| **langchain** | 核心组件（Models, Tools, Prompts） | 构建基础 LLM 应用 |
| **langgraph** | 状态工作流编排 | 复杂多步骤任务、条件路由 |
| **langsmith** | 监控、追踪、评估 | 生产环境监控和调试 |
| **langserve** | 部署 REST API | 将 Chain/Agent 部署为服务 |
| **langchain-community** | 社区集成 | 使用第三方工具和服务 |

---

## 第二部分：安装与环境配置

### 2.1 基础安装

#### 最小化安装

```bash
# 只安装核心库
pip install langchain langchain-core

# 验证安装
python -c "import langchain; print(langchain.__version__)"
```

#### 完整安装（推荐）

```bash
# 核心库
pip install langchain langchain-core

# 文本分割器
pip install langchain-text-splitters

# 社区集成
pip install langchain-community

# 工作流编排
pip install langgraph

# 监控追踪
pip install langsmith

# 常用工具
pip install beautifulsoup4  # 网页加载
pip install faiss-cpu       # 向量存储
```

#### 使用 requirements.txt

创建 `requirements.txt` 文件：

```txt
# requirements.txt
langchain>=0.1.0
langchain-core>=0.1.0
langchain-community>=0.0.20
langchain-text-splitters>=0.0.1
langgraph>=0.0.40
langsmith>=0.0.70

# 模型提供商（根据需要选择）
langchain-openai>=0.0.5
langchain-anthropic>=0.0.5
langchain-google-genai>=0.0.5

# 工具库
beautifulsoup4>=4.12.0
faiss-cpu>=1.7.4
```

安装：
```bash
pip install -r requirements.txt
```

### 2.2 各模型提供商集成

#### OpenAI

```bash
# 安装
pip install langchain-openai
```

```python
import os
from langchain_openai import ChatOpenAI

# 设置 API Key
os.environ["OPENAI_API_KEY"] = "sk-..."

# 初始化模型
model = ChatOpenAI(
    model="gpt-4",
    temperature=0.7
)

# 测试调用
response = model.invoke("你好，世界！")
print(response.content)
```

#### Anthropic Claude

```bash
# 安装
pip install langchain-anthropic
```

```python
import os
from langchain_anthropic import ChatAnthropic

# 设置 API Key
os.environ["ANTHROPIC_API_KEY"] = "sk-ant-..."

# 初始化模型
model = ChatAnthropic(
    model="claude-sonnet-4-5-20250929",
    temperature=0.7,
    max_tokens=1024
)

# 测试调用
response = model.invoke("你好，Claude！")
print(response.content)
```

#### Google Gemini

```bash
# 安装
pip install langchain-google-genai
```

```python
import os
from langchain_google_genai import ChatGoogleGenerativeAI

# 设置 API Key
os.environ["GOOGLE_API_KEY"] = "..."

# 初始化模型
model = ChatGoogleGenerativeAI(
    model="gemini-2.5-flash-lite",
    temperature=0.7
)

# 测试调用
response = model.invoke("你好，Gemini！")
print(response.content)
```

#### Azure OpenAI

```bash
# 安装
pip install langchain-openai
```

```python
import os
from langchain_openai import AzureChatOpenAI

# 设置环境变量
os.environ["AZURE_OPENAI_API_KEY"] = "..."
os.environ["AZURE_OPENAI_ENDPOINT"] = "https://your-resource.openai.azure.com/"
os.environ["OPENAI_API_VERSION"] = "2024-02-15-preview"

# 初始化模型
model = AzureChatOpenAI(
    model="gpt-4",
    azure_deployment="your-deployment-name",
    temperature=0.7
)

# 测试调用
response = model.invoke("你好，Azure OpenAI！")
print(response.content)
```

### 2.3 环境变量配置最佳实践

#### 使用 .env 文件（推荐）

创建 `.env` 文件：

```bash
# .env 文件

# OpenAI
OPENAI_API_KEY=sk-...

# Anthropic
ANTHROPIC_API_KEY=sk-ant-...

# Google
GOOGLE_API_KEY=...

# LangSmith（可选）
LANGSMITH_API_KEY=...
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=my-project
```

使用 python-dotenv 加载：

```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv
import os

# 加载 .env 文件
load_dotenv()

# 现在可以使用环境变量
api_key = os.getenv("OPENAI_API_KEY")

from langchain_openai import ChatOpenAI
model = ChatOpenAI()  # 自动从环境变量读取 API Key
```

#### 使用 getpass（交互式输入）

```python
import getpass
import os

# 如果环境变量不存在，提示用户输入
if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass("请输入 OpenAI API Key: ")

from langchain_openai import ChatOpenAI
model = ChatOpenAI()
```

### 2.4 验证安装

创建测试脚本 `test_installation.py`：

```python
"""
LangChain 安装验证脚本
"""

def test_basic_import():
    """测试基础导入"""
    try:
        import langchain
        import langchain_core
        import langgraph
        print("✅ 基础库导入成功")
        print(f"   LangChain 版本: {langchain.__version__}")
        return True
    except ImportError as e:
        print(f"❌ 基础库导入失败: {e}")
        return False

def test_openai():
    """测试 OpenAI"""
    try:
        from langchain_openai import ChatOpenAI
        import os
        
        if not os.environ.get("OPENAI_API_KEY"):
            print("⚠️  OPENAI_API_KEY 未设置，跳过测试")
            return True
        
        model = ChatOpenAI(model="gpt-3.5-turbo")
        response = model.invoke("测试")
        print("✅ OpenAI 连接成功")
        return True
    except Exception as e:
        print(f"❌ OpenAI 连接失败: {e}")
        return False

def test_anthropic():
    """测试 Anthropic"""
    try:
        from langchain_anthropic import ChatAnthropic
        import os
        
        if not os.environ.get("ANTHROPIC_API_KEY"):
            print("⚠️  ANTHROPIC_API_KEY 未设置，跳过测试")
            return True
        
        model = ChatAnthropic(model="claude-3-haiku-20240307")
        response = model.invoke("测试")
        print("✅ Anthropic 连接成功")
        return True
    except Exception as e:
        print(f"❌ Anthropic 连接失败: {e}")
        return False

def test_agent_creation():
    """测试 Agent 创建"""
    try:
        from langchain.agents import create_agent
        from langchain.tools import tool
        
        @tool
        def dummy_tool(input: str) -> str:
            """测试工具"""
            return "测试结果"
        
        agent = create_agent(
            model="gpt-3.5-turbo",
            tools=[dummy_tool]
        )
        print("✅ Agent 创建成功")
        return True
    except Exception as e:
        print(f"❌ Agent 创建失败: {e}")
        return False

if __name__ == "__main__":
    print("=" * 50)
    print("开始验证 LangChain 安装...")
    print("=" * 50)
    
    tests = [
        test_basic_import,
        test_openai,
        test_anthropic,
        test_agent_creation
    ]
    
    results = [test() for test in tests]
    
    print("=" * 50)
    print(f"测试完成：{sum(results)}/{len(results)} 通过")
    print("=" * 50)
```

运行验证：
```bash
python test_installation.py
```

---

## 第三部分：核心架构深入理解

### 3.1 Runnable 接口详解

**Runnable** 是 LangChain 中所有组件的基础接口。理解 Runnable 是掌握 LangChain 的关键。

#### 什么是 Runnable？

```python
from langchain_core.runnables import Runnable

# 所有这些都是 Runnable：
# - Model (ChatOpenAI, ChatAnthropic, ...)
# - PromptTemplate
# - OutputParser
# - Tool
# - Chain
# - Agent
```

#### Runnable 的核心方法

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-3.5-turbo")

# 1. invoke() - 单次同步调用
response = model.invoke("你好")
print(response.content)

# 2. batch() - 批量处理
responses = model.batch([
    "第一个问题",
    "第二个问题",
    "第三个问题"
])
for resp in responses:
    print(resp.content)

# 3. stream() - 流式输出
for chunk in model.stream("给我讲个长故事"):
    print(chunk.content, end="", flush=True)

# 4. ainvoke() - 异步调用
import asyncio

async def async_call():
    response = await model.ainvoke("你好")
    print(response.content)

asyncio.run(async_call())

# 5. abatch() - 异步批量
async def async_batch():
    responses = await model.abatch(["问题1", "问题2"])
    for resp in responses:
        print(resp.content)

asyncio.run(async_batch())

# 6. astream() - 异步流式
async def async_stream():
    async for chunk in model.astream("讲个故事"):
        print(chunk.content, end="", flush=True)

asyncio.run(async_stream())
```

#### Runnable 方法对比

| 方法 | 同步/异步 | 返回类型 | 使用场景 |
|------|----------|---------|---------|
| `invoke()` | 同步 | 单个结果 | 单个请求，等待完成 |
| `batch()` | 同步 | 结果列表 | 多个请求，批量处理 |
| `stream()` | 同步 | 生成器 | 长文本，实时显示 |
| `ainvoke()` | 异步 | 单个结果 | 异步环境，单个请求 |
| `abatch()` | 异步 | 结果列表 | 异步环境，批量请求 |
| `astream()` | 异步 | 异步生成器 | 异步环境，流式输出 |

#### 实战示例：批量翻译

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

# 创建翻译提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的翻译助手"),
    ("user", "将以下文本翻译成英文：{text}")
])

model = ChatOpenAI(model="gpt-3.5-turbo")

# 创建翻译链
translation_chain = prompt | model

# 批量翻译
texts_to_translate = [
    {"text": "你好，世界"},
    {"text": "今天天气真好"},
    {"text": "我喜欢编程"}
]

# 使用 batch 方法
translations = translation_chain.batch(texts_to_translate)

for original, translation in zip(texts_to_translate, translations):
    print(f"原文: {original['text']}")
    print(f"译文: {translation.content}")
    print("-" * 50)
```

### 3.2 LCEL 表达式语言

**LCEL (LangChain Expression Language)** 是一种声明式的方式来组合 LangChain 组件。

#### 基础语法

```python
# LCEL 使用 | 操作符连接组件
chain = component1 | component2 | component3

# 等价于函数组合
result = component3(component2(component1(input)))
```

#### 示例 1：简单的提示 → 模型 → 解析链

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 组件 1: 提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手"),
    ("user", "{question}")
])

# 组件 2: 模型
model = ChatOpenAI(model="gpt-3.5-turbo")

# 组件 3: 输出解析器
output_parser = StrOutputParser()

# 使用 LCEL 组合
chain = prompt | model | output_parser

# 调用链
result = chain.invoke({"question": "什么是量子计算？"})
print(result)  # 直接得到字符串结果
```

#### 示例 2：多步骤处理链

```python
from langchain_core.runnables import RunnableLambda

# 自定义处理函数
def to_uppercase(text: str) -> str:
    """转大写"""
    return text.upper()

def add_prefix(text: str) -> str:
    """添加前缀"""
    return f"[处理结果] {text}"

# 创建 Runnable
uppercase_runnable = RunnableLambda(to_uppercase)
prefix_runnable = RunnableLambda(add_prefix)

# 组合链
processing_chain = prompt | model | output_parser | uppercase_runnable | prefix_runnable

result = processing_chain.invoke({"question": "hello"})
print(result)
# 输出: [处理结果] HELLO, I'M AN AI ASSISTANT...
```

#### 示例 3：条件路由

```python
from langchain_core.runnables import RunnableBranch

# 定义不同的处理路径
def is_question(input_dict):
    """判断是否是问题"""
    return "?" in input_dict.get("text", "")

# 路由 1: 问题处理链
question_chain = (
    ChatPromptTemplate.from_template("回答问题：{text}")
    | model
    | StrOutputParser()
)

# 路由 2: 陈述处理链
statement_chain = (
    ChatPromptTemplate.from_template("总结陈述：{text}")
    | model
    | StrOutputParser()
)

# 创建分支路由
branch = RunnableBranch(
    (is_question, question_chain),  # 如果是问题，使用问题链
    statement_chain  # 否则使用陈述链
)

# 测试
print(branch.invoke({"text": "什么是AI？"}))  # 使用问题链
print(branch.invoke({"text": "今天天气很好。"}))  # 使用陈述链
```

### 3.3 链式调用机制

#### 理解数据流

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("讲一个关于{topic}的笑话")
model = ChatOpenAI()
parser = StrOutputParser()

chain = prompt | model | parser

# 数据流向：
# 1. {"topic": "程序员"} 
#     ↓
# 2. prompt.invoke() → "讲一个关于程序员的笑话"
#     ↓
# 3. model.invoke() → AIMessage(content="...")
#     ↓
# 4. parser.invoke() → "笑话内容字符串"
```

#### 调试链式调用

```python
from langchain_core.runnables import RunnablePassthrough

def debug_print(x):
    """调试打印"""
    print(f"🔍 中间结果: {x}")
    return x

# 在链中插入调试节点
debug_runnable = RunnableLambda(debug_print)

chain_with_debug = (
    prompt 
    | debug_runnable  # 调试点 1
    | model 
    | debug_runnable  # 调试点 2
    | parser
)

result = chain_with_debug.invoke({"topic": "猫"})
```

### 3.4 组件化设计思想

#### 单一职责原则

每个组件只做一件事：

```python
# ✅ 好的设计 - 每个组件职责单一
from langchain.tools import tool

@tool
def get_weather(city: str) -> str:
    """只负责获取天气"""
    return f"{city}的天气"

@tool
def translate_text(text: str, target_lang: str) -> str:
    """只负责翻译"""
    return f"翻译结果"

# ❌ 不好的设计 - 一个工具做太多事
@tool
def do_everything(city: str, translate: bool, lang: str) -> str:
    """又查天气又翻译，职责不清"""
    weather = f"{city}的天气"
    if translate:
        return f"翻译: {weather}"
    return weather
```

#### 可组合性

```python
# 小组件可以组合成大组件

# 基础组件
weather_tool = get_weather
translation_tool = translate_text

# 组合成 Agent
weather_agent = create_agent(
    model="gpt-4",
    tools=[weather_tool]
)

multilingual_weather_agent = create_agent(
    model="gpt-4",
    tools=[weather_tool, translation_tool]
)
```

#### 可复用性

```python
# 定义一次，到处使用

# 定义可复用的提示模板
HELPFUL_ASSISTANT_PROMPT = ChatPromptTemplate.from_messages([
    ("system", "你是一个有帮助的助手"),
    ("user", "{input}")
])

# 在多个链中复用
chain1 = HELPFUL_ASSISTANT_PROMPT | model1 | parser
chain2 = HELPFUL_ASSISTANT_PROMPT | model2 | parser
chain3 = HELPFUL_ASSISTANT_PROMPT | model3 | parser
```

---

## 第四部分：模型集成完全指南

### 4.1 初始化模型的三种方式

#### 方式 1：使用字符串标识符（最简单）

```python
from langchain.agents import create_agent

# LangChain 会自动推断提供商
agent = create_agent(
    "gpt-4",  # 自动识别为 OpenAI
    tools=[]
)

# 对于其他提供商，使用前缀
agent2 = create_agent(
    "google_genai:gemini-2.5-flash-lite",  # Google Gemini
    tools=[]
)
```

**优点**：
- 代码简洁
- 快速原型开发
- 易于切换模型

**缺点**：
- 无法精细控制参数
- 需要遵循命名规范

#### 方式 2：使用 init_chat_model（推荐）

```python
from langchain.chat_models import init_chat_model

# 基础初始化
model = init_chat_model("gpt-4")

# 带参数初始化
model = init_chat_model(
    "claude-sonnet-4-5-20250929",
    temperature=0.7,      # 创造性
    timeout=30,           # 超时时间
    max_tokens=1000,      # 最大 token 数
    max_retries=3         # 重试次数
)

# 指定提供商
model = init_chat_model(
    model="gpt-4",
    model_provider="openai"
)
```

**优点**：
- 统一的接口
- 支持参数配置
- 便于切换提供商

#### 方式 3：直接实例化（最灵活）

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model="gpt-4",
    temperature=0.7,
    max_tokens=2000,
    timeout=60,
    max_retries=3,
    api_key="sk-...",  # 可以直接传入
    organization="org-...",
    base_url="https://api.openai.com/v1"
)
```

**优点**：
- 完全控制所有参数
- 支持高级配置
- 类型提示完整

### 4.2 模型参数详解

#### temperature（温度）

控制输出的随机性和创造性：

```python
from langchain_openai import ChatOpenAI

# 低温度 - 更确定性，更一致
formal_model = ChatOpenAI(
    model="gpt-4",
    temperature=0.1  # 适合：代码生成、数据提取、精确任务
)

response = formal_model.invoke("2 + 2 = ?")
# 输出: "4"（每次都相同）

# 中温度 - 平衡
balanced_model = ChatOpenAI(
    model="gpt-4",
    temperature=0.7  # 适合：对话、问答、一般任务
)

# 高温度 - 更创造性，更多样
creative_model = ChatOpenAI(
    model="gpt-4",
    temperature=1.5  # 适合：创意写作、头脑风暴
)

response = creative_model.invoke("写一个科幻故事开头")
# 每次输出都不同，更有创意
```

| Temperature | 特点 | 适用场景 |
|------------|------|---------|
| 0.0 - 0.3 | 高确定性、低创造性 | 代码生成、数据提取、分类 |
| 0.4 - 0.7 | 平衡性 | 对话、问答、翻译 |
| 0.8 - 1.5 | 高创造性、低确定性 | 创意写作、头脑风暴 |

#### max_tokens（最大 token 数）

```python
# 限制输出长度
short_model = ChatOpenAI(
    model="gpt-4",
    max_tokens=100  # 适合：简短回答、摘要
)

long_model = ChatOpenAI(
    model="gpt-4",
    max_tokens=4000  # 适合：长文本生成、详细解释
)

# 示例
response = short_model.invoke("解释量子计算")
print(len(response.content))  # 大约 100 个 token

response = long_model.invoke("详细解释量子计算的历史和应用")
print(len(response.content))  # 可以达到 4000 个 token
```

#### timeout（超时时间）

```python
# 设置超时避免无限等待
model = ChatOpenAI(
    model="gpt-4",
    timeout=30  # 30 秒超时
)

try:
    response = model.invoke("非常复杂的问题...")
except Exception as e:
    print(f"请求超时: {e}")
```

#### max_retries（最大重试次数）

```python
# 自动重试失败的请求
robust_model = ChatOpenAI(
    model="gpt-4",
    max_retries=5  # 失败后重试 5 次
)

# 适合生产环境，增加可靠性
```

### 4.3 实战：构建可配置的模型管理器

```python
from typing import Literal
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic

class ModelManager:
    """模型管理器 - 统一管理不同的模型配置"""
    
    def __init__(self):
        self.models = {}
    
    def register_model(
        self,
        name: str,
        provider: Literal["openai", "anthropic"],
        model_name: str,
        **kwargs
    ):
        """注册模型配置"""
        if provider == "openai":
            self.models[name] = ChatOpenAI(
                model=model_name,
                **kwargs
            )
        elif provider == "anthropic":
            self.models[name] = ChatAnthropic(
                model=model_name,
                **kwargs
            )
    
    def get_model(self, name: str):
        """获取模型"""
        return self.models.get(name)
    
    def list_models(self):
        """列出所有模型"""
        return list(self.models.keys())

# 使用示例
manager = ModelManager()

# 注册不同用途的模型
manager.register_model(
    name="fast",
    provider="openai",
    model_name="gpt-3.5-turbo",
    temperature=0.5,
    max_tokens=500
)

manager.register_model(
    name="smart",
    provider="openai",
    model_name="gpt-4",
    temperature=0.7,
    max_tokens=2000
)

manager.register_model(
    name="creative",
    provider="anthropic",
    model_name="claude-sonnet-4-5-20250929",
    temperature=1.0,
    max_tokens=3000
)

# 根据任务选择模型
fast_model = manager.get_model("fast")
response = fast_model.invoke("快速问答")

smart_model = manager.get_model("smart")
response = smart_model.invoke("复杂推理")

creative_model = manager.get_model("creative")
response = creative_model.invoke("创意写作")
```

### 4.4 模型调用高级技巧

#### 技巧 1：使用消息类型

```python
from langchain_core.messages import (
    HumanMessage,
    AIMessage,
    SystemMessage
)
from langchain_openai import ChatOpenAI

model = ChatOpenAI()

# 结构化的对话历史
messages = [
    SystemMessage(content="你是一个 Python 编程专家"),
    HumanMessage(content="什么是装饰器？"),
    AIMessage(content="装饰器是修改函数行为的函数..."),
    HumanMessage(content="给我一个示例")
]

response = model.invoke(messages)
print(response.content)
```

#### 技巧 2：流式输出优化

```python
import sys

def stream_response(query: str):
    """流式输出并显示进度"""
    model = ChatOpenAI(model="gpt-4")
    
    print("AI: ", end="", flush=True)
    full_response = ""
    
    for chunk in model.stream(query):
        content = chunk.content
        full_response += content
        print(content, end="", flush=True)
        sys.stdout.flush()
    
    print()  # 换行
    return full_response

# 使用
response = stream_response("解释机器学习")
```

#### 技巧 3：错误处理

```python
from langchain_openai import ChatOpenAI
from langchain_core.exceptions import LangChainException
import time

def invoke_with_retry(model, query, max_retries=3):
    """带重试的模型调用"""
    for attempt in range(max_retries):
        try:
            return model.invoke(query)
        except LangChainException as e:
            if attempt == max_retries - 1:
                raise
            print(f"尝试 {attempt + 1} 失败，重试中...")
            time.sleep(2 ** attempt)  # 指数退避

# 使用
model = ChatOpenAI(model="gpt-4")
response = invoke_with_retry(model, "你好")
```

---

## 第五部分：创建第一个 Agent

### 5.1 Agent 基本概念

#### 什么是 Agent？

Agent 是一个能够：
1. **理解任务**：解析用户意图
2. **选择工具**：决定使用哪些工具
3. **执行操作**：调用工具获取结果
4. **综合回答**：基于结果生成最终答案

```
用户问题
    ↓
【Agent 推理】
    ↓
选择工具 A → 执行 → 获取结果 A
    ↓
【Agent 推理】
    ↓
选择工具 B → 执行 → 获取结果 B
    ↓
【Agent 推理】
    ↓
综合 A 和 B → 生成最终答案
```

### 5.2 创建最简单的 Agent

```python
from langchain.agents import create_agent
from langchain.tools import tool

# 步骤 1: 定义工具
@tool
def calculator(expression: str) -> str:
    """计算数学表达式。
    
    Args:
        expression: 数学表达式，如 "2 + 3 * 4"
    """
    try:
        result = eval(expression)
        return f"结果是: {result}"
    except Exception as e:
        return f"计算错误: {e}"

# 步骤 2: 创建 Agent
agent = create_agent(
    model="gpt-4",
    tools=[calculator],
    system_prompt="你是一个数学助手，帮助用户进行计算"
)

# 步骤 3: 使用 Agent
response = agent.invoke({
    "messages": [{"role": "user", "content": "计算 123 * 456"}]
})

print(response["messages"][-1].content)
```

### 5.3 多工具 Agent

```python
from langchain.agents import create_agent
from langchain.tools import tool
from datetime import datetime

# 工具 1: 获取当前时间
@tool
def get_current_time() -> str:
    """获取当前时间"""
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# 工具 2: 天气查询
@tool
def get_weather(city: str) -> str:
    """获取城市天气
    
    Args:
        city: 城市名称
    """
    # 模拟天气数据
    weather_data = {
        "北京": "晴天，15°C",
        "上海": "多云，18°C",
        "深圳": "阴天，22°C"
    }
    return weather_data.get(city, f"未找到 {city} 的天气数据")

# 工具 3: 货币转换
@tool
def convert_currency(amount: float, from_currency: str, to_currency: str) -> str:
    """货币转换
    
    Args:
        amount: 金额
        from_currency: 源货币
        to_currency: 目标货币
    """
    # 模拟汇率
    rates = {
        ("USD", "CNY"): 7.2,
        ("CNY", "USD"): 0.14,
        ("USD", "EUR"): 0.92,
    }
    rate = rates.get((from_currency, to_currency), 1.0)
    result = amount * rate
    return f"{amount} {from_currency} = {result:.2f} {to_currency}"

# 创建多功能 Agent
assistant = create_agent(
    model="gpt-4",
    tools=[get_current_time, get_weather, convert_currency],
    system_prompt="""你是一个多功能助手，可以：
    1. 查询当前时间
    2. 查询天气
    3. 进行货币转换
    
    根据用户问题，选择合适的工具来回答。
    """
)

# 测试 Agent
test_queries = [
    "现在几点了？",
    "北京的天气怎么样？",
    "100 美元是多少人民币？"
]

for query in test_queries:
    print(f"\n用户: {query}")
    response = assistant.invoke({
        "messages": [{"role": "user", "content": query}]
    })
    print(f"AI: {response['messages'][-1].content}")
```

### 5.4 Agent 执行流程详解

让我们深入理解 Agent 的执行过程：

```python
from langchain.agents import create_agent
from langchain.tools import tool

@tool
def search(query: str) -> str:
    """搜索信息"""
    return f"搜索结果：关于 {query} 的信息..."

@tool
def calculate(expression: str) -> str:
    """计算数学表达式"""
    return f"计算结果：{eval(expression)}"

# 创建 Agent
agent = create_agent(
    model="gpt-4",
    tools=[search, calculate]
)

# 执行并观察流程
response = agent.invoke({
    "messages": [{"role": "user", "content": "搜索量子计算，然后计算 2^10"}]
})

# Agent 的执行过程：
# 
# 【第1轮】
# Agent 思考：用户要我搜索量子计算
# ↓
# 调用工具：search("量子计算")
# ↓
# 工具返回："搜索结果：关于量子计算的信息..."
# 
# 【第2轮】
# Agent 思考：现在需要计算 2^10
# ↓
# 调用工具：calculate("2**10")
# ↓
# 工具返回："计算结果：1024"
# 
# 【第3轮】
# Agent 思考：已经完成两个任务，可以回答用户了
# ↓
# 生成回答："我已经搜索了量子计算的信息，并计算出 2^10 = 1024"

print(response["messages"][-1].content)
```

### 5.5 实战项目：个人助理 Agent

创建一个完整的个人助理 Agent：

```python
from langchain.agents import create_agent
from langchain.tools import tool
from langgraph.checkpoint.memory import InMemorySaver
from datetime import datetime
import random

# === 工具定义 ===

@tool
def get_time() -> str:
    """获取当前时间"""
    return datetime.now().strftime("%Y年%m月%d日 %H:%M:%S")

@tool
def set_reminder(task: str, time: str) -> str:
    """设置提醒
    
    Args:
        task: 提醒事项
        time: 提醒时间
    """
    return f"✅ 已设置提醒：{task}，时间：{time}"

@tool
def get_weather(city: str) -> str:
    """获取天气"""
    weathers = ["晴天", "多云", "小雨", "阴天"]
    temp = random.randint(15, 30)
    return f"{city}的天气：{random.choice(weathers)}，温度{temp}°C"

@tool
def search_info(query: str) -> str:
    """搜索信息"""
    return f"关于'{query}'的搜索结果：这是一个重要的话题..."

@tool
def calculate(expression: str) -> str:
    """计算器"""
    try:
        result = eval(expression)
        return f"计算结果：{result}"
    except:
        return "计算出错，请检查表达式"

@tool
def translate(text: str, target_lang: str) -> str:
    """翻译文本
    
    Args:
        text: 要翻译的文本
        target_lang: 目标语言（如：英文、日文）
    """
    return f"[翻译结果] {text} → {target_lang}"

# === 创建个人助理 ===

personal_assistant = create_agent(
    model="gpt-4",
    tools=[
        get_time,
        set_reminder,
        get_weather,
        search_info,
        calculate,
        translate
    ],
    system_prompt="""你是一个专业的个人助理，名字叫小智。

你的能力：
1. 📅 时间管理：查询时间、设置提醒
2. 🌤️  天气查询：查询各城市天气
3. 🔍 信息搜索：搜索各类信息
4. 🧮 数学计算：进行数学运算
5. 🌐 文本翻译：翻译多种语言

工作原则：
- 友好、专业、高效
- 主动理解用户需求
- 提供准确的信息
- 必要时使用多个工具完成任务
    """,
    checkpointer=InMemorySaver()  # 记忆系统
)

# === 交互式测试 ===

def chat_with_assistant():
    """与助理交互"""
    print("=" * 60)
    print("个人助理小智已就绪！输入 'quit' 退出")
    print("=" * 60)
    
    conversation_id = "user_001"
    
    while True:
        user_input = input("\n你: ").strip()
        
        if user_input.lower() in ['quit', 'exit', '退出']:
            print("小智: 再见！祝你有美好的一天！")
            break
        
        if not user_input:
            continue
        
        # 调用 Agent
        response = personal_assistant.invoke(
            {"messages": [{"role": "user", "content": user_input}]},
            {"configurable": {"thread_id": conversation_id}}
        )
        
        # 显示回答
        ai_response = response["messages"][-1].content
        print(f"\n小智: {ai_response}")

# 运行（注释掉以防止阻塞）
# chat_with_assistant()

# 测试示例
test_conversations = [
    "现在几点了？",
    "明天上午9点提醒我开会",
    "北京的天气怎么样？",
    "搜索量子计算",
    "计算 (123 + 456) * 2",
    "把 'Hello World' 翻译成中文"
]

print("\n" + "=" * 60)
print("测试对话")
print("=" * 60)

for query in test_conversations:
    print(f"\n用户: {query}")
    response = personal_assistant.invoke(
        {"messages": [{"role": "user", "content": query}]},
        {"configurable": {"thread_id": "test_session"}}
    )
    print(f"小智: {response['messages'][-1].content}")
```

### 5.6 第一阶段总结与练习

#### 你已经学会了：

✅ **LangChain 概述**
- LangChain 的核心价值
- 解决的问题和应用场景
- 生态系统组成

✅ **安装与配置**
- 安装各种组件
- 配置模型提供商
- 环境变量管理

✅ **核心架构**
- Runnable 接口
- LCEL 表达式语言
- 组件化设计

✅ **模型集成**
- 三种初始化方式
- 参数配置和优化
- 高级调用技巧

✅ **Agent 创建**
- 工具定义
- Agent 构建
- 执行流程理解

#### 练习项目

**练习 1：构建天气助手**
```python
# 要求：
# 1. 创建获取天气的工具
# 2. 创建获取时间的工具
# 3. 构建一个 Agent 可以回答：
#    - "今天天气怎么样？"
#    - "现在几点了？"
#    - "明天的天气适合户外运动吗？"
```

**练习 2：创建学习助手**
```python
# 要求：
# 1. 创建搜索工具（模拟搜索功能）
# 2. 创建总结工具（提取关键信息）
# 3. 创建翻译工具
# 4. Agent 能够：
#    - 搜索学习资料
#    - 总结重点
#    - 翻译内容
```

**练习 3：构建任务管理 Agent**
```python
# 要求：
# 1. 创建添加任务工具
# 2. 创建查看任务工具
# 3. 创建完成任务工具
# 4. Agent 能够管理待办事项
```

---

## 🎓 第一阶段完成！

恭喜你完成了 LangChain 第一阶段的学习！你现在已经掌握了：

- LangChain 的基础概念和核心价值
- 如何安装和配置 LangChain
- Runnable 接口和 LCEL 的使用
- 模型初始化和参数调优
- 创建和使用 Agent

### 📌 下一阶段预告

**第二阶段：LangGraph 与工作流编排**

你将学习：
1. 🔄 LangGraph 深入理解
2. 📊 StateGraph 状态管理
3. 🔀 条件路由和并行执行
4. 🏗️ 复杂工作流构建
5. 💾 Checkpointer 和持久化

准备好了吗？让我们进入下一阶段的学习！

