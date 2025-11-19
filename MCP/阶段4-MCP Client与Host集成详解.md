# 阶段 4：MCP Client/Host 集成详解

> **学习目标**：掌握 MCP Client 开发、Host 应用集成、多 Server 管理和高级集成模式

---

## 目录

1. [Client 开发基础](#1-client-开发基础)
2. [Client 会话管理](#2-client-会话管理)
3. [多 Server 管理](#3-多-server-管理)
4. [Host 应用集成](#4-host-应用集成)
5. [高级集成模式](#5-高级集成模式)
6. [最佳实践与总结](#6-最佳实践与总结)

---

## 1. Client 开发基础

### 1.1 Client 的角色

在 MCP 架构中，Client 是连接 Host 应用和 Server 的桥梁。

#### MCP 三层架构

```
┌─────────────────────────────────────┐
│         Host Application            │  ← 你的应用（IDE、聊天机器人等）
│  (Claude Desktop, IDE, Custom App)  │
└──────────────┬──────────────────────┘
               │
               │ 集成层
               ↓
┌─────────────────────────────────────┐
│         MCP Client                  │  ← 本阶段重点
│  - 连接管理                          │
│  - 协议处理                          │
│  - 多 Server 协调                    │
└──────────────┬──────────────────────┘
               │
               │ MCP 协议
               ↓
┌─────────────────────────────────────┐
│         MCP Servers                 │
│  (文件系统、数据库、API 等)           │
└─────────────────────────────────────┘
```

#### Client 的核心职责

```
1. 传输管理
   ├─ Stdio（标准输入输出）
   ├─ HTTP/SSE（服务器推送事件）
   ├─ StreamableHTTP（流式 HTTP）
   └─ WebSocket（双向通信）

2. 会话管理
   ├─ 初始化握手
   ├─ 能力协商
   ├─ 心跳保持
   └─ 优雅关闭

3. 请求处理
   ├─ 工具调用
   ├─ 资源读取
   ├─ 提示获取
   └─ 采样请求

4. 错误处理
   ├─ 连接错误
   ├─ 协议错误
   ├─ 超时处理
   └─ 重连机制
```

### 1.2 环境准备

#### Python 环境

```bash
# 安装 MCP Python SDK
pip install mcp

# 或使用 uv（推荐）
uv pip install mcp

# 验证安装
python -c "import mcp; print(mcp.__version__)"
```

#### TypeScript/Node.js 环境

```bash
# 安装 MCP TypeScript SDK
npm install @modelcontextprotocol/sdk

# 或使用 pnpm（推荐）
pnpm add @modelcontextprotocol/sdk

# 验证安装
node -e "console.log(require('@modelcontextprotocol/sdk/package.json').version)"
```

### 1.3 第一个 Client：连接到 Server

#### Python：基础 Client 实现

```python
# basic_client.py

import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client


async def main():
    """基础 MCP Client 示例"""
    
    # 1. 配置 Server 参数
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_mcp_server"]
    )
    
    # 2. 建立连接
    async with stdio_client(server_params) as (read, write):
        # 3. 创建会话
        async with ClientSession(read, write) as session:
            # 4. 初始化连接
            init_result = await session.initialize()
            
            print(f"✅ 连接成功！")
            print(f"Server: {init_result.serverInfo.name}")
            print(f"Version: {init_result.serverInfo.version}")
            print(f"Protocol: {init_result.protocolVersion}")
            
            # 5. 列出可用工具
            tools_result = await session.list_tools()
            print(f"\n📦 可用工具 ({len(tools_result.tools)}):")
            for tool in tools_result.tools:
                print(f"  - {tool.name}: {tool.description}")
            
            # 6. 调用工具
            if tools_result.tools:
                tool_name = tools_result.tools[0].name
                result = await session.call_tool(
                    tool_name,
                    arguments={"param": "test"}
                )
                print(f"\n🔧 工具调用结果:")
                print(f"  {result.content[0].text}")


if __name__ == "__main__":
    asyncio.run(main())
```

#### TypeScript：基础 Client 实现

```typescript
// basicClient.ts

import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function main() {
    // 1. 创建 Client 实例
    const client = new Client({
        name: 'my-client',
        version: '1.0.0'
    });
    
    // 2. 配置传输层
    const transport = new StdioClientTransport({
        command: 'node',
        args: ['server.js']
    });
    
    try {
        // 3. 连接到 Server
        await client.connect(transport);
        console.log('✅ 连接成功！');
        
        // 4. 列出工具
        const tools = await client.listTools();
        console.log(`\n📦 可用工具 (${tools.tools.length}):`);
        tools.tools.forEach(tool => {
            console.log(`  - ${tool.name}: ${tool.description}`);
        });
        
        // 5. 调用工具
        if (tools.tools.length > 0) {
            const result = await client.callTool({
                name: tools.tools[0].name,
                arguments: { param: 'test' }
            });
            console.log('\n🔧 工具调用结果:');
            console.log(`  ${result.content[0].text}`);
        }
        
    } catch (error) {
        console.error('❌ 错误:', error);
    } finally {
        // 6. 关闭连接
        await client.close();
        console.log('\n👋 连接已关闭');
    }
}

main();
```

### 1.4 传输层详解

MCP 支持多种传输方式，适用于不同场景。

#### 1. Stdio 传输（本地进程）

**适用场景**：本地 Server，如 CLI 工具、桌面应用

```python
# Python Stdio Client
from mcp import StdioServerParameters
from mcp.client.stdio import stdio_client

server_params = StdioServerParameters(
    command="uv",
    args=["run", "python", "-m", "my_server"],
    env={"DEBUG": "1"}  # 可选环境变量
)

async with stdio_client(server_params) as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        # 使用 session...
```

```typescript
// TypeScript Stdio Client
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
    command: 'node',
    args: ['server.js'],
    env: { DEBUG: '1' }  // 可选环境变量
});

await client.connect(transport);
```

#### 2. HTTP/SSE 传输（远程 Server）

**适用场景**：远程 Server，Web 服务

```python
# Python HTTP Client（需要额外库）
from mcp.client.sse import sse_client

async with sse_client("http://localhost:3000/sse") as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        # 使用 session...
```

```typescript
// TypeScript SSE Client
import { SSEClientTransport } from '@modelcontextprotocol/sdk/client/sse.js';

const transport = new SSEClientTransport(
    new URL('http://localhost:3000/sse')
);

await client.connect(transport);
```

#### 3. StreamableHTTP 传输（推荐）

**适用场景**：现代 Web 应用，支持流式响应

```python
# Python StreamableHTTP Client
from mcp.client.streamable_http import streamablehttp_client

async with streamablehttp_client("http://localhost:3000/mcp") as (read, write, _):
    async with ClientSession(read, write) as session:
        await session.initialize()
        # 使用 session...
```

```typescript
// TypeScript StreamableHTTP Client
import { StreamableHTTPClientTransport } from '@modelcontextprotocol/sdk/client/streamableHttp.js';

const transport = new StreamableHTTPClientTransport(
    new URL('http://localhost:3000/mcp')
);

await client.connect(transport);
```

#### 4. WebSocket 传输（双向实时）

**适用场景**：需要实时双向通信的应用

```typescript
// TypeScript WebSocket Client
import { WebSocketClientTransport } from '@modelcontextprotocol/sdk/client/websocket.js';

const transport = new WebSocketClientTransport(
    new URL('ws://localhost:3000/mcp')
);

await client.connect(transport);
```

#### 传输方式对比

| 传输方式 | 适用场景 | 优点 | 缺点 |
|---------|---------|------|------|
| **Stdio** | 本地进程 | 简单、低延迟 | 仅限本地 |
| **HTTP/SSE** | 远程服务 | 标准协议、防火墙友好 | 单向推送 |
| **StreamableHTTP** | 现代 Web | 流式响应、高效 | 需要现代浏览器 |
| **WebSocket** | 实时应用 | 双向实时、低延迟 | 连接管理复杂 |

### 1.5 Client 生命周期

```python
async def client_lifecycle_demo():
    """Client 完整生命周期示例"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    print("1️⃣ 建立传输连接...")
    async with stdio_client(server_params) as (read, write):
        
        print("2️⃣ 创建会话...")
        async with ClientSession(read, write) as session:
            
            print("3️⃣ 初始化握手...")
            init_result = await session.initialize()
            print(f"   Server: {init_result.serverInfo.name}")
            print(f"   Capabilities: {init_result.capabilities}")
            
            print("4️⃣ 使用 Server 功能...")
            # 列出工具
            tools = await session.list_tools()
            print(f"   发现 {len(tools.tools)} 个工具")
            
            # 列出资源
            resources = await session.list_resources()
            print(f"   发现 {len(resources.resources)} 个资源")
            
            # 列出提示
            prompts = await session.list_prompts()
            print(f"   发现 {len(prompts.prompts)} 个提示")
            
            print("5️⃣ 会话结束，自动清理...")
        
        print("6️⃣ 传输连接关闭...")
    
    print("✅ Client 生命周期完成")
```

### 1.6 错误处理

```python
import asyncio
from mcp import ClientSession
from mcp.client.stdio import stdio_client, StdioServerParameters


async def robust_client():
    """带错误处理的健壮 Client"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    try:
        async with stdio_client(server_params) as (read, write):
            async with ClientSession(read, write) as session:
                # 初始化超时
                try:
                    await asyncio.wait_for(
                        session.initialize(),
                        timeout=10.0
                    )
                except asyncio.TimeoutError:
                    print("❌ 初始化超时")
                    return
                
                # 工具调用错误处理
                try:
                    result = await session.call_tool(
                        "nonexistent_tool",
                        arguments={}
                    )
                except Exception as e:
                    print(f"❌ 工具调用失败: {e}")
                    # 继续执行其他操作
                
                # 资源读取错误处理
                try:
                    resource = await session.read_resource(
                        "invalid://uri"
                    )
                except Exception as e:
                    print(f"❌ 资源读取失败: {e}")
    
    except ConnectionError as e:
        print(f"❌ 连接错误: {e}")
    except Exception as e:
        print(f"❌ 未知错误: {e}")
```

---

## 2. Client 会话管理

### 2.1 会话初始化详解

会话初始化是 Client 和 Server 建立连接后的第一步，涉及能力协商和信息交换。

#### 初始化流程

```
Client                                    Server
  │                                         │
  │  ──── initialize request ────>         │
  │  {                                      │
  │    protocolVersion: "2025-06-18",      │
  │    capabilities: {...},                │
  │    clientInfo: {...}                   │
  │  }                                      │
  │                                         │
  │  <──── initialize response ────        │
  │  {                                      │
  │    protocolVersion: "2025-06-18",      │
  │    capabilities: {...},                │
  │    serverInfo: {...}                   │
  │  }                                      │
  │                                         │
  │  ──── initialized notification ────>   │
  │                                         │
```

#### Python：详细初始化

```python
from mcp import ClientSession
from mcp.client.stdio import stdio_client, StdioServerParameters
from mcp.types import ClientCapabilities, Implementation


async def detailed_initialization():
    """详细的初始化过程"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # 发送初始化请求
            init_result = await session.initialize()
            
            # 检查协议版本
            print(f"协议版本: {init_result.protocolVersion}")
            
            # 检查 Server 信息
            server_info = init_result.serverInfo
            print(f"\nServer 信息:")
            print(f"  名称: {server_info.name}")
            print(f"  版本: {server_info.version}")
            
            # 检查 Server 能力
            capabilities = init_result.capabilities
            print(f"\nServer 能力:")
            
            if capabilities.tools:
                print(f"  ✅ 支持工具")
                if hasattr(capabilities.tools, 'listChanged'):
                    print(f"     - 支持工具列表变更通知")
            
            if capabilities.resources:
                print(f"  ✅ 支持资源")
                if hasattr(capabilities.resources, 'subscribe'):
                    print(f"     - 支持资源订阅")
                if hasattr(capabilities.resources, 'listChanged'):
                    print(f"     - 支持资源列表变更通知")
            
            if capabilities.prompts:
                print(f"  ✅ 支持提示")
                if hasattr(capabilities.prompts, 'listChanged'):
                    print(f"     - 支持提示列表变更通知")
            
            if capabilities.logging:
                print(f"  ✅ 支持日志")
            
            return session
```

#### TypeScript：详细初始化

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function detailedInitialization() {
    const client = new Client({
        name: 'detailed-client',
        version: '1.0.0',
        capabilities: {
            // 声明 Client 能力
            sampling: {},  // 支持采样
            roots: {       // 支持根目录列表
                listChanged: true
            }
        }
    });
    
    const transport = new StdioClientTransport({
        command: 'node',
        args: ['server.js']
    });
    
    await client.connect(transport);
    
    // 获取 Server 信息
    const serverInfo = client.getServerVersion();
    console.log('Server 信息:');
    console.log(`  名称: ${serverInfo?.name}`);
    console.log(`  版本: ${serverInfo?.version}`);
    
    // 检查 Server 能力
    const capabilities = client.getServerCapabilities();
    console.log('\nServer 能力:');
    
    if (capabilities?.tools) {
        console.log('  ✅ 支持工具');
    }
    if (capabilities?.resources) {
        console.log('  ✅ 支持资源');
    }
    if (capabilities?.prompts) {
        console.log('  ✅ 支持提示');
    }
    
    return client;
}
```

### 2.2 工具调用管理

#### 基础工具调用

```python
async def call_tools_example(session: ClientSession):
    """工具调用示例"""
    
    # 1. 列出所有工具
    tools_result = await session.list_tools()
    
    print(f"可用工具: {len(tools_result.tools)}")
    for tool in tools_result.tools:
        print(f"\n工具: {tool.name}")
        print(f"  描述: {tool.description}")
        print(f"  输入模式: {tool.inputSchema}")
    
    # 2. 调用工具（基础）
    result = await session.call_tool(
        "add",
        arguments={"a": 5, "b": 3}
    )
    
    # 3. 处理结果
    for content in result.content:
        if content.type == "text":
            print(f"文本结果: {content.text}")
        elif content.type == "image":
            print(f"图片结果: {content.data[:50]}...")
        elif content.type == "resource":
            print(f"资源结果: {content.resource.uri}")
    
    # 4. 处理结构化结果
    if result.structuredContent:
        print(f"结构化结果: {result.structuredContent}")
```

#### 批量工具调用

```python
import asyncio
from typing import List, Dict, Any


async def batch_tool_calls(
    session: ClientSession,
    calls: List[Dict[str, Any]]
) -> List[Any]:
    """批量并发调用工具"""
    
    async def call_single(call_info: Dict[str, Any]):
        try:
            result = await session.call_tool(
                call_info["name"],
                arguments=call_info.get("arguments", {})
            )
            return {
                "success": True,
                "tool": call_info["name"],
                "result": result
            }
        except Exception as e:
            return {
                "success": False,
                "tool": call_info["name"],
                "error": str(e)
            }
    
    # 并发执行所有调用
    results = await asyncio.gather(
        *[call_single(call) for call in calls],
        return_exceptions=True
    )
    
    return results


# 使用示例
async def batch_example(session: ClientSession):
    calls = [
        {"name": "add", "arguments": {"a": 1, "b": 2}},
        {"name": "multiply", "arguments": {"a": 3, "b": 4}},
        {"name": "divide", "arguments": {"a": 10, "b": 2}}
    ]
    
    results = await batch_tool_calls(session, calls)
    
    for result in results:
        if result["success"]:
            print(f"✅ {result['tool']}: {result['result']}")
        else:
            print(f"❌ {result['tool']}: {result['error']}")
```

### 2.3 资源读取管理

#### 基础资源读取

```python
from pydantic import AnyUrl


async def read_resources_example(session: ClientSession):
    """资源读取示例"""
    
    # 1. 列出所有资源
    resources_result = await session.list_resources()
    
    print(f"可用资源: {len(resources_result.resources)}")
    for resource in resources_result.resources:
        print(f"\n资源: {resource.uri}")
        print(f"  名称: {resource.name}")
        print(f"  描述: {resource.description}")
        print(f"  MIME 类型: {resource.mimeType}")
    
    # 2. 读取静态资源
    if resources_result.resources:
        uri = resources_result.resources[0].uri
        content = await session.read_resource(uri)
        
        for item in content.contents:
            if item.uri:
                print(f"URI: {item.uri}")
            if hasattr(item, 'text'):
                print(f"文本内容: {item.text[:100]}...")
            if hasattr(item, 'blob'):
                print(f"二进制内容: {len(item.blob)} 字节")
    
    # 3. 读取模板资源（动态）
    template_uri = AnyUrl("user://123/profile")
    try:
        content = await session.read_resource(template_uri)
        print(f"模板资源内容: {content.contents[0].text}")
    except Exception as e:
        print(f"读取模板资源失败: {e}")
```

#### 资源订阅（监听变更）

```python
async def subscribe_to_resource(session: ClientSession):
    """订阅资源变更"""
    
    # 订阅资源
    uri = AnyUrl("file:///path/to/file.txt")
    await session.subscribe_resource(uri)
    
    print(f"✅ 已订阅资源: {uri}")
    
    # 在实际应用中，你需要设置通知处理器
    # 当资源变更时，Server 会发送通知
    
    # 取消订阅
    await session.unsubscribe_resource(uri)
    print(f"❌ 已取消订阅: {uri}")
```

### 2.4 提示管理

#### 获取和使用提示

```python
async def prompts_example(session: ClientSession):
    """提示使用示例"""
    
    # 1. 列出所有提示
    prompts_result = await session.list_prompts()
    
    print(f"可用提示: {len(prompts_result.prompts)}")
    for prompt in prompts_result.prompts:
        print(f"\n提示: {prompt.name}")
        print(f"  描述: {prompt.description}")
        if prompt.arguments:
            print(f"  参数:")
            for arg in prompt.arguments:
                print(f"    - {arg.name}: {arg.description}")
                print(f"      必需: {arg.required}")
    
    # 2. 获取提示
    if prompts_result.prompts:
        prompt_name = prompts_result.prompts[0].name
        
        # 提供参数
        prompt_result = await session.get_prompt(
            prompt_name,
            arguments={
                "name": "Alice",
                "style": "friendly"
            }
        )
        
        # 3. 使用提示消息
        print(f"\n提示消息:")
        for message in prompt_result.messages:
            print(f"  角色: {message.role}")
            print(f"  内容: {message.content}")
```

### 2.5 分页处理

当 Server 返回大量数据时，使用分页可以提高性能。

```python
from mcp.types import PaginatedRequestParams


async def paginated_resources(session: ClientSession):
    """分页获取资源"""
    
    all_resources = []
    cursor = None
    page_num = 1
    
    while True:
        print(f"获取第 {page_num} 页...")
        
        # 请求一页数据
        result = await session.list_resources(
            params=PaginatedRequestParams(cursor=cursor)
        )
        
        # 收集资源
        all_resources.extend(result.resources)
        print(f"  本页资源数: {len(result.resources)}")
        
        # 检查是否有下一页
        if result.nextCursor:
            cursor = result.nextCursor
            page_num += 1
        else:
            break
    
    print(f"\n总共获取 {len(all_resources)} 个资源")
    return all_resources


async def paginated_tools(session: ClientSession):
    """分页获取工具"""
    
    all_tools = []
    cursor = None
    
    while True:
        result = await session.list_tools(
            params=PaginatedRequestParams(cursor=cursor)
        )
        
        all_tools.extend(result.tools)
        
        if result.nextCursor:
            cursor = result.nextCursor
        else:
            break
    
    return all_tools
```

### 2.6 采样请求处理

采样（Sampling）允许 Server 请求 Client 调用 LLM。

```python
from mcp import RequestContext
from mcp.types import CreateMessageRequestParams, CreateMessageResult, TextContent


async def handle_sampling(
    context: RequestContext[ClientSession, None],
    params: CreateMessageRequestParams
) -> CreateMessageResult:
    """处理 Server 的采样请求"""
    
    print(f"收到采样请求:")
    print(f"  模型: {params.modelPreferences}")
    print(f"  消息数: {len(params.messages)}")
    print(f"  最大 tokens: {params.maxTokens}")
    
    # 在这里调用你的 LLM
    # 例如：OpenAI、Anthropic、本地模型等
    
    # 模拟 LLM 响应
    response_text = "这是 LLM 的响应"
    
    return CreateMessageResult(
        role="assistant",
        content=TextContent(
            type="text",
            text=response_text
        ),
        model="gpt-4",
        stopReason="endTurn"
    )


async def client_with_sampling():
    """创建支持采样的 Client"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    async with stdio_client(server_params) as (read, write):
        # 传入采样回调
        async with ClientSession(
            read,
            write,
            sampling_callback=handle_sampling
        ) as session:
            await session.initialize()
            
            # 现在 Server 可以请求 LLM 调用
            result = await session.call_tool(
                "analyze_with_llm",
                arguments={"text": "分析这段文本"}
            )
            
            print(f"结果: {result.content[0].text}")
```

### 2.7 通知处理

Client 可以接收来自 Server 的各种通知。

```python
async def setup_notification_handlers(session: ClientSession):
    """设置通知处理器"""
    
    # 工具列表变更通知
    @session.notification_handler("notifications/tools/list_changed")
    async def on_tools_changed():
        print("🔔 工具列表已变更，重新获取...")
        tools = await session.list_tools()
        print(f"   新的工具数量: {len(tools.tools)}")
    
    # 资源列表变更通知
    @session.notification_handler("notifications/resources/list_changed")
    async def on_resources_changed():
        print("🔔 资源列表已变更，重新获取...")
        resources = await session.list_resources()
        print(f"   新的资源数量: {len(resources.resources)}")
    
    # 资源更新通知
    @session.notification_handler("notifications/resources/updated")
    async def on_resource_updated(uri: str):
        print(f"🔔 资源已更新: {uri}")
        # 重新读取资源
        content = await session.read_resource(uri)
        print(f"   新内容: {content.contents[0].text[:50]}...")
    
    # 日志通知
    @session.notification_handler("notifications/message")
    async def on_log_message(level: str, logger: str, data: str):
        print(f"📝 [{level}] {logger}: {data}")
```

### 2.8 会话状态管理

```python
from enum import Enum
from typing import Optional
import time


class SessionState(Enum):
    """会话状态"""
    DISCONNECTED = "disconnected"
    CONNECTING = "connecting"
    CONNECTED = "connected"
    INITIALIZING = "initializing"
    READY = "ready"
    ERROR = "error"


class ManagedSession:
    """带状态管理的会话包装器"""
    
    def __init__(self, server_params: StdioServerParameters):
        self.server_params = server_params
        self.state = SessionState.DISCONNECTED
        self.session: Optional[ClientSession] = None
        self.last_activity = time.time()
        self.error: Optional[str] = None
    
    async def connect(self):
        """连接到 Server"""
        try:
            self.state = SessionState.CONNECTING
            
            self._transport = stdio_client(self.server_params)
            read, write = await self._transport.__aenter__()
            
            self.state = SessionState.CONNECTED
            
            self._session_context = ClientSession(read, write)
            self.session = await self._session_context.__aenter__()
            
            self.state = SessionState.INITIALIZING
            await self.session.initialize()
            
            self.state = SessionState.READY
            self.last_activity = time.time()
            
            print(f"✅ 会话已就绪")
            
        except Exception as e:
            self.state = SessionState.ERROR
            self.error = str(e)
            print(f"❌ 连接失败: {e}")
            raise
    
    async def disconnect(self):
        """断开连接"""
        try:
            if self.session:
                await self._session_context.__aexit__(None, None, None)
            await self._transport.__aexit__(None, None, None)
            
            self.state = SessionState.DISCONNECTED
            self.session = None
            print(f"👋 会话已断开")
            
        except Exception as e:
            print(f"⚠️ 断开连接时出错: {e}")
    
    def is_ready(self) -> bool:
        """检查会话是否就绪"""
        return self.state == SessionState.READY
    
    def update_activity(self):
        """更新最后活动时间"""
        self.last_activity = time.time()
    
    def is_idle(self, timeout: float = 300) -> bool:
        """检查会话是否空闲"""
        return time.time() - self.last_activity > timeout


# 使用示例
async def managed_session_example():
    """使用托管会话"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    managed = ManagedSession(server_params)
    
    try:
        await managed.connect()
        
        if managed.is_ready():
            # 使用会话
            tools = await managed.session.list_tools()
            managed.update_activity()
            
            print(f"工具数量: {len(tools.tools)}")
        
    finally:
        await managed.disconnect()
```

---

## 3. 多 Server 管理

### 3.1 为什么需要多 Server？

在实际应用中，你可能需要同时连接多个 MCP Server 来访问不同的功能。

#### 多 Server 场景

```
应用场景示例：

┌─────────────────────────────────────┐
│      AI 助手应用                     │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │   Client    │
        └──────┬──────┘
               │
    ┌──────────┼──────────┐
    │          │          │
    ↓          ↓          ↓
┌────────┐ ┌────────┐ ┌────────┐
│文件系统│ │ 数据库 │ │  API   │
│ Server │ │ Server │ │ Server │
└────────┘ └────────┘ └────────┘

用户请求："分析 report.csv 中的数据，
          查询用户数据库，
          然后调用天气 API"

需要协调三个 Server 完成任务！
```

### 3.2 ClientSessionGroup 基础

Python SDK 提供 `ClientSessionGroup` 来管理多个 Server 连接。

#### 基础用法

```python
from mcp import ClientSessionGroup, StdioServerParameters


async def basic_multi_server():
    """基础多 Server 管理"""
    
    # 创建 Server 组
    async with ClientSessionGroup() as group:
        
        # 定义多个 Server
        file_server = StdioServerParameters(
            command="uv",
            args=["run", "python", "-m", "file_server"]
        )
        
        db_server = StdioServerParameters(
            command="uv",
            args=["run", "python", "-m", "database_server"]
        )
        
        api_server = StdioServerParameters(
            command="uv",
            args=["run", "python", "-m", "api_server"]
        )
        
        # 连接到所有 Server
        print("连接到 Server...")
        await group.connect_to_server(file_server)
        await group.connect_to_server(db_server)
        await group.connect_to_server(api_server)
        
        print(f"✅ 已连接 {len(group.sessions)} 个 Server")
        
        # 访问聚合的工具
        print(f"\n所有可用工具:")
        for tool_name in group.tools.keys():
            print(f"  - {tool_name}")
        
        # 访问聚合的资源
        print(f"\n所有可用资源:")
        for resource_uri in group.resources.keys():
            print(f"  - {resource_uri}")
        
        # 调用工具（自动路由到正确的 Server）
        result = await group.call_tool(
            "read_file",
            {"path": "/data/report.csv"}
        )
        print(f"\n工具调用结果: {result}")
```

### 3.3 组件名称冲突处理

当多个 Server 提供同名工具或资源时，需要避免冲突。

#### 使用命名钩子

```python
async def multi_server_with_naming():
    """使用命名钩子避免冲突"""
    
    def naming_hook(name: str, server_info) -> str:
        """为组件名称添加 Server 前缀"""
        server_name = server_info.name.lower().replace(" ", "_")
        return f"{server_name}_{name}"
    
    # 创建带命名钩子的 Server 组
    async with ClientSessionGroup(
        component_name_hook=naming_hook
    ) as group:
        
        # 连接 Server
        weather_server = StdioServerParameters(
            command="uv",
            args=["run", "python", "-m", "weather_server"]
        )
        
        news_server = StdioServerParameters(
            command="uv",
            args=["run", "python", "-m", "news_server"]
        )
        
        await group.connect_to_server(weather_server)
        await group.connect_to_server(news_server)
        
        # 现在工具名称会自动添加前缀
        print("可用工具:")
        for tool_name in group.tools.keys():
            print(f"  - {tool_name}")
        
        # 输出示例:
        # - weather_server_get_forecast
        # - weather_server_get_current
        # - news_server_get_headlines
        # - news_server_search_articles
        
        # 调用特定 Server 的工具
        weather = await group.call_tool(
            "weather_server_get_forecast",
            {"city": "Beijing"}
        )
        
        news = await group.call_tool(
            "news_server_get_headlines",
            {"category": "technology"}
        )
        
        print(f"\n天气: {weather}")
        print(f"新闻: {news}")
```

### 3.4 动态 Server 管理

在运行时添加和移除 Server。

```python
from typing import Dict, List


class DynamicServerManager:
    """动态 Server 管理器"""
    
    def __init__(self):
        self.group: Optional[ClientSessionGroup] = None
        self.server_configs: Dict[str, StdioServerParameters] = {}
    
    async def start(self):
        """启动管理器"""
        self.group = ClientSessionGroup()
        await self.group.__aenter__()
        print("✅ Server 管理器已启动")
    
    async def stop(self):
        """停止管理器"""
        if self.group:
            await self.group.__aexit__(None, None, None)
            self.group = None
        print("👋 Server 管理器已停止")
    
    async def add_server(
        self,
        name: str,
        command: str,
        args: List[str]
    ) -> bool:
        """添加新 Server"""
        try:
            server_params = StdioServerParameters(
                command=command,
                args=args
            )
            
            await self.group.connect_to_server(server_params)
            self.server_configs[name] = server_params
            
            print(f"✅ 已添加 Server: {name}")
            print(f"   当前 Server 数量: {len(self.group.sessions)}")
            
            return True
            
        except Exception as e:
            print(f"❌ 添加 Server 失败: {e}")
            return False
    
    async def remove_server(self, session_index: int) -> bool:
        """移除 Server"""
        try:
            if 0 <= session_index < len(self.group.sessions):
                session = self.group.sessions[session_index]
                await self.group.disconnect_from_server(session)
                
                print(f"✅ 已移除 Server (索引 {session_index})")
                print(f"   剩余 Server 数量: {len(self.group.sessions)}")
                
                return True
            else:
                print(f"❌ 无效的 Server 索引: {session_index}")
                return False
                
        except Exception as e:
            print(f"❌ 移除 Server 失败: {e}")
            return False
    
    def list_servers(self):
        """列出所有连接的 Server"""
        print(f"\n已连接的 Server ({len(self.group.sessions)}):")
        for i, session in enumerate(self.group.sessions):
            print(f"  [{i}] Session ID: {id(session)}")
    
    def list_tools(self):
        """列出所有可用工具"""
        print(f"\n可用工具 ({len(self.group.tools)}):")
        for tool_name in self.group.tools.keys():
            print(f"  - {tool_name}")
    
    async def call_tool(self, name: str, arguments: Dict[str, Any]):
        """调用工具"""
        try:
            result = await self.group.call_tool(name, arguments)
            return result
        except Exception as e:
            print(f"❌ 工具调用失败: {e}")
            return None


# 使用示例
async def dynamic_server_example():
    """动态 Server 管理示例"""
    
    manager = DynamicServerManager()
    
    try:
        # 启动管理器
        await manager.start()
        
        # 动态添加 Server
        await manager.add_server(
            "file_server",
            "uv",
            ["run", "python", "-m", "file_server"]
        )
        
        await manager.add_server(
            "db_server",
            "uv",
            ["run", "python", "-m", "database_server"]
        )
        
        # 列出 Server 和工具
        manager.list_servers()
        manager.list_tools()
        
        # 使用工具
        result = await manager.call_tool(
            "read_file",
            {"path": "/data/test.txt"}
        )
        print(f"\n工具结果: {result}")
        
        # 移除一个 Server
        await manager.remove_server(0)
        
        # 再次列出
        manager.list_servers()
        manager.list_tools()
        
    finally:
        # 停止管理器
        await manager.stop()
```

### 3.5 Server 健康检查

监控 Server 连接状态并自动重连。

```python
import asyncio
from datetime import datetime
from typing import Dict


class HealthMonitor:
    """Server 健康监控"""
    
    def __init__(self, group: ClientSessionGroup):
        self.group = group
        self.health_status: Dict[int, Dict] = {}
        self.monitoring = False
    
    async def start_monitoring(self, interval: float = 30.0):
        """开始监控"""
        self.monitoring = True
        print(f"🏥 开始健康监控 (间隔: {interval}秒)")
        
        while self.monitoring:
            await self.check_all_servers()
            await asyncio.sleep(interval)
    
    def stop_monitoring(self):
        """停止监控"""
        self.monitoring = False
        print("🏥 停止健康监控")
    
    async def check_all_servers(self):
        """检查所有 Server"""
        print(f"\n🔍 健康检查 - {datetime.now().strftime('%H:%M:%S')}")
        
        for i, session in enumerate(self.group.sessions):
            is_healthy = await self.check_server(session, i)
            
            status = "✅ 健康" if is_healthy else "❌ 异常"
            print(f"  Server {i}: {status}")
    
    async def check_server(
        self,
        session: ClientSession,
        index: int
    ) -> bool:
        """检查单个 Server"""
        try:
            # 尝试列出工具（简单的健康检查）
            result = await asyncio.wait_for(
                session.list_tools(),
                timeout=5.0
            )
            
            # 更新健康状态
            self.health_status[index] = {
                "healthy": True,
                "last_check": datetime.now(),
                "tools_count": len(result.tools)
            }
            
            return True
            
        except asyncio.TimeoutError:
            print(f"    ⚠️ Server {index} 响应超时")
            self.health_status[index] = {
                "healthy": False,
                "last_check": datetime.now(),
                "error": "timeout"
            }
            return False
            
        except Exception as e:
            print(f"    ⚠️ Server {index} 错误: {e}")
            self.health_status[index] = {
                "healthy": False,
                "last_check": datetime.now(),
                "error": str(e)
            }
            return False
    
    def get_health_report(self) -> Dict:
        """获取健康报告"""
        total = len(self.health_status)
        healthy = sum(1 for s in self.health_status.values() if s["healthy"])
        
        return {
            "total_servers": total,
            "healthy_servers": healthy,
            "unhealthy_servers": total - healthy,
            "health_rate": f"{healthy/total*100:.1f}%" if total > 0 else "0%",
            "details": self.health_status
        }


# 使用示例
async def health_monitoring_example():
    """健康监控示例"""
    
    async with ClientSessionGroup() as group:
        # 连接 Server
        await group.connect_to_server(
            StdioServerParameters(
                command="uv",
                args=["run", "python", "-m", "server1"]
            )
        )
        await group.connect_to_server(
            StdioServerParameters(
                command="uv",
                args=["run", "python", "-m", "server2"]
            )
        )
        
        # 创建健康监控器
        monitor = HealthMonitor(group)
        
        # 启动监控（后台任务）
        monitor_task = asyncio.create_task(
            monitor.start_monitoring(interval=10.0)
        )
        
        try:
            # 运行一段时间
            await asyncio.sleep(60)
            
            # 获取健康报告
            report = monitor.get_health_report()
            print(f"\n📊 健康报告:")
            print(f"  总 Server 数: {report['total_servers']}")
            print(f"  健康: {report['healthy_servers']}")
            print(f"  异常: {report['unhealthy_servers']}")
            print(f"  健康率: {report['health_rate']}")
            
        finally:
            # 停止监控
            monitor.stop_monitoring()
            await monitor_task
```

### 3.6 负载均衡

当有多个相同功能的 Server 时，可以实现负载均衡。

```python
from collections import defaultdict
from typing import List
import random


class LoadBalancer:
    """简单的负载均衡器"""
    
    def __init__(self, group: ClientSessionGroup):
        self.group = group
        self.call_counts: Dict[str, int] = defaultdict(int)
    
    def get_tool_servers(self, tool_name: str) -> List[ClientSession]:
        """获取提供指定工具的所有 Server"""
        servers = []
        
        for session in self.group.sessions:
            # 检查这个 session 是否有这个工具
            # （实际实现需要维护工具到 session 的映射）
            servers.append(session)
        
        return servers
    
    def select_server_round_robin(
        self,
        tool_name: str,
        servers: List[ClientSession]
    ) -> ClientSession:
        """轮询选择 Server"""
        if not servers:
            raise ValueError(f"没有 Server 提供工具: {tool_name}")
        
        # 轮询算法
        index = self.call_counts[tool_name] % len(servers)
        self.call_counts[tool_name] += 1
        
        return servers[index]
    
    def select_server_random(
        self,
        servers: List[ClientSession]
    ) -> ClientSession:
        """随机选择 Server"""
        if not servers:
            raise ValueError("没有可用的 Server")
        
        return random.choice(servers)
    
    def select_server_least_loaded(
        self,
        servers: List[ClientSession]
    ) -> ClientSession:
        """选择负载最低的 Server"""
        # 简化实现：选择调用次数最少的
        if not servers:
            raise ValueError("没有可用的 Server")
        
        # 获取每个 server 的调用次数
        server_loads = {
            id(server): sum(
                count for tool, count in self.call_counts.items()
                # 这里需要检查 tool 是否属于这个 server
            )
            for server in servers
        }
        
        # 选择负载最低的
        return min(servers, key=lambda s: server_loads.get(id(s), 0))
    
    async def call_tool_balanced(
        self,
        tool_name: str,
        arguments: Dict[str, Any],
        strategy: str = "round_robin"
    ):
        """使用负载均衡调用工具"""
        
        # 获取提供此工具的 Server
        servers = self.get_tool_servers(tool_name)
        
        # 根据策略选择 Server
        if strategy == "round_robin":
            server = self.select_server_round_robin(tool_name, servers)
        elif strategy == "random":
            server = self.select_server_random(servers)
        elif strategy == "least_loaded":
            server = self.select_server_least_loaded(servers)
        else:
            raise ValueError(f"未知的策略: {strategy}")
        
        print(f"📍 选择 Server: {id(server)} (策略: {strategy})")
        
        # 调用工具
        result = await server.call_tool(tool_name, arguments)
        
        return result


# 使用示例
async def load_balancing_example():
    """负载均衡示例"""
    
    async with ClientSessionGroup() as group:
        # 连接多个相同功能的 Server
        for i in range(3):
            await group.connect_to_server(
                StdioServerParameters(
                    command="uv",
                    args=["run", "python", "-m", f"worker_server_{i}"]
                )
            )
        
        # 创建负载均衡器
        balancer = LoadBalancer(group)
        
        # 使用不同策略调用工具
        print("使用轮询策略:")
        for i in range(5):
            result = await balancer.call_tool_balanced(
                "process_data",
                {"data": f"item_{i}"},
                strategy="round_robin"
            )
            print(f"  结果 {i}: {result}")
        
        print("\n使用随机策略:")
        for i in range(5):
            result = await balancer.call_tool_balanced(
                "process_data",
                {"data": f"item_{i}"},
                strategy="random"
            )
            print(f"  结果 {i}: {result}")
```

---
## 4. Host 应用集成

### 4.1 什么是 Host 应用？

Host 应用是集成 MCP Client 的应用程序，如 IDE、聊天机器人、桌面应用等。

#### Host 应用架构

```
┌──────────────────────────────────────┐
│         Host Application             │
│                                      │
│  ┌────────────────────────────────┐ │
│  │      User Interface            │ │
│  │  (Chat, IDE, Dashboard, etc.)  │ │
│  └──────────────┬─────────────────┘ │
│                 │                    │
│  ┌──────────────▼─────────────────┐ │
│  │    Application Logic           │ │
│  │  - Request routing             │ │
│  │  - Response processing         │ │
│  │  - State management            │ │
│  └──────────────┬─────────────────┘ │
│                 │                    │
│  ┌──────────────▼─────────────────┐ │
│  │    MCP Client Layer            │ │ ← 集成点
│  │  - Server connections          │ │
│  │  - Tool/Resource management    │ │
│  └────────────────────────────────┘ │
└──────────────────────────────────────┘
```

### 4.2 简单聊天机器人集成

创建一个简单的命令行聊天机器人，集成 MCP Client。

```python
# simple_chatbot.py

import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
from typing import List, Dict


class SimpleChatbot:
    """简单的聊天机器人"""
    
    def __init__(self):
        self.session: Optional[ClientSession] = None
        self.conversation_history: List[Dict[str, str]] = []
    
    async def connect_to_server(self, server_params: StdioServerParameters):
        """连接到 MCP Server"""
        print("🔌 连接到 MCP Server...")
        
        self._transport = stdio_client(server_params)
        read, write = await self._transport.__aenter__()
        
        self._session_context = ClientSession(read, write)
        self.session = await self._session_context.__aenter__()
        
        await self.session.initialize()
        print("✅ 连接成功！\n")
        
        # 显示可用工具
        tools = await self.session.list_tools()
        print(f"📦 可用工具 ({len(tools.tools)}):")
        for tool in tools.tools:
            print(f"  - {tool.name}: {tool.description}")
        print()
    
    async def disconnect(self):
        """断开连接"""
        if self.session:
            await self._session_context.__aexit__(None, None, None)
            await self._transport.__aexit__(None, None, None)
        print("\n👋 已断开连接")
    
    async def process_message(self, user_message: str) -> str:
        """处理用户消息"""
        # 添加到历史
        self.conversation_history.append({
            "role": "user",
            "content": user_message
        })
        
        # 简单的命令解析
        if user_message.startswith("/tool "):
            # 调用工具：/tool tool_name arg1=value1 arg2=value2
            return await self._handle_tool_command(user_message[6:])
        elif user_message.startswith("/list"):
            # 列出工具
            return await self._handle_list_command()
        elif user_message.startswith("/help"):
            return self._handle_help_command()
        else:
            # 普通对话
            response = f"收到消息: {user_message}\n使用 /help 查看可用命令"
            self.conversation_history.append({
                "role": "assistant",
                "content": response
            })
            return response
    
    async def _handle_tool_command(self, command: str) -> str:
        """处理工具调用命令"""
        parts = command.split()
        if not parts:
            return "❌ 请指定工具名称"
        
        tool_name = parts[0]
        
        # 解析参数
        arguments = {}
        for part in parts[1:]:
            if "=" in part:
                key, value = part.split("=", 1)
                arguments[key] = value
        
        try:
            result = await self.session.call_tool(tool_name, arguments)
            response = f"🔧 工具调用结果:\n{result.content[0].text}"
            
            self.conversation_history.append({
                "role": "assistant",
                "content": response
            })
            return response
            
        except Exception as e:
            return f"❌ 工具调用失败: {e}"
    
    async def _handle_list_command(self) -> str:
        """处理列表命令"""
        tools = await self.session.list_tools()
        
        response = f"📦 可用工具 ({len(tools.tools)}):\n"
        for tool in tools.tools:
            response += f"  - {tool.name}: {tool.description}\n"
        
        return response
    
    def _handle_help_command(self) -> str:
        """处理帮助命令"""
        return """
📖 可用命令:
  /tool <name> [args]  - 调用工具
  /list                - 列出所有工具
  /help                - 显示帮助
  /quit                - 退出

示例:
  /tool add a=5 b=3
  /tool search query=Python
"""
    
    async def run(self):
        """运行聊天机器人"""
        print("🤖 简单聊天机器人")
        print("=" * 50)
        
        while True:
            try:
                user_input = input("\n你: ").strip()
                
                if not user_input:
                    continue
                
                if user_input.lower() in ["/quit", "/exit", "quit", "exit"]:
                    break
                
                response = await self.process_message(user_input)
                print(f"\n机器人: {response}")
                
            except KeyboardInterrupt:
                break
            except Exception as e:
                print(f"\n❌ 错误: {e}")


async def main():
    """主函数"""
    # 配置 Server
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_mcp_server"]
    )
    
    # 创建并运行聊天机器人
    bot = SimpleChatbot()
    
    try:
        await bot.connect_to_server(server_params)
        await bot.run()
    finally:
        await bot.disconnect()


if __name__ == "__main__":
    asyncio.run(main())
```

### 4.3 Web 应用集成

将 MCP Client 集成到 Web 应用中。

```python
# web_app.py 

from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.responses import HTMLResponse
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
import asyncio
import json


app = FastAPI()

# 全局 MCP Client
mcp_client: Optional[ClientSession] = None


@app.on_event("startup")
async def startup_event():
    """应用启动时连接到 MCP Server"""
    global mcp_client
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_mcp_server"]
    )
    
    transport = stdio_client(server_params)
    read, write = await transport.__aenter__()
    
    session_context = ClientSession(read, write)
    mcp_client = await session_context.__aenter__()
    
    await mcp_client.initialize()
    print("✅ MCP Client 已连接")


@app.on_event("shutdown")
async def shutdown_event():
    """应用关闭时断开连接"""
    global mcp_client
    if mcp_client:
        # 清理连接
        print("👋 断开 MCP Client")


@app.get("/")
async def get_index():
    """主页"""
    html_content = """
    <!DOCTYPE html>
    <html>
    <head>
        <title>MCP Web App</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            #messages { height: 400px; overflow-y: scroll; border: 1px solid #ccc; padding: 10px; }
            .message { margin: 10px 0; }
            .user { color: blue; }
            .assistant { color: green; }
        </style>
    </head>
    <body>
        <h1>MCP Web Chat</h1>
        <div id="messages"></div>
        <input type="text" id="input" placeholder="输入消息..." style="width: 80%;">
        <button onclick="sendMessage()">发送</button>
        
        <script>
            const ws = new WebSocket("ws://localhost:8000/ws");
            const messages = document.getElementById("messages");
            const input = document.getElementById("input");
            
            ws.onmessage = function(event) {
                const data = JSON.parse(event.data);
                addMessage(data.role, data.content);
            };
            
            function addMessage(role, content) {
                const div = document.createElement("div");
                div.className = "message " + role;
                div.textContent = role + ": " + content;
                messages.appendChild(div);
                messages.scrollTop = messages.scrollHeight;
            }
            
            function sendMessage() {
                const message = input.value;
                if (message) {
                    ws.send(JSON.stringify({message: message}));
                    addMessage("user", message);
                    input.value = "";
                }
            }
            
            input.addEventListener("keypress", function(event) {
                if (event.key === "Enter") {
                    sendMessage();
                }
            });
        </script>
    </body>
    </html>
    """
    return HTMLResponse(content=html_content)


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    """WebSocket 端点"""
    await websocket.accept()
    
    try:
        while True:
            # 接收消息
            data = await websocket.receive_text()
            message_data = json.loads(data)
            user_message = message_data["message"]
            
            # 处理消息（简单示例：调用工具）
            if user_message.startswith("/tool "):
                tool_name = user_message[6:].strip()
                
                try:
                    result = await mcp_client.call_tool(
                        tool_name,
                        arguments={}
                    )
                    response = result.content[0].text
                except Exception as e:
                    response = f"错误: {e}"
            else:
                # 列出工具
                tools = await mcp_client.list_tools()
                response = f"可用工具: {', '.join(t.name for t in tools.tools)}"
            
            # 发送响应
            await websocket.send_text(json.dumps({
                "role": "assistant",
                "content": response
            }))
            
    except WebSocketDisconnect:
        print("客户端断开连接")


@app.get("/api/tools")
async def get_tools():
    """获取工具列表 API"""
    tools = await mcp_client.list_tools()
    return {
        "tools": [
            {
                "name": tool.name,
                "description": tool.description,
                "inputSchema": tool.inputSchema
            }
            for tool in tools.tools
        ]
    }


@app.post("/api/tool/{tool_name}")
async def call_tool(tool_name: str, arguments: dict):
    """调用工具 API"""
    try:
        result = await mcp_client.call_tool(tool_name, arguments)
        return {
            "success": True,
            "result": result.content[0].text
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 5. 高级集成模式

### 5.1 OAuth 认证集成

为 Client 添加 OAuth 认证支持。

```python
from mcp.client.auth import OAuthClientProvider, TokenStorage
from mcp.shared.auth import OAuthClientMetadata, OAuthToken
from mcp.client.streamable_http import streamablehttp_client
from pydantic import AnyUrl


class SecureTokenStorage(TokenStorage):
    """安全的 Token 存储"""
    
    def __init__(self, storage_file: str = ".mcp_tokens.json"):
        self.storage_file = storage_file
        self.tokens: Optional[OAuthToken] = None
        self._load_tokens()
    
    def _load_tokens(self):
        """从文件加载 tokens"""
        try:
            with open(self.storage_file, 'r') as f:
                data = json.load(f)
                self.tokens = OAuthToken(**data)
        except FileNotFoundError:
            pass
    
    def _save_tokens(self):
        """保存 tokens 到文件"""
        if self.tokens:
            with open(self.storage_file, 'w') as f:
                json.dump(self.tokens.dict(), f)
    
    async def get_tokens(self) -> Optional[OAuthToken]:
        """获取 tokens"""
        return self.tokens
    
    async def set_tokens(self, tokens: OAuthToken) -> None:
        """设置 tokens"""
        self.tokens = tokens
        self._save_tokens()


async def oauth_client_example():
    """OAuth 认证 Client 示例"""
    
    # 配置 OAuth
    oauth_provider = OAuthClientProvider(
        server_url="https://mcp-server.example.com",
        client_metadata=OAuthClientMetadata(
            client_name="My MCP Client",
            redirect_uris=[AnyUrl("http://localhost:3000/callback")],
            grant_types=["authorization_code", "refresh_token"],
            scope="read write"
        ),
        storage=SecureTokenStorage(),
        redirect_handler=lambda url: print(f"请访问: {url}"),
        callback_handler=lambda: input("粘贴回调 URL: ")
    )
    
    # 使用 OAuth 连接
    async with streamablehttp_client(
        "https://mcp-server.example.com/mcp",
        auth=oauth_provider
    ) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # 所有请求都会包含认证信息
            tools = await session.list_tools()
            print(f"可用工具: {[t.name for t in tools.tools]}")
```

### 5.2 缓存层实现

为 Client 添加智能缓存。

```python
from functools import lru_cache
from datetime import datetime, timedelta
from typing import Any, Optional
import hashlib
import json


class MCPClientCache:
    """MCP Client 缓存层"""
    
    def __init__(self, ttl_seconds: int = 300):
        self.ttl_seconds = ttl_seconds
        self.cache: Dict[str, Dict[str, Any]] = {}
    
    def _make_key(self, method: str, **kwargs) -> str:
        """生成缓存键"""
        data = json.dumps({"method": method, **kwargs}, sort_keys=True)
        return hashlib.md5(data.encode()).hexdigest()
    
    def get(self, method: str, **kwargs) -> Optional[Any]:
        """获取缓存"""
        key = self._make_key(method, **kwargs)
        
        if key in self.cache:
            entry = self.cache[key]
            
            # 检查是否过期
            if datetime.now() < entry["expires_at"]:
                print(f"💾 缓存命中: {method}")
                return entry["data"]
            else:
                # 过期，删除
                del self.cache[key]
        
        return None
    
    def set(self, method: str, data: Any, **kwargs):
        """设置缓存"""
        key = self._make_key(method, **kwargs)
        
        self.cache[key] = {
            "data": data,
            "expires_at": datetime.now() + timedelta(seconds=self.ttl_seconds),
            "created_at": datetime.now()
        }
        
        print(f"💾 缓存保存: {method}")
    
    def invalidate(self, method: Optional[str] = None):
        """使缓存失效"""
        if method is None:
            # 清空所有缓存
            self.cache.clear()
            print("💾 清空所有缓存")
        else:
            # 清空特定方法的缓存
            keys_to_delete = [
                k for k, v in self.cache.items()
                if method in k
            ]
            for key in keys_to_delete:
                del self.cache[key]
            print(f"💾 清空缓存: {method}")


class CachedMCPClient:
    """带缓存的 MCP Client"""
    
    def __init__(self, session: ClientSession, cache_ttl: int = 300):
        self.session = session
        self.cache = MCPClientCache(ttl_seconds=cache_ttl)
    
    async def list_tools(self, use_cache: bool = True):
        """列出工具（带缓存）"""
        if use_cache:
            cached = self.cache.get("list_tools")
            if cached:
                return cached
        
        result = await self.session.list_tools()
        
        if use_cache:
            self.cache.set("list_tools", result)
        
        return result
    
    async def list_resources(self, use_cache: bool = True):
        """列出资源（带缓存）"""
        if use_cache:
            cached = self.cache.get("list_resources")
            if cached:
                return cached
        
        result = await self.session.list_resources()
        
        if use_cache:
            self.cache.set("list_resources", result)
        
        return result
    
    async def call_tool(
        self,
        name: str,
        arguments: Dict[str, Any],
        use_cache: bool = False  # 工具调用默认不缓存
    ):
        """调用工具（可选缓存）"""
        if use_cache:
            cached = self.cache.get("call_tool", name=name, arguments=arguments)
            if cached:
                return cached
        
        result = await self.session.call_tool(name, arguments)
        
        if use_cache:
            self.cache.set("call_tool", result, name=name, arguments=arguments)
        
        return result
    
    def invalidate_cache(self, method: Optional[str] = None):
        """使缓存失效"""
        self.cache.invalidate(method)


# 使用示例
async def cached_client_example():
    """使用缓存 Client"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # 创建缓存 Client
            cached_client = CachedMCPClient(session, cache_ttl=60)
            
            # 第一次调用（从 Server 获取）
            print("第一次调用:")
            tools1 = await cached_client.list_tools()
            print(f"工具数: {len(tools1.tools)}")
            
            # 第二次调用（从缓存获取）
            print("\n第二次调用:")
            tools2 = await cached_client.list_tools()
            print(f"工具数: {len(tools2.tools)}")
            
            # 使缓存失效
            cached_client.invalidate_cache("list_tools")
            
            # 第三次调用（重新从 Server 获取）
            print("\n第三次调用（缓存已失效）:")
            tools3 = await cached_client.list_tools()
            print(f"工具数: {len(tools3.tools)}")
```

---

## 6. 最佳实践与总结

### 6.1 设计最佳实践

#### 1. 连接管理

```python
# ✅ 好的做法：使用上下文管理器
async with stdio_client(server_params) as (read, write):
    async with ClientSession(read, write) as session:
        # 使用 session
        pass
# 自动清理资源

# ❌ 避免：手动管理连接
# 容易忘记清理，导致资源泄漏
```

#### 2. 错误处理

```python
# ✅ 好的做法：细粒度错误处理
try:
    result = await session.call_tool(name, arguments)
except ConnectionError as e:
    logger.error(f"连接错误: {e}")
    # 尝试重连
except TimeoutError as e:
    logger.error(f"超时: {e}")
    # 返回默认值或重试
except Exception as e:
    logger.error(f"未知错误: {e}")
    # 通用处理

# ❌ 避免：捕获所有异常而不处理
try:
    result = await session.call_tool(name, arguments)
except:
    pass  # 忽略错误
```

#### 3. 资源清理

```python
# ✅ 好的做法：确保清理
try:
    await client.connect()
    # 使用 client
finally:
    await client.disconnect()

# ❌ 避免：忘记清理
await client.connect()
# 使用 client
# 忘记断开连接
```

### 6.2 性能优化建议

1. **使用连接池**：复用连接，减少建立连接的开销
2. **批量操作**：合并多个请求，减少网络往返
3. **异步并发**：使用 `asyncio.gather` 并发执行独立操作
4. **智能缓存**：缓存不常变化的数据（如工具列表）
5. **超时设置**：为所有操作设置合理的超时时间

### 6.3 安全建议

1. **认证授权**：使用 OAuth 等标准认证机制
2. **输入验证**：验证所有用户输入和 Server 响应
3. **密钥管理**：安全存储认证令牌和密钥
4. **传输加密**：使用 HTTPS/TLS 加密通信
5. **权限控制**：限制 Client 可访问的资源和操作

### 6.4 阶段 4 完成检查清单

- [ ] 能够创建基本的 MCP Client
- [ ] 理解不同传输方式的使用场景
- [ ] 掌握会话初始化和管理
- [ ] 能够处理工具调用、资源读取和提示
- [ ] 理解多 Server 管理和协调
- [ ] 能够集成 MCP Client 到 Host 应用
- [ ] 掌握高级模式（OAuth、缓存等）
- [ ] 了解最佳实践和性能优化

### 6.5 下一步

恭喜完成阶段 4！现在你已经：

✅ 掌握了 MCP Client 开发  
✅ 能够管理多个 Server 连接  
✅ 理解了 Host 应用集成  
✅ 学会了高级集成模式  
✅ 了解了最佳实践  

**接下来**，你可以：

👉 **实战项目**
- 开发完整的 MCP 应用
- 集成到现有系统
- 优化性能和用户体验

👉 **深入学习**
- 研究 MCP 协议细节
- 贡献开源项目
- 分享经验和最佳实践

---

## 📚 附录

### A. 常用代码模板

#### 基础 Client 模板

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client


async def main():
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_server"]
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            
            # 使用 session
            tools = await session.list_tools()
            print(f"工具: {[t.name for t in tools.tools]}")


if __name__ == "__main__":
    asyncio.run(main())
```

### B. 推荐资源

**官方文档**：
- MCP 规范：https://modelcontextprotocol.io/specification
- Python SDK：https://github.com/modelcontextprotocol/python-sdk
- TypeScript SDK：https://github.com/modelcontextprotocol/typescript-sdk

**示例项目**：
- Claude Desktop：MCP 集成示例
- 官方示例：https://github.com/modelcontextprotocol/servers

**社区资源**：
- MCP Discord 社区
- GitHub Discussions
- Stack Overflow (标签: model-context-protocol)

---

**祝你开发愉快！🚀**

> 本文档最后更新：2025年11月
> 
> 如有任何问题或建议，欢迎反馈！
