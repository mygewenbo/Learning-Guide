# MCP 学习计划 - 阶段 1：MCP 整体认知与核心概念详解

> **本文档目标**：深入理解 MCP 的整体架构、核心角色和能力类型，能够用自己的话清晰解释 MCP 的工作原理。

---

## 📚 目录

1. [MCP 是什么？深度解析](#1-mcp-是什么深度解析)
2. [MCP 的核心角色](#2-mcp-的核心角色)
3. [MCP 的三大核心能力](#3-mcp-的三大核心能力)
4. [MCP 的架构设计](#4-mcp-的架构设计)
5. [连接与传输方式](#5-连接与传输方式)
6. [实战练习：画出架构图](#6-实战练习画出架构图)
7. [总结与检查清单](#7-总结与检查清单)

---

## 1. MCP 是什么？深度解析

### 1.1 从问题出发

在学习 MCP 之前，让我们先理解它要解决的问题。

#### 场景 1：没有 MCP 的世界

假设你是一个 AI 应用开发者，想让你的 AI 助手能够：
- 查询天气信息
- 搜索网页内容
- 读取本地文件
- 查询数据库

**你需要做什么？**

```python
# 为每个功能写独立的集成代码

# 天气 API
def get_weather(city):
    response = requests.get(f"https://weather-api.com/v1/{city}")
    return parse_weather_response(response)

# 搜索 API
def web_search(query):
    response = requests.get(f"https://search-api.com/search?q={query}")
    return parse_search_response(response)

# 文件系统
def read_file(path):
    with open(path, 'r') as f:
        return f.read()

# 数据库
def query_database(sql):
    conn = psycopg2.connect(...)
    cursor = conn.cursor()
    cursor.execute(sql)
    return cursor.fetchall()

# 然后你还需要：
# 1. 为每个功能写 LLM 的工具描述
# 2. 处理参数验证
# 3. 处理错误
# 4. 管理认证信息
# 5. ...
```

**问题**：
- 每个功能都需要单独集成
- 代码重复，维护困难
- 如果换一个 AI 应用，需要重新写一遍
- 没有统一的标准

#### 场景 2：有了 MCP 的世界

```python
# 使用 MCP，你只需要：

from mcp import ClientSession

# 连接到 MCP Server
async with stdio_client(server_config) as (read, write):
    async with ClientSession(read, write) as session:
        # 初始化
        await session.initialize()
        
        # 自动发现所有可用工具
        tools = await session.list_tools()
        
        # 调用任何工具，统一的接口
        result = await session.call_tool("get_weather", {"city": "北京"})
        result = await session.call_tool("web_search", {"query": "MCP"})
        result = await session.call_tool("read_file", {"path": "/data/file.txt"})
```

**优势**：
- ✅ 统一的接口
- ✅ 自动发现能力
- ✅ 标准化的错误处理
- ✅ 一次集成，到处使用

### 1.2 MCP 的本质

**MCP 是一个协议标准**，就像 HTTP 是网页通信的标准一样。

#### 类比理解

| 概念 | 现实世界 | MCP 世界 |
|------|---------|---------|
| **标准接口** | USB-C 接口 | MCP 协议 |
| **设备** | 手机、电脑、充电器 | AI 应用、工具服务 |
| **好处** | 一个接口，连接所有设备 | 一个协议，连接所有工具 |

#### MCP 的定义（官方）

> **Model Context Protocol (MCP)** 是一个开放协议，用于标准化 AI 应用如何与外部数据源、工具和服务进行交互。

**关键词解析**：
- **开放协议**：任何人都可以实现，不被单一公司控制
- **标准化**：统一的规范，所有人遵循同样的规则
- **AI 应用**：大语言模型、AI 助手、Agent 等
- **外部数据源**：数据库、文件系统、API 等
- **工具和服务**：各种可以被调用的功能

### 1.3 MCP 的价值主张

#### 对于开发者

```
传统方式：
AI 应用 1 ──自定义集成──> 工具 A
AI 应用 1 ──自定义集成──> 工具 B
AI 应用 2 ──自定义集成──> 工具 A  ← 重复工作
AI 应用 2 ──自定义集成──> 工具 B  ← 重复工作

MCP 方式：
AI 应用 1 ──MCP──> 工具 A (MCP Server)
AI 应用 1 ──MCP──> 工具 B (MCP Server)
AI 应用 2 ──MCP──> 工具 A (MCP Server)  ← 无需重新开发
AI 应用 2 ──MCP──> 工具 B (MCP Server)  ← 无需重新开发
```

**节省的工作量**：
- 不需要为每个 AI 应用重新集成
- 不需要学习每个工具的独特 API
- 不需要维护多套集成代码

#### 对于生态系统

```
MCP 生态：

┌─────────────────────────────────────┐
│         MCP Host (AI 应用)          │
│  Claude Desktop / VS Code / 自建    │
└──────────────┬──────────────────────┘
               │ MCP 协议
    ┌──────────┼──────────┬──────────┐
    │          │          │          │
┌───▼───┐  ┌──▼───┐  ┌───▼───┐  ┌──▼───┐
│文件系统│  │数据库 │  │GitHub │  │Slack │
│Server │  │Server │  │Server │  │Server│
└───────┘  └──────┘  └───────┘  └──────┘

任何人都可以：
1. 开发新的 MCP Server
2. 立即被所有 MCP Host 使用
3. 分享给社区
```

### 1.4 MCP 与其他技术的对比

#### MCP vs 传统 API

| 特性 | 传统 API | MCP |
|------|---------|-----|
| **标准化** | 每个 API 不同 | 统一标准 |
| **发现机制** | 需要查文档 | 自动发现 |
| **工具描述** | 手动编写 | 自动提供 |
| **错误处理** | 各不相同 | 标准化 |
| **适用场景** | 通用 | 专为 AI/LLM 设计 |

#### MCP vs LangChain Tools

| 特性 | LangChain Tools | MCP |
|------|----------------|-----|
| **绑定** | 绑定到 LangChain | 独立协议 |
| **跨应用** | 需要适配 | 原生支持 |
| **标准化** | 框架内标准 | 行业标准 |
| **传输层** | 内存调用 | 多种传输方式 |

---

## 2. MCP 的核心角色

MCP 架构中有三个核心角色，理解它们的职责和关系是掌握 MCP 的关键。

### 2.1 Host（宿主）

#### 什么是 Host？

**Host** 是用户直接交互的应用程序，它负责管理整个 MCP 生态。

#### Host 的典型例子

1. **Claude Desktop**
   - Anthropic 开发的桌面应用
   - 内置 MCP 支持
   - 用户通过配置文件添加 MCP Servers

2. **VS Code / Windsurf**
   - 代码编辑器
   - 通过插件支持 MCP
   - 为开发者提供 AI 辅助

3. **自建 AI 应用**
   - 你自己开发的聊天应用
   - 企业内部的 AI 助手
   - 定制化的 Agent 系统

#### Host 的职责

```
┌─────────────────────────────────────┐
│              Host                   │
│                                     │
│  职责：                             │
│  1. 管理用户界面                    │
│  2. 接收用户输入                    │
│  3. 管理多个 MCP Client             │
│  4. 协调 LLM 和工具调用             │
│  5. 展示结果给用户                  │
└─────────────────────────────────────┘
```

**详细职责**：

1. **连接管理**
   - 启动和管理多个 MCP Client
   - 维护与 MCP Servers 的连接
   - 处理连接断开和重连

2. **能力聚合**
   - 从所有连接的 Servers 收集工具
   - 合并所有可用的 Resources
   - 整理所有 Prompts

3. **决策制定**
   - 决定何时调用哪个工具
   - 管理工具调用的顺序
   - 处理工具调用的结果

4. **用户交互**
   - 展示 UI 界面
   - 接收用户命令
   - 显示执行结果

#### Host 的工作流程

```python
# Host 的典型工作流程（伪代码）

class MCPHost:
    def __init__(self):
        self.clients = []  # 管理多个 Client
        self.llm = LLM()   # 大语言模型
        
    async def start(self):
        # 1. 启动所有配置的 MCP Clients
        for server_config in self.config.servers:
            client = await self.connect_to_server(server_config)
            self.clients.append(client)
        
        # 2. 收集所有可用工具
        all_tools = []
        for client in self.clients:
            tools = await client.list_tools()
            all_tools.extend(tools)
        
        # 3. 注册工具到 LLM
        self.llm.register_tools(all_tools)
    
    async def handle_user_message(self, message):
        # 4. 用户发送消息
        response = await self.llm.chat(message)
        
        # 5. 如果 LLM 决定调用工具
        if response.tool_calls:
            for tool_call in response.tool_calls:
                # 找到对应的 Client
                client = self.find_client_for_tool(tool_call.name)
                
                # 调用工具
                result = await client.call_tool(
                    tool_call.name, 
                    tool_call.arguments
                )
                
                # 将结果返回给 LLM
                final_response = await self.llm.continue_with_result(result)
        
        # 6. 展示给用户
        return final_response
```

### 2.2 Client（客户端）

#### 什么是 Client？

**Client** 是 Host 内部的组件，每个 Client 负责与一个 MCP Server 通信。

#### Client 与 Host 的关系

```
┌─────────────────────────────────────────┐
│              Host                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │Client 1 │  │Client 2 │  │Client 3 │ │
│  └────┬────┘  └────┬────┘  └────┬────┘ │
└───────┼────────────┼────────────┼───────┘
        │            │            │
        │            │            │
   ┌────▼────┐  ┌───▼────┐  ┌───▼────┐
   │Server 1 │  │Server 2│  │Server 3│
   └─────────┘  └────────┘  └────────┘
```

**关键理解**：
- 一个 Host 可以有多个 Client
- 每个 Client 连接一个 Server
- Client 是 Host 和 Server 之间的桥梁

#### Client 的职责

```
┌─────────────────────────────────────┐
│            MCP Client               │
│                                     │
│  职责：                             │
│  1. 建立与 Server 的连接            │
│  2. 执行协议初始化                  │
│  3. 发现 Server 的能力              │
│  4. 发送请求到 Server               │
│  5. 接收并处理响应                  │
└─────────────────────────────────────┘
```

#### Client 的生命周期

```python
# Client 的典型生命周期

class MCPClient:
    async def connect(self, server_config):
        # 1. 建立传输连接（stdio/SSE/WebSocket）
        self.transport = await self.create_transport(server_config)
        
        # 2. 初始化会话
        init_response = await self.send_request({
            "method": "initialize",
            "params": {
                "protocolVersion": "2025-06-18",
                "capabilities": {...},
                "clientInfo": {
                    "name": "my-client",
                    "version": "1.0.0"
                }
            }
        })
        
        # 3. 保存 Server 的能力
        self.server_capabilities = init_response.capabilities
        
        # 4. 发送 initialized 通知
        await self.send_notification("notifications/initialized")
    
    async def list_tools(self):
        # 5. 发现工具
        response = await self.send_request({
            "method": "tools/list"
        })
        return response.tools
    
    async def call_tool(self, name, arguments):
        # 6. 调用工具
        response = await self.send_request({
            "method": "tools/call",
            "params": {
                "name": name,
                "arguments": arguments
            }
        })
        return response.content
    
    async def close(self):
        # 7. 关闭连接
        await self.transport.close()
```

### 2.3 Server（服务器）

#### 什么是 Server？

**Server** 是提供实际功能的服务进程，它暴露工具、资源和提示给 Client。

#### Server 的典型例子

1. **文件系统 Server**
   ```
   提供的工具：
   - read_file(path)
   - write_file(path, content)
   - list_directory(path)
   ```

2. **数据库 Server**
   ```
   提供的工具：
   - execute_query(sql)
   - get_schema()
   - list_tables()
   ```

3. **GitHub Server**
   ```
   提供的工具：
   - create_issue(repo, title, body)
   - list_pull_requests(repo)
   - search_code(query)
   ```

#### Server 的职责

```
┌─────────────────────────────────────┐
│           MCP Server                │
│                                     │
│  职责：                             │
│  1. 实现具体的业务逻辑              │
│  2. 暴露 Tools（工具）              │
│  3. 暴露 Resources（资源）          │
│  4. 暴露 Prompts（提示模板）        │
│  5. 处理来自 Client 的请求          │
│  6. 返回标准化的响应                │
└─────────────────────────────────────┘
```

#### Server 的实现示例

```python
# 一个简单的天气 Server 示例

from mcp.server import Server
from mcp.types import Tool, TextContent
import httpx

class WeatherServer:
    def __init__(self):
        self.server = Server("weather-server")
        self.setup_tools()
    
    def setup_tools(self):
        # 注册工具
        @self.server.tool()
        async def get_weather(city: str) -> str:
            """获取指定城市的天气信息"""
            # 调用天气 API
            async with httpx.AsyncClient() as client:
                response = await client.get(
                    f"https://api.weather.com/v1/{city}"
                )
                data = response.json()
            
            # 返回结果
            return f"{city}的天气：{data['weather']}，温度：{data['temp']}°C"
    
    async def run(self):
        # 启动 Server
        from mcp.server.stdio import stdio_server
        async with stdio_server() as (read, write):
            await self.server.run(read, write)

# 使用
if __name__ == "__main__":
    server = WeatherServer()
    asyncio.run(server.run())
```

### 2.4 三者关系总结

```
用户交互层：
┌─────────────────────────────────────┐
│              Host                   │
│        (用户看到的应用)              │
│                                     │
│  - 管理 UI                          │
│  - 协调 LLM                         │
│  - 决策工具调用                     │
└──────────────┬──────────────────────┘
               │
协议层：        │
    ┌──────────┼──────────┬──────────┐
    │          │          │          │
┌───▼───┐  ┌──▼───┐  ┌───▼───┐  ┌──▼───┐
│Client │  │Client│  │Client │  │Client│
│  1    │  │  2   │  │  3    │  │  4   │
└───┬───┘  └──┬───┘  └───┬───┘  └──┬───┘
    │         │          │         │
    │ MCP协议 │          │         │
    │         │          │         │
┌───▼───┐  ┌──▼───┐  ┌───▼───┐  ┌──▼───┐
│Server │  │Server│  │Server │  │Server│
│  1    │  │  2   │  │  3    │  │  4   │
└───┬───┘  └──┬───┘  └───┬───┘  └──┬───┘
    │         │          │         │
实现层：      │          │         │
    │         │          │         │
┌───▼─────────▼──────────▼─────────▼───┐
│        实际的业务逻辑和数据源          │
│  文件系统 | 数据库 | API | 服务...   │
└───────────────────────────────────────┘
```

**关键点**：
- **Host**：用户交互的入口，决策者
- **Client**：通信代理，一对一连接
- **Server**：能力提供者，实现具体功能

---

## 3. MCP 的三大核心能力

MCP Server 可以向 Client 暴露三种类型的能力：Tools、Resources 和 Prompts。

### 3.1 Tools（工具）

#### 什么是 Tools？

**Tools** 是可以被调用的函数，通常有明确的输入参数和输出结果。

#### Tools 的特点

```
特点：
✓ 可执行的操作
✓ 有明确的输入 schema
✓ 有明确的输出格式
✓ 可能有副作用（如修改数据）
```

#### Tools 的典型例子

**1. 搜索工具**
```json
{
  "name": "web_search",
  "description": "在互联网上搜索信息",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "搜索关键词"
      },
      "max_results": {
        "type": "integer",
        "description": "最多返回结果数",
        "default": 10
      }
    },
    "required": ["query"]
  }
}
```

**2. 数据库查询工具**
```json
{
  "name": "execute_sql",
  "description": "执行 SQL 查询",
  "inputSchema": {
    "type": "object",
    "properties": {
      "sql": {
        "type": "string",
        "description": "SQL 查询语句"
      },
      "database": {
        "type": "string",
        "description": "数据库名称"
      }
    },
    "required": ["sql"]
  }
}
```

**3. 文件操作工具**
```json
{
  "name": "read_file",
  "description": "读取文件内容",
  "inputSchema": {
    "type": "object",
    "properties": {
      "path": {
        "type": "string",
        "description": "文件路径"
      },
      "encoding": {
        "type": "string",
        "description": "文件编码",
        "default": "utf-8"
      }
    },
    "required": ["path"]
  }
}
```

#### Tool 的调用流程

```
1. Client 发现工具
   ↓
   Client: tools/list
   Server: 返回所有工具列表

2. LLM 决定调用
   ↓
   User: "帮我搜索 MCP 相关信息"
   LLM: 决定调用 web_search 工具

3. Client 调用工具
   ↓
   Client: tools/call
   {
     "name": "web_search",
     "arguments": {
       "query": "Model Context Protocol",
       "max_results": 5
     }
   }

4. Server 执行并返回
   ↓
   Server: 返回搜索结果
   {
     "content": [
       {
         "type": "text",
         "text": "找到以下结果：\n1. ..."
       }
     ]
   }

5. LLM 生成最终回答
   ↓
   LLM: "根据搜索结果，MCP 是..."
```

#### 实现一个 Tool（Python）

```python
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("example-server")

@server.tool()
async def calculate_bmi(
    weight_kg: float,
    height_m: float
) -> str:
    """
    计算身体质量指数（BMI）
    
    Args:
        weight_kg: 体重（千克）
        height_m: 身高（米）
    
    Returns:
        BMI 值和健康建议
    """
    bmi = weight_kg / (height_m ** 2)
    
    if bmi < 18.5:
        category = "偏瘦"
    elif bmi < 24:
        category = "正常"
    elif bmi < 28:
        category = "偏胖"
    else:
        category = "肥胖"
    
    return f"您的 BMI 为 {bmi:.2f}，属于{category}范围"
```

#### Tool 设计最佳实践

**1. 单一职责**
```python
# ✅ 好的设计：每个工具做一件事
@server.tool()
async def get_user(user_id: str):
    """获取用户信息"""
    pass

@server.tool()
async def update_user(user_id: str, data: dict):
    """更新用户信息"""
    pass

# ❌ 不好的设计：一个工具做太多事
@server.tool()
async def manage_user(action: str, user_id: str, data: dict):
    """管理用户（获取/更新/删除）"""
    pass
```

**2. 清晰的参数**
```python
# ✅ 好的设计：参数明确
@server.tool()
async def search_products(
    keyword: str,
    category: str,
    min_price: float,
    max_price: float,
    sort_by: str = "relevance"
):
    pass

# ❌ 不好的设计：参数模糊
@server.tool()
async def search(query: dict):
    pass
```

**3. 详细的文档**
```python
# ✅ 好的设计：有详细说明
@server.tool()
async def send_email(
    to: str,
    subject: str,
    body: str,
    cc: list[str] = None
) -> str:
    """
    发送电子邮件
    
    Args:
        to: 收件人邮箱地址
        subject: 邮件主题
        body: 邮件正文（支持 HTML）
        cc: 抄送地址列表（可选）
    
    Returns:
        发送状态消息
    
    Raises:
        ValueError: 如果邮箱地址格式不正确
        SMTPError: 如果发送失败
    """
    pass
```

### 3.2 Resources（资源）

#### 什么是 Resources？

**Resources** 是可以通过 URI 访问的内容，通常是静态或半静态的数据。

#### Resources 的特点

```
特点：
✓ 通过 URI 标识
✓ 通常是只读的
✓ 可以是静态或动态生成
✓ 支持模板化 URI
```

#### Resources 的类型

**1. 静态资源**
```
URI: company://policies
内容: 公司规章制度文档（固定内容）

URI: docs://api-reference
内容: API 文档（固定内容）
```

**2. 模板资源**
```
URI 模板: user://{user_id}
实际 URI: user://12345
内容: 用户 12345 的信息（动态生成）

URI 模板: file://{path}
实际 URI: file:///home/user/document.txt
内容: 指定路径的文件内容（动态读取）
```

#### Resources 的典型例子

**1. 知识库资源**
```json
{
  "uri": "kb://company-handbook",
  "name": "公司手册",
  "description": "公司员工手册完整内容",
  "mimeType": "text/markdown"
}
```

**2. 配置资源**
```json
{
  "uri": "config://database",
  "name": "数据库配置",
  "description": "当前数据库连接配置",
  "mimeType": "application/json"
}
```

**3. 动态资源模板**
```json
{
  "uriTemplate": "user://{user_id}/profile",
  "name": "用户资料",
  "description": "获取指定用户的个人资料",
  "mimeType": "application/json"
}
```

#### Resource 的访问流程

```
1. Client 发现资源
   ↓
   Client: resources/list
   Server: 返回所有资源列表

2. Client 读取资源
   ↓
   Client: resources/read
   {
     "uri": "kb://company-handbook"
   }

3. Server 返回内容
   ↓
   Server: 
   {
     "contents": [
       {
         "uri": "kb://company-handbook",
         "mimeType": "text/markdown",
         "text": "# 公司手册\n\n## 第一章..."
       }
     ]
   }
```

#### 实现 Resources（Python）

```python
from mcp.server import Server

server = Server("knowledge-server")

# 静态资源
@server.resource("company://policies")
async def get_company_policies() -> str:
    """公司政策文档"""
    return """
    # 公司政策
    
    ## 考勤制度
    - 工作时间：9:00-18:00
    - 弹性工作：支持
    
    ## 假期制度
    - 年假：15天
    - 病假：按实际情况
    """

# 动态资源（模板）
@server.resource("user://{user_id}/profile")
async def get_user_profile(user_id: str) -> dict:
    """获取用户资料"""
    # 从数据库查询
    user = await db.get_user(user_id)
    return {
        "id": user.id,
        "name": user.name,
        "email": user.email,
        "department": user.department
    }

# 文件资源
@server.resource("file://{path}")
async def read_file(path: str) -> str:
    """读取文件内容"""
    with open(path, 'r', encoding='utf-8') as f:
        return f.read()
```

#### Resources vs Tools 的区别

| 特性 | Resources | Tools |
|------|-----------|-------|
| **用途** | 提供数据/内容 | 执行操作 |
| **访问方式** | URI | 函数名 + 参数 |
| **副作用** | 通常无 | 可能有 |
| **缓存** | 适合缓存 | 不适合缓存 |
| **示例** | 文档、配置、知识库 | 搜索、计算、修改数据 |

**何时使用 Resources？**
- ✅ 提供文档或知识库内容
- ✅ 暴露配置信息
- ✅ 提供静态或半静态数据
- ✅ 内容可以被 LLM 直接理解

**何时使用 Tools？**
- ✅ 需要执行计算或操作
- ✅ 需要修改状态
- ✅ 需要复杂的参数验证
- ✅ 需要实时查询外部系统

### 3.3 Prompts（提示模板）

#### 什么是 Prompts？

**Prompts** 是可参数化的提示模板，用于标准化常见的 LLM 交互模式。

#### Prompts 的特点

```
特点：
✓ 可复用的提示模板
✓ 支持参数化
✓ 标准化常见任务
✓ 提高一致性
```

#### Prompts 的典型例子

**1. 代码审查提示**
```json
{
  "name": "code-review",
  "description": "审查代码质量和最佳实践",
  "arguments": [
    {
      "name": "code",
      "description": "要审查的代码",
      "required": true
    },
    {
      "name": "language",
      "description": "编程语言",
      "required": true
    }
  ]
}
```

**使用效果**：
```
输入参数：
- code: "def add(a,b): return a+b"
- language: "Python"

生成的提示：
"请审查以下 Python 代码的质量和最佳实践：

```python
def add(a,b): return a+b
```

请关注：
1. 代码风格
2. 命名规范
3. 类型提示
4. 文档字符串
5. 潜在问题
"
```

**2. 翻译提示**
```json
{
  "name": "translate",
  "description": "将文本翻译成目标语言",
  "arguments": [
    {
      "name": "text",
      "description": "要翻译的文本",
      "required": true
    },
    {
      "name": "target_language",
      "description": "目标语言",
      "required": true
    },
    {
      "name": "style",
      "description": "翻译风格（正式/口语）",
      "required": false
    }
  ]
}
```

**3. 数据分析提示**
```json
{
  "name": "analyze-data",
  "description": "分析数据并提供洞察",
  "arguments": [
    {
      "name": "data",
      "description": "数据（JSON 格式）",
      "required": true
    },
    {
      "name": "focus",
      "description": "分析重点",
      "required": false
    }
  ]
}
```

#### Prompt 的使用流程

```
1. Client 发现提示
   ↓
   Client: prompts/list
   Server: 返回所有提示模板

2. Client 获取提示
   ↓
   Client: prompts/get
   {
     "name": "code-review",
     "arguments": {
       "code": "def add(a,b): return a+b",
       "language": "Python"
     }
   }

3. Server 返回生成的提示
   ↓
   Server:
   {
     "messages": [
       {
         "role": "user",
         "content": {
           "type": "text",
           "text": "请审查以下 Python 代码..."
         }
       }
     ]
   }

4. Host 将提示发送给 LLM
   ↓
   LLM 根据提示生成回答
```

#### 实现 Prompts（Python）

```python
from mcp.server import Server
from mcp.types import PromptMessage

server = Server("prompt-server")

@server.prompt()
async def code_review(code: str, language: str) -> list[PromptMessage]:
    """代码审查提示"""
    return [
        PromptMessage(
            role="user",
            content={
                "type": "text",
                "text": f"""请审查以下 {language} 代码的质量和最佳实践：

```{language.lower()}
{code}
```

请关注：
1. 代码风格和格式
2. 命名规范
3. 类型提示（如适用）
4. 文档和注释
5. 潜在的 bug 或性能问题
6. 安全性考虑

请提供具体的改进建议。"""
            }
        )
    ]

@server.prompt()
async def translate(
    text: str,
    target_language: str,
    style: str = "正式"
) -> list[PromptMessage]:
    """翻译提示"""
    return [
        PromptMessage(
            role="user",
            content={
                "type": "text",
                "text": f"""请将以下文本翻译成{target_language}，使用{style}风格：

原文：
{text}

要求：
1. 准确传达原意
2. 符合目标语言的表达习惯
3. 保持{style}的语气
4. 如有专业术语，请保留或注释"""
            }
        )
    ]

@server.prompt()
async def sql_expert(
    question: str,
    schema: str
) -> list[PromptMessage]:
    """SQL 专家提示"""
    return [
        PromptMessage(
            role="system",
            content={
                "type": "text",
                "text": "你是一个 SQL 专家，擅长编写高效、安全的 SQL 查询。"
            }
        ),
        PromptMessage(
            role="user",
            content={
                "type": "text",
                "text": f"""数据库 Schema：
{schema}

用户问题：
{question}

请提供：
1. SQL 查询语句
2. 查询说明
3. 性能考虑
4. 注意事项"""
            }
        )
    ]
```

#### Prompts 的最佳实践

**1. 清晰的参数定义**
```python
# ✅ 好的设计
@server.prompt()
async def summarize(
    text: str,              # 必需参数
    max_length: int = 200,  # 可选参数，有默认值
    style: str = "简洁"     # 可选参数，有默认值
):
    pass

# ❌ 不好的设计
@server.prompt()
async def summarize(options: dict):  # 参数不明确
    pass
```

**2. 结构化的提示**
```python
# ✅ 好的设计：结构清晰
return [
    PromptMessage(
        role="system",
        content={"type": "text", "text": "你是专家..."}
    ),
    PromptMessage(
        role="user",
        content={"type": "text", "text": f"""
任务：{task}

输入：
{input_data}

要求：
1. ...
2. ...
"""}
    )
]

# ❌ 不好的设计：混乱的提示
return [
    PromptMessage(
        role="user",
        content={"type": "text", "text": f"{task} {input_data} 请帮我..."}
    )
]
```

**3. 可复用性**
```python
# ✅ 好的设计：通用的模板
@server.prompt()
async def analyze(
    data_type: str,  # 数据类型
    data: str,       # 数据内容
    focus: str       # 分析重点
):
    """通用分析提示，适用于多种数据类型"""
    pass

# ❌ 不好的设计：过于具体
@server.prompt()
async def analyze_sales_data_for_q1_2024():
    """只能分析 2024 Q1 销售数据"""
    pass
```

---

## 4. MCP 的架构设计

### 4.1 整体架构图

```
┌─────────────────────────────────────────────────────────┐
│                    用户层                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  用户 1  │  │  用户 2  │  │  用户 3  │              │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘              │
└───────┼─────────────┼─────────────┼────────────────────┘
        │             │             │
┌───────▼─────────────▼─────────────▼────────────────────┐
│                  应用层（Host）                          │
│  ┌─────────────────────────────────────────────────┐   │
│  │              AI 应用 / IDE                       │   │
│  │  - 用户界面                                      │   │
│  │  - LLM 集成                                      │   │
│  │  - 工具调用协调                                  │   │
│  └─────────────────────────────────────────────────┘   │
│                                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │Client 1 │  │Client 2 │  │Client 3 │  │Client 4 │   │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘   │
└───────┼────────────┼────────────┼────────────┼─────────┘
        │            │            │            │
        │ MCP 协议   │            │            │
        │            │            │            │
┌───────▼────────────▼────────────▼────────────▼─────────┐
│                  协议层（MCP）                           │
│  - JSON-RPC 2.0 消息格式                                │
│  - 标准化的请求/响应                                    │
│  - 能力协商                                             │
│  - 生命周期管理                                         │
└───────┬────────────┬────────────┬────────────┬─────────┘
        │            │            │            │
┌───────▼────┐  ┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│  Server 1  │  │Server 2│  │Server 3│  │Server 4│
│  文件系统  │  │ 数据库 │  │ GitHub │  │  Slack │
└───────┬────┘  └───┬────┘  └───┬────┘  └───┬────┘
        │           │           │           │
┌───────▼───────────▼───────────▼───────────▼─────────┐
│                  实现层                               │
│  - 具体业务逻辑                                       │
│  - 外部系统集成                                       │
│  - 数据处理                                           │
└───────────────────────────────────────────────────────┘
```

### 4.2 通信模式

#### 模式 1：一对一连接

```
每个 Client 只连接一个 Server

Host
 ├─ Client A ──── Server A
 ├─ Client B ──── Server B
 └─ Client C ──── Server C

特点：
✓ 简单清晰
✓ 隔离性好
✓ 易于管理
```

#### 模式 2：能力聚合

```
Host 聚合所有 Server 的能力

┌─────────────────────────────┐
│          Host               │
│                             │
│  所有工具：                 │
│  - Server A 的工具 1-5      │
│  - Server B 的工具 6-10     │
│  - Server C 的工具 11-15    │
│                             │
│  LLM 可以调用任何工具       │
└─────────────────────────────┘
```

### 4.3 会话生命周期

```
1. 启动阶段
   ┌─────────┐
   │ Host    │ 启动应用
   │ 启动    │
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │ Client  │ 创建 Client 实例
   │ 创建    │
   └────┬────┘
        │
        ▼
   ┌─────────┐
   │ Server  │ 启动 Server 进程
   │ 启动    │
   └─────────┘

2. 初始化阶段
   Client ──initialize──> Server
          <──response───
          
   交换信息：
   - 协议版本
   - 支持的能力
   - 客户端/服务器信息

3. 能力发现阶段
   Client ──list_tools──> Server
          <──tools list──
          
   Client ──list_resources──> Server
          <──resources list──
          
   Client ──list_prompts──> Server
          <──prompts list──

4. 活跃阶段
   循环：
   - 用户输入
   - LLM 决策
   - 工具调用
   - 结果返回
   - 生成回答

5. 关闭阶段
   Client ──close──> Server
   释放资源
```

### 4.4 错误处理

```
错误传播链：

Server 错误
    ↓
转换为标准 MCP 错误
    ↓
通过协议返回给 Client
    ↓
Client 处理错误
    ↓
Host 决定如何展示
    ↓
用户看到友好的错误信息
```

**标准错误格式**：
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32600,
    "message": "Invalid Request",
    "data": {
      "details": "Missing required parameter: city"
    }
  }
}
```

---

## 5. 连接与传输方式

MCP 支持多种传输方式，每种都有其适用场景。

### 5.1 Stdio（标准输入输出）

#### 什么是 Stdio？

**Stdio** 是通过进程的标准输入输出进行通信的方式。

#### 工作原理

```
┌─────────────────┐
│   Host Process  │
│                 │
│  ┌───────────┐  │
│  │  Client   │  │
│  └─────┬─────┘  │
└────────┼────────┘
         │
    stdin/stdout
         │
┌────────▼────────┐
│ Server Process  │
│                 │
│  ┌───────────┐  │
│  │  Server   │  │
│  └───────────┘  │
└─────────────────┘

通信方式：
Client 写入 → Server 的 stdin
Server 写入 → Client 的 stdout
```

#### 优点和缺点

**优点**：
- ✅ 简单易实现
- ✅ 跨平台支持好
- ✅ 适合本地进程
- ✅ 无需网络配置

**缺点**：
- ❌ 只能本地通信
- ❌ 不支持远程连接
- ❌ 进程管理复杂

#### 使用场景

```
适合：
✓ 本地工具集成
✓ IDE 插件
✓ 桌面应用
✓ 命令行工具

不适合：
✗ 远程服务
✗ Web 应用
✗ 分布式系统
```

#### 配置示例（Claude Desktop）

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Documents"
      ]
    },
    "database": {
      "command": "python",
      "args": [
        "-m",
        "mcp_server_database"
      ],
      "env": {
        "DB_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

### 5.2 SSE（Server-Sent Events）

#### 什么是 SSE？

**SSE** 是基于 HTTP 的服务器推送技术，允许服务器主动向客户端发送消息。

#### 工作原理

```
┌─────────────────┐
│     Client      │
│   (Browser/     │
│    App)         │
└────────┬────────┘
         │
    HTTP/HTTPS
         │
┌────────▼────────┐
│   MCP Server    │
│   (HTTP Server) │
└─────────────────┘

通信方式：
Client → Server: HTTP POST 请求
Server → Client: SSE 事件流
```

#### 优点和缺点

**优点**：
- ✅ 支持远程连接
- ✅ 基于标准 HTTP
- ✅ 支持服务器推送
- ✅ 防火墙友好

**缺点**：
- ❌ 需要 HTTP 服务器
- ❌ 配置相对复杂
- ❌ 需要处理网络问题

#### 使用场景

```
适合：
✓ Web 应用
✓ 远程服务
✓ 云端部署
✓ 需要服务器推送

不适合：
✗ 简单本地工具
✗ 无网络环境
```

### 5.3 传输方式对比

| 特性 | Stdio | SSE | WebSocket |
|------|-------|-----|-----------|
| **部署** | 简单 | 中等 | 中等 |
| **远程** | ❌ | ✅ | ✅ |
| **双向** | ✅ | 部分 | ✅ |
| **推送** | ✅ | ✅ | ✅ |
| **防火墙** | N/A | 友好 | 可能被阻 |
| **适用场景** | 本地 | Web/远程 | 实时应用 |

---

## 6. 实战练习：画出架构图

### 6.1 练习目标

通过绘制架构图，加深对 MCP 整体架构的理解。

### 6.2 练习任务

**任务 1：绘制基础架构图**

画出包含以下元素的架构图：
- 1 个 Host（AI 应用）
- 3 个 Client
- 3 个 Server（文件系统、数据库、GitHub）
- 标注通信方式

**任务 2：绘制工具调用流程图**

画出一次完整的工具调用流程：
1. 用户输入问题
2. LLM 决定调用工具
3. Client 发送请求
4. Server 执行并返回
5. LLM 生成最终回答

**任务 3：绘制能力发现流程**

画出 Client 如何发现 Server 的所有能力：
- Tools 发现
- Resources 发现
- Prompts 发现

### 6.3 参考答案

**基础架构图**：

```
用户
 ↓
┌──────────────────────────────────┐
│        AI 应用 (Host)            │
│                                  │
│  ┌────────┐ ┌────────┐ ┌────────┐│
│  │Client 1│ │Client 2│ │Client 3││
│  └───┬────┘ └───┬────┘ └───┬────┘│
└──────┼──────────┼──────────┼─────┘
       │          │          │
     stdio      stdio      stdio
       │          │          │
  ┌────▼───┐ ┌───▼────┐ ┌───▼────┐
  │文件系统│ │ 数据库 │ │ GitHub │
  │ Server │ │ Server │ │ Server │
  └────────┘ └────────┘ └────────┘
```

**工具调用流程图**：

```
1. 用户: "查询北京的天气"
   ↓
2. Host → LLM: 处理请求
   ↓
3. LLM: 决定调用 get_weather 工具
   ↓
4. Host → Client: call_tool("get_weather", {"city": "北京"})
   ↓
5. Client → Server: 发送 MCP 请求
   ↓
6. Server: 调用天气 API
   ↓
7. Server → Client: 返回结果 "晴天，25°C"
   ↓
8. Client → Host: 转发结果
   ↓
9. Host → LLM: 继续对话，附带工具结果
   ↓
10. LLM: 生成回答 "北京今天天气晴朗，温度25度..."
    ↓
11. Host → 用户: 显示最终回答
```

---

## 7. 总结与检查清单

### 7.1 核心概念总结

**MCP 的本质**：
- 一个开放的协议标准
- 连接 AI 应用和外部工具的桥梁
- 类似于 AI 世界的 USB-C 接口

**三大角色**：
1. **Host**：用户交互的应用，决策者
2. **Client**：通信代理，连接 Host 和 Server
3. **Server**：能力提供者，实现具体功能

**三大能力**：
1. **Tools**：可执行的函数，有输入输出
2. **Resources**：可访问的内容，通过 URI 标识
3. **Prompts**：可复用的提示模板，标准化任务

**架构特点**：
- 一对一连接（每个 Client 连接一个 Server）
- 能力聚合（Host 汇总所有能力）
- 标准化通信（JSON-RPC 2.0）
- 多种传输方式（Stdio、SSE 等）

### 7.2 阶段 1 完成检查清单

完成以下所有项目，即可进入阶段 2：

- [ ] 能用自己的话解释 MCP 是什么
- [ ] 理解 MCP 解决的核心问题
- [ ] 清楚 Host、Client、Server 的角色和职责
- [ ] 理解 Tools、Resources、Prompts 的区别和用途
- [ ] 知道何时使用 Tools，何时使用 Resources
- [ ] 了解 MCP 的整体架构设计
- [ ] 理解一对一连接模式
- [ ] 知道 Stdio 和 SSE 的区别
- [ ] 能画出 MCP 的基础架构图
- [ ] 能描述一次工具调用的完整流程

### 7.3 自我测试题

**问题 1**：用一句话解释 MCP 是什么？

<details>
<summary>参考答案</summary>

MCP 是一个开放协议，用于标准化 AI 应用如何与外部工具和数据源进行交互，就像 USB-C 为设备提供统一接口一样。
</details>

**问题 2**：Host、Client、Server 各自的主要职责是什么？

<details>
<summary>参考答案</summary>

- **Host**：管理用户界面，协调 LLM 和工具调用，决策何时调用哪个工具
- **Client**：作为通信代理，一对一连接 Server，处理协议层通信
- **Server**：提供具体功能，暴露 Tools/Resources/Prompts，执行实际业务逻辑
</details>

**问题 3**：什么时候应该使用 Resource 而不是 Tool？

<details>
<summary>参考答案</summary>

当你需要提供静态或半静态的内容（如文档、配置、知识库）时使用 Resource。当需要执行操作、计算或修改状态时使用 Tool。
</details>

**问题 4**：Stdio 和 SSE 传输方式的主要区别是什么？

<details>
<summary>参考答案</summary>

Stdio 用于本地进程间通信，简单但不支持远程；SSE 基于 HTTP，支持远程连接和服务器推送，适合 Web 应用。
</details>

**问题 5**：一个 Host 可以连接多少个 Server？

<details>
<summary>参考答案</summary>

一个 Host 可以通过多个 Client 连接任意数量的 Server，每个 Client 与一个 Server 建立一对一连接。
</details>

### 7.4 实践建议

**1. 动手画图**
- 在纸上或使用工具（如 draw.io、Excalidraw）画出架构图
- 标注每个组件的职责
- 画出数据流向

**2. 思考实际场景**
- 想象你要开发一个 AI 助手
- 列出需要哪些工具
- 思考如何组织这些工具

**3. 阅读示例代码**
- 查看官方 SDK 的 examples 目录
- 理解每个示例的架构
- 尝试修改参数运行

**4. 记录学习笔记**
- 用自己的话总结核心概念
- 记录不理解的地方
- 准备问题以便后续查询

---

## 📝 下一步

恭喜你完成了阶段 1！现在你已经：

✅ 深入理解了 MCP 的本质和价值  
✅ 掌握了 Host、Client、Server 三大角色  
✅ 理解了 Tools、Resources、Prompts 三大能力  
✅ 了解了 MCP 的整体架构设计  
✅ 知道了不同的传输方式及其适用场景  

**接下来**，你将进入：

👉 **阶段 2：协议规范与消息模型**

在阶段 2 中，你将学习：
- MCP 协议的详细规范
- JSON-RPC 2.0 消息格式
- 初始化和能力协商流程
- 请求/响应/通知的区别
- 错误处理机制
- 完整的消息交互流程

---

## 📚 附录

### A. 术语表

| 术语 | 英文 | 解释 |
|------|------|------|
| **宿主** | Host | 用户直接交互的应用程序 |
| **客户端** | Client | Host 内部的通信组件 |
| **服务器** | Server | 提供具体功能的服务进程 |
| **工具** | Tool | 可调用的函数 |
| **资源** | Resource | 可访问的内容 |
| **提示** | Prompt | 可复用的提示模板 |
| **传输** | Transport | 通信方式（Stdio/SSE等） |
| **能力** | Capability | Server 提供的功能 |
| **会话** | Session | Client 和 Server 的连接 |

### B. 快速参考

**MCP 核心概念速查**：

```
MCP = 协议标准
  ├─ 角色
  │   ├─ Host（决策者）
  │   ├─ Client（通信代理）
  │   └─ Server（能力提供者）
  ├─ 能力
  │   ├─ Tools（可执行函数）
  │   ├─ Resources（可访问内容）
  │   └─ Prompts（提示模板）
  └─ 传输
      ├─ Stdio（本地）
      ├─ SSE（远程）
      └─ WebSocket（实时）
```

**工具调用流程速查**：

```
用户输入
  ↓
Host 接收
  ↓
LLM 分析
  ↓
决定调用工具
  ↓
Client 发送请求
  ↓
Server 执行
  ↓
Server 返回结果
  ↓
Client 转发
  ↓
LLM 生成回答
  ↓
Host 展示给用户
```

### C. 推荐资源

**官方资源**：
- MCP 规范：https://modelcontextprotocol.io/specification
- Python SDK：https://github.com/modelcontextprotocol/python-sdk
- TypeScript SDK：https://github.com/modelcontextprotocol/typescript-sdk

**社区资源**：
- MCP Servers 列表：https://github.com/modelcontextprotocol/servers
- 示例项目：查看各 SDK 的 examples 目录

**学习建议**：
1. 先理解概念，再看代码
2. 多画图，帮助理解架构
3. 从简单示例开始实践
4. 逐步增加复杂度

---

## 🎓 学习心得记录区

**建议在这里记录你的学习心得**：

```
日期：____________________

今天学到的最重要的概念：
1. 
2. 
3. 

还不太理解的地方：
1. 
2. 

计划下一步：
1. 
2. 
```

---

**祝你学习愉快！🚀**

> 本文档最后更新：2025年11月
> 
> 如有任何问题或建议，欢迎反馈！
