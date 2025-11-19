# MCP 学习计划 - 阶段 3：MCP Server 开发实战详解

> **本文档目标**：掌握 MCP Server 的开发技能，能够从零开始创建功能完整的 MCP Server。

---

## 📚 目录

1. [Server 开发入门](#1-server-开发入门)
2. [实现工具（Tools）](#2-实现工具tools)
3. [实现资源（Resources）](#3-实现资源resources)
4. [实现提示（Prompts）](#4-实现提示prompts)
5. [生命周期与上下文管理](#5-生命周期与上下文管理)
6. [测试与调试](#6-测试与调试)
7. [最佳实践与项目实战](#7-最佳实践与项目实战)

---

## 1. Server 开发入门

### 1.1 为什么要开发 MCP Server？

在阶段 1 和 2 中，我们学习了 MCP 的架构和协议。现在让我们动手创建自己的 MCP Server。

#### 开发 MCP Server 的价值

```
场景 1：企业内部工具集成
─────────────────────────────
你的公司有多个内部系统：
- 客户管理系统（CRM）
- 项目管理系统
- 知识库系统

通过 MCP Server，可以：
✅ 统一暴露这些系统的功能
✅ 让 AI 助手能够访问和操作
✅ 无需修改现有系统

场景 2：个人工具增强
─────────────────────────────
你想让 AI 助手能够：
- 搜索本地文件
- 查询天气信息
- 管理待办事项

通过 MCP Server，可以：
✅ 快速实现这些功能
✅ 与任何支持 MCP 的 AI 应用集成
✅ 功能可复用和分享

场景 3：开源贡献
─────────────────────────────
你可以：
✅ 为社区贡献新的 MCP Server
✅ 让更多人受益
✅ 建立个人技术影响力
```

### 1.2 选择开发语言

MCP 官方提供了两个 SDK：

#### Python SDK（推荐初学者）

**优势**：
- ✅ 简单易学
- ✅ FastMCP 框架开发效率高
- ✅ 丰富的第三方库
- ✅ 适合 AI/数据处理场景

**适合场景**：
- 数据分析工具
- API 集成
- 文件处理
- 机器学习模型调用

#### TypeScript SDK（推荐 Web 开发者）

**优势**：
- ✅ 类型安全
- ✅ Node.js 生态丰富
- ✅ 适合 Web 服务集成
- ✅ 性能优秀

**适合场景**：
- Web API 集成
- 数据库操作
- 实时服务
- 浏览器自动化

### 1.3 环境准备

#### Python 环境搭建

**1. 安装 Python 3.10+**

```bash
# 检查 Python 版本
python --version  # 应该 >= 3.10

# 如果版本过低，请安装最新版本
# Windows: 从 python.org 下载安装
# macOS: brew install python@3.11
# Linux: sudo apt install python3.11
```

**2. 安装 uv（推荐的包管理器）**

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# 验证安装
uv --version
```

**3. 创建项目**

```bash
# 创建项目目录
mkdir my-mcp-server
cd my-mcp-server

# 初始化 Python 项目
uv init

# 安装 MCP SDK
uv add mcp

# 安装开发依赖
uv add --dev pytest pytest-asyncio
```

**4. 项目结构**

```
my-mcp-server/
├── pyproject.toml          # 项目配置
├── README.md               # 项目说明
├── src/
│   └── my_mcp_server/
│       ├── __init__.py
│       └── server.py       # Server 主文件
└── tests/
    └── test_server.py      # 测试文件
```

#### TypeScript 环境搭建（可选）

**1. 安装 Node.js 18+**

```bash
# 检查 Node.js 版本
node --version  # 应该 >= 18

# 安装 pnpm
npm install -g pnpm

# 验证安装
pnpm --version
```

**2. 创建项目**

```bash
# 创建项目目录
mkdir my-mcp-server
cd my-mcp-server

# 初始化项目
pnpm init

# 安装 MCP SDK
pnpm add @modelcontextprotocol/sdk

# 安装 TypeScript
pnpm add -D typescript @types/node

# 初始化 TypeScript 配置
pnpm tsc --init
```

**3. 项目结构**

```
my-mcp-server/
├── package.json
├── tsconfig.json
├── src/
│   └── server.ts           # Server 主文件
└── dist/                   # 编译输出
```

### 1.4 第一个 MCP Server（Hello World）

让我们创建一个最简单的 MCP Server。

#### Python 版本（使用 FastMCP）

```python
# src/my_mcp_server/server.py

from mcp.server.fastmcp import FastMCP

# 创建 MCP Server 实例
mcp = FastMCP("Hello World Server")

@mcp.tool()
def hello(name: str = "World") -> str:
    """
    向指定的人问好
    
    Args:
        name: 要问好的人的名字
    
    Returns:
        问候语
    """
    return f"Hello, {name}!"

@mcp.tool()
def add(a: int, b: int) -> int:
    """
    计算两个数的和
    
    Args:
        a: 第一个数
        b: 第二个数
    
    Returns:
        两数之和
    """
    return a + b

def main():
    """启动 Server"""
    # 使用 stdio 传输方式
    mcp.run()

if __name__ == "__main__":
    main()
```

**运行 Server**：

```bash
# 直接运行
uv run python src/my_mcp_server/server.py

# 或者安装后运行
uv pip install -e .
my-mcp-server
```

#### TypeScript 版本

```typescript
// src/server.ts

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// 创建 Server 实例
const server = new McpServer({
  name: "hello-world-server",
  version: "1.0.0"
});

// 注册 hello 工具
server.registerTool(
  "hello",
  {
    title: "Hello Tool",
    description: "向指定的人问好",
    inputSchema: {
      name: z.string().default("World")
    }
  },
  async ({ name }) => ({
    content: [{
      type: "text",
      text: `Hello, ${name}!`
    }]
  })
);

// 注册 add 工具
server.registerTool(
  "add",
  {
    title: "Addition Tool",
    description: "计算两个数的和",
    inputSchema: {
      a: z.number(),
      b: z.number()
    }
  },
  async ({ a, b }) => ({
    content: [{
      type: "text",
      text: String(a + b)
    }]
  })
);

// 启动 Server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Hello World Server running on stdio");
}

main().catch(console.error);
```

**运行 Server**：

```bash
# 编译
pnpm tsc

# 运行
node dist/server.js
```

### 1.5 测试 Server

#### 使用 MCP Inspector 测试

MCP Inspector 是官方提供的调试工具。

**1. 安装 MCP Inspector**

```bash
npx @modelcontextprotocol/inspector
```

**2. 配置 Server**

在 Inspector 中输入：

```json
{
  "command": "uv",
  "args": ["run", "python", "src/my_mcp_server/server.py"]
}
```

**3. 测试工具**

- 点击 "Connect" 连接到 Server
- 查看 "Tools" 标签，应该看到 `hello` 和 `add` 工具
- 点击工具，输入参数测试

#### 使用 Python 客户端测试

```python
# tests/test_server.py

import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def test_hello_world_server():
    """测试 Hello World Server"""
    
    # 配置 Server
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "src/my_mcp_server/server.py"]
    )
    
    # 连接到 Server
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            
            # 初始化
            await session.initialize()
            
            # 列出工具
            tools = await session.list_tools()
            print(f"Available tools: {[t.name for t in tools.tools]}")
            
            # 测试 hello 工具
            result = await session.call_tool("hello", {"name": "Alice"})
            print(f"hello result: {result.content[0].text}")
            
            # 测试 add 工具
            result = await session.call_tool("add", {"a": 5, "b": 3})
            print(f"add result: {result.content[0].text}")

if __name__ == "__main__":
    asyncio.run(test_hello_world_server())
```

**运行测试**：

```bash
uv run python tests/test_server.py
```

**预期输出**：

```
Available tools: ['hello', 'add']
hello result: Hello, Alice!
add result: 8
```

### 1.6 Server 配置选项

#### FastMCP 配置（Python）

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP(
    name="My Server",                    # Server 名称
    version="1.0.0",                     # 版本号
    description="My awesome MCP server", # 描述
    
    # 传输方式配置
    transport="stdio",  # 或 "sse"
    
    # SSE 配置（如果使用 SSE）
    host="0.0.0.0",
    port=8000,
    
    # 日志配置
    log_level="INFO",  # DEBUG, INFO, WARNING, ERROR
    
    # 其他选项
    stateless_http=False,  # 是否无状态（SSE）
)
```

#### McpServer 配置（TypeScript）

```typescript
const server = new McpServer({
  name: "my-server",
  version: "1.0.0",
  
  // 能力声明
  capabilities: {
    tools: {},
    resources: {},
    prompts: {}
  }
});
```

### 1.7 常见问题排查

#### 问题 1：Server 无法启动

**症状**：运行 Server 时没有任何输出

**解决方案**：
```python
# 添加日志输出
import logging
logging.basicConfig(level=logging.DEBUG)

# 或者在 FastMCP 中启用调试
mcp = FastMCP("My Server", log_level="DEBUG")
```

#### 问题 2：Client 连接不上

**症状**：Client 报错 "Connection refused"

**检查清单**：
- [ ] Server 是否正在运行
- [ ] 传输方式是否匹配（stdio/SSE）
- [ ] 端口是否被占用（SSE）
- [ ] 命令路径是否正确

#### 问题 3：工具调用失败

**症状**：工具列表能看到，但调用时报错

**常见原因**：
1. 参数类型不匹配
2. 函数抛出异常
3. 返回值格式不正确

**调试方法**：
```python
@mcp.tool()
def my_tool(param: str) -> str:
    try:
        # 添加日志
        print(f"Received param: {param}")
        
        result = process(param)
        
        print(f"Returning: {result}")
        return result
        
    except Exception as e:
        print(f"Error: {e}")
        raise
```

### 1.8 开发工作流

推荐的开发流程：

```
1. 设计阶段
   ↓
   确定要实现的功能
   设计工具的输入输出
   
2. 实现阶段
   ↓
   编写工具函数
   添加类型注解和文档
   
3. 测试阶段
   ↓
   使用 MCP Inspector 手动测试
   编写自动化测试
   
4. 调试阶段
   ↓
   查看日志
   使用断点调试
   
5. 优化阶段
   ↓
   性能优化
   错误处理完善
   
6. 部署阶段
   ↓
   打包发布
   编写使用文档
```

---

## 2. 实现工具（Tools）

### 2.1 工具设计原则

在实现工具之前，让我们先了解好的工具设计原则。

#### 单一职责原则

```python
# ❌ 不好的设计：一个工具做太多事
@mcp.tool()
def manage_user(action: str, user_id: str, data: dict) -> str:
    """管理用户（创建、更新、删除）"""
    if action == "create":
        return create_user(data)
    elif action == "update":
        return update_user(user_id, data)
    elif action == "delete":
        return delete_user(user_id)

# ✅ 好的设计：每个工具职责单一
@mcp.tool()
def create_user(name: str, email: str) -> str:
    """创建新用户"""
    return f"Created user: {name}"

@mcp.tool()
def update_user(user_id: str, name: str = None, email: str = None) -> str:
    """更新用户信息"""
    return f"Updated user {user_id}"

@mcp.tool()
def delete_user(user_id: str) -> str:
    """删除用户"""
    return f"Deleted user {user_id}"
```

#### 明确的输入输出

```python
# ❌ 不好的设计：参数不明确
@mcp.tool()
def search(query: str) -> str:
    """搜索"""  # 搜索什么？返回什么？
    pass

# ✅ 好的设计：清晰的参数和返回值
@mcp.tool()
def search_products(
    keyword: str,
    category: str = "all",
    max_results: int = 10
) -> str:
    """
    搜索商品
    
    Args:
        keyword: 搜索关键词
        category: 商品类别（all, electronics, books, clothing）
        max_results: 最多返回结果数（1-100）
    
    Returns:
        JSON 格式的商品列表
    """
    pass
```

### 2.2 基础工具实现

#### 简单的同步工具

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Basic Tools Server")

@mcp.tool()
def calculate_bmi(weight_kg: float, height_m: float) -> str:
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
    
    return f"BMI: {bmi:.2f} ({category})"

@mcp.tool()
def format_json(json_string: str, indent: int = 2) -> str:
    """
    格式化 JSON 字符串
    
    Args:
        json_string: 要格式化的 JSON 字符串
        indent: 缩进空格数
    
    Returns:
        格式化后的 JSON 字符串
    """
    import json
    
    try:
        data = json.loads(json_string)
        return json.dumps(data, indent=indent, ensure_ascii=False)
    except json.JSONDecodeError as e:
        return f"JSON 解析错误: {e}"
```

#### 异步工具

```python
import asyncio
import httpx

@mcp.tool()
async def fetch_url(url: str) -> str:
    """
    获取 URL 的内容
    
    Args:
        url: 要获取的 URL
    
    Returns:
        URL 的文本内容
    """
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, timeout=10.0)
            response.raise_for_status()
            return response.text
        except httpx.HTTPError as e:
            return f"HTTP 错误: {e}"

@mcp.tool()
async def search_github(query: str, max_results: int = 5) -> str:
    """
    搜索 GitHub 仓库
    
    Args:
        query: 搜索关键词
        max_results: 最多返回结果数
    
    Returns:
        JSON 格式的仓库列表
    """
    import json
    
    async with httpx.AsyncClient() as client:
        response = await client.get(
            "https://api.github.com/search/repositories",
            params={"q": query, "per_page": max_results},
            headers={"Accept": "application/vnd.github.v3+json"}
        )
        
        data = response.json()
        repos = []
        
        for item in data.get("items", []):
            repos.append({
                "name": item["full_name"],
                "description": item["description"],
                "stars": item["stargazers_count"],
                "url": item["html_url"]
            })
        
        return json.dumps(repos, indent=2, ensure_ascii=False)
```

### 2.3 带参数验证的工具

#### 使用 Pydantic 进行验证

```python
from pydantic import BaseModel, Field, validator
from typing import Literal

class SearchParams(BaseModel):
    """搜索参数"""
    keyword: str = Field(..., min_length=1, max_length=100, description="搜索关键词")
    category: Literal["all", "electronics", "books", "clothing"] = Field(
        default="all",
        description="商品类别"
    )
    min_price: float = Field(default=0, ge=0, description="最低价格")
    max_price: float = Field(default=10000, ge=0, description="最高价格")
    
    @validator("max_price")
    def validate_price_range(cls, v, values):
        """验证价格范围"""
        if "min_price" in values and v < values["min_price"]:
            raise ValueError("最高价格不能小于最低价格")
        return v

@mcp.tool()
def search_products_validated(params: SearchParams) -> str:
    """
    搜索商品（带参数验证）
    
    Args:
        params: 搜索参数
    
    Returns:
        商品列表
    """
    # 参数已经通过 Pydantic 验证
    return f"搜索 {params.keyword} in {params.category}, 价格范围: {params.min_price}-{params.max_price}"
```

#### 自定义参数验证

```python
@mcp.tool()
def send_email(
    to: str,
    subject: str,
    body: str,
    cc: list[str] = None
) -> str:
    """
    发送邮件
    
    Args:
        to: 收件人邮箱
        subject: 邮件主题
        body: 邮件正文
        cc: 抄送列表
    
    Returns:
        发送结果
    """
    import re
    
    # 验证邮箱格式
    email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    
    if not re.match(email_pattern, to):
        return f"错误：无效的收件人邮箱 '{to}'"
    
    if cc:
        for email in cc:
            if not re.match(email_pattern, email):
                return f"错误：无效的抄送邮箱 '{email}'"
    
    # 验证主题长度
    if len(subject) > 200:
        return "错误：邮件主题过长（最多 200 字符）"
    
    # 发送邮件（示例）
    return f"邮件已发送到 {to}"
```

### 2.4 错误处理

#### 优雅的错误处理

```python
from typing import Optional

class ToolError(Exception):
    """工具执行错误"""
    pass

@mcp.tool()
async def get_weather(city: str) -> str:
    """
    获取城市天气
    
    Args:
        city: 城市名称
    
    Returns:
        天气信息
    """
    try:
        # 模拟 API 调用
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"https://api.weather.com/v1/weather",
                params={"city": city},
                timeout=5.0
            )
            
            if response.status_code == 404:
                return f"错误：找不到城市 '{city}'"
            
            response.raise_for_status()
            data = response.json()
            
            return f"{city} 天气：{data['condition']}，温度 {data['temperature']}°C"
            
    except httpx.TimeoutException:
        return f"错误：获取 {city} 天气超时，请稍后重试"
    
    except httpx.HTTPError as e:
        return f"错误：天气服务暂时不可用 ({e})"
    
    except Exception as e:
        # 记录未预期的错误
        import logging
        logging.error(f"Unexpected error in get_weather: {e}")
        return "错误：服务器内部错误"
```

#### 返回结构化错误

```python
import json
from typing import Union

@mcp.tool()
def divide(a: float, b: float) -> str:
    """
    除法运算
    
    Args:
        a: 被除数
        b: 除数
    
    Returns:
        计算结果或错误信息（JSON 格式）
    """
    if b == 0:
        return json.dumps({
            "success": False,
            "error": {
                "code": "DIVISION_BY_ZERO",
                "message": "除数不能为零"
            }
        })
    
    result = a / b
    return json.dumps({
        "success": True,
        "result": result
    })
```

### 2.5 返回多种内容类型

#### 返回文本

```python
@mcp.tool()
def generate_report(data: dict) -> str:
    """生成文本报告"""
    return f"报告内容：{data}"
```

#### 返回 JSON

```python
import json

@mcp.tool()
def get_user_info(user_id: str) -> str:
    """
    获取用户信息
    
    Returns:
        JSON 格式的用户信息
    """
    user = {
        "id": user_id,
        "name": "张三",
        "email": "zhangsan@example.com",
        "created_at": "2024-01-01"
    }
    
    return json.dumps(user, ensure_ascii=False, indent=2)
```

#### 返回图片（Base64）

```python
import base64
from io import BytesIO
from PIL import Image

@mcp.tool()
def generate_qrcode(text: str) -> str:
    """
    生成二维码
    
    Args:
        text: 要编码的文本
    
    Returns:
        Base64 编码的二维码图片
    """
    import qrcode
    
    # 生成二维码
    qr = qrcode.QRCode(version=1, box_size=10, border=5)
    qr.add_data(text)
    qr.make(fit=True)
    
    img = qr.make_image(fill_color="black", back_color="white")
    
    # 转换为 Base64
    buffer = BytesIO()
    img.save(buffer, format="PNG")
    img_base64 = base64.b64encode(buffer.getvalue()).decode()
    
    return json.dumps({
        "type": "image",
        "format": "png",
        "data": img_base64
    })
```

### 2.6 工具组合与链式调用

#### 设计可组合的工具

```python
@mcp.tool()
def fetch_webpage(url: str) -> str:
    """获取网页内容"""
    # 实现略
    return html_content

@mcp.tool()
def extract_links(html: str) -> str:
    """从 HTML 中提取链接"""
    from bs4 import BeautifulSoup
    
    soup = BeautifulSoup(html, 'html.parser')
    links = [a['href'] for a in soup.find_all('a', href=True)]
    
    return json.dumps(links)

@mcp.tool()
def summarize_text(text: str, max_length: int = 200) -> str:
    """总结文本"""
    # 简单的总结实现
    if len(text) <= max_length:
        return text
    return text[:max_length] + "..."

# LLM 可以链式调用：
# 1. fetch_webpage("https://example.com")
# 2. extract_links(result)
# 3. 对每个链接调用 fetch_webpage
# 4. 对每个页面调用 summarize_text
```

### 2.7 实战示例：文件管理工具集

```python
import os
import json
from pathlib import Path
from typing import List

mcp = FastMCP("File Manager Server")

@mcp.tool()
def list_files(directory: str, pattern: str = "*") -> str:
    """
    列出目录中的文件
    
    Args:
        directory: 目录路径
        pattern: 文件名模式（支持通配符）
    
    Returns:
        文件列表（JSON 格式）
    """
    try:
        path = Path(directory)
        
        if not path.exists():
            return json.dumps({"error": f"目录不存在: {directory}"})
        
        if not path.is_dir():
            return json.dumps({"error": f"不是目录: {directory}"})
        
        files = []
        for file_path in path.glob(pattern):
            files.append({
                "name": file_path.name,
                "path": str(file_path),
                "size": file_path.stat().st_size if file_path.is_file() else 0,
                "is_dir": file_path.is_dir(),
                "modified": file_path.stat().st_mtime
            })
        
        return json.dumps(files, indent=2)
        
    except Exception as e:
        return json.dumps({"error": str(e)})

@mcp.tool()
def read_file(file_path: str, encoding: str = "utf-8") -> str:
    """
    读取文件内容
    
    Args:
        file_path: 文件路径
        encoding: 文件编码
    
    Returns:
        文件内容
    """
    try:
        path = Path(file_path)
        
        if not path.exists():
            return f"错误：文件不存在 '{file_path}'"
        
        if not path.is_file():
            return f"错误：不是文件 '{file_path}'"
        
        # 检查文件大小
        size_mb = path.stat().st_size / (1024 * 1024)
        if size_mb > 10:
            return f"错误：文件过大 ({size_mb:.2f} MB)，最大支持 10 MB"
        
        content = path.read_text(encoding=encoding)
        return content
        
    except UnicodeDecodeError:
        return f"错误：无法使用 {encoding} 编码读取文件"
    except Exception as e:
        return f"错误：{e}"

@mcp.tool()
def write_file(file_path: str, content: str, encoding: str = "utf-8") -> str:
    """
    写入文件
    
    Args:
        file_path: 文件路径
        content: 文件内容
        encoding: 文件编码
    
    Returns:
        操作结果
    """
    try:
        path = Path(file_path)
        
        # 创建父目录
        path.parent.mkdir(parents=True, exist_ok=True)
        
        # 写入文件
        path.write_text(content, encoding=encoding)
        
        return f"成功写入文件: {file_path}"
        
    except Exception as e:
        return f"错误：{e}"

@mcp.tool()
def search_in_files(directory: str, keyword: str, file_pattern: str = "*.txt") -> str:
    """
    在文件中搜索关键词
    
    Args:
        directory: 搜索目录
        keyword: 搜索关键词
        file_pattern: 文件名模式
    
    Returns:
        搜索结果（JSON 格式）
    """
    try:
        path = Path(directory)
        results = []
        
        for file_path in path.rglob(file_pattern):
            if file_path.is_file():
                try:
                    content = file_path.read_text(encoding="utf-8")
                    if keyword in content:
                        # 找到关键词所在行
                        lines = content.split('\n')
                        matching_lines = [
                            {"line_number": i + 1, "content": line}
                            for i, line in enumerate(lines)
                            if keyword in line
                        ]
                        
                        results.append({
                            "file": str(file_path),
                            "matches": len(matching_lines),
                            "lines": matching_lines[:5]  # 最多返回 5 行
                        })
                except:
                    continue
        
        return json.dumps(results, indent=2, ensure_ascii=False)
        
    except Exception as e:
        return json.dumps({"error": str(e)})
```

---

## 3. 实现资源（Resources）

### 3.1 资源的概念回顾

资源（Resources）是 MCP Server 暴露的可读取内容，通过 URI 标识。

#### 资源 vs 工具

```
工具（Tools）：
- 执行操作
- 可能有副作用
- 主动调用

资源（Resources）：
- 提供内容
- 只读，无副作用
- 被动读取
```

### 3.2 静态资源

静态资源有固定的 URI，内容相对稳定。

#### 基础静态资源

```python
from mcp.server.fastmcp import FastMCP
import json

mcp = FastMCP("Resource Server")

@mcp.resource("config://app")
def get_app_config() -> str:
    """
    应用配置
    
    Returns:
        应用配置（JSON 格式）
    """
    config = {
        "app_name": "My Application",
        "version": "1.0.0",
        "api_endpoint": "https://api.example.com",
        "timeout": 30
    }
    return json.dumps(config, indent=2)

@mcp.resource("system://info")
def get_system_info() -> str:
    """
    系统信息
    
    Returns:
        系统信息
    """
    import platform
    import psutil
    
    info = {
        "platform": platform.system(),
        "platform_version": platform.version(),
        "python_version": platform.python_version(),
        "cpu_count": psutil.cpu_count(),
        "memory_total_gb": psutil.virtual_memory().total / (1024**3)
    }
    
    return json.dumps(info, indent=2)
```

#### 文件资源

```python
from pathlib import Path

@mcp.resource("file://README.md")
def get_readme() -> str:
    """
    项目 README
    
    Returns:
        README 内容
    """
    readme_path = Path("README.md")
    if readme_path.exists():
        return readme_path.read_text(encoding="utf-8")
    return "README.md not found"

@mcp.resource("file://config.json")
def get_config_file() -> str:
    """
    配置文件
    
    Returns:
        配置文件内容
    """
    config_path = Path("config.json")
    if config_path.exists():
        return config_path.read_text(encoding="utf-8")
    return json.dumps({"error": "Config file not found"})
```

### 3.3 动态资源（模板资源）

动态资源使用模板 URI，支持参数化。

#### 基础模板资源

```python
@mcp.resource("user://{user_id}/profile")
def get_user_profile(user_id: str) -> str:
    """
    用户资料
    
    Args:
        user_id: 用户 ID
    
    Returns:
        用户资料（JSON 格式）
    """
    # 模拟从数据库获取
    users = {
        "1": {"name": "张三", "email": "zhangsan@example.com"},
        "2": {"name": "李四", "email": "lisi@example.com"}
    }
    
    user = users.get(user_id)
    if user:
        return json.dumps(user, ensure_ascii=False, indent=2)
    else:
        return json.dumps({"error": f"User {user_id} not found"})

@mcp.resource("file://{path}")
def get_file_content(path: str) -> str:
    """
    读取文件内容
    
    Args:
        path: 文件路径
    
    Returns:
        文件内容
    """
    try:
        file_path = Path(path)
        if not file_path.exists():
            return f"File not found: {path}"
        
        if not file_path.is_file():
            return f"Not a file: {path}"
        
        return file_path.read_text(encoding="utf-8")
    except Exception as e:
        return f"Error reading file: {e}"
```

#### 复杂模板资源

```python
@mcp.resource("api://{service}/{endpoint}")
def get_api_data(service: str, endpoint: str) -> str:
    """
    API 数据
    
    Args:
        service: 服务名称
        endpoint: 端点路径
    
    Returns:
        API 响应数据
    """
    # 模拟 API 调用
    apis = {
        "users": {
            "list": [{"id": 1, "name": "User 1"}, {"id": 2, "name": "User 2"}],
            "count": {"total": 2}
        },
        "products": {
            "list": [{"id": 1, "name": "Product 1"}],
            "count": {"total": 1}
        }
    }
    
    service_data = apis.get(service, {})
    endpoint_data = service_data.get(endpoint)
    
    if endpoint_data is not None:
        return json.dumps(endpoint_data, indent=2)
    else:
        return json.dumps({"error": f"Endpoint not found: {service}/{endpoint}"})

@mcp.resource("db://{table}/{id}")
def get_database_record(table: str, id: str) -> str:
    """
    数据库记录
    
    Args:
        table: 表名
        id: 记录 ID
    
    Returns:
        数据库记录（JSON 格式）
    """
    # 模拟数据库查询
    database = {
        "users": {
            "1": {"id": 1, "name": "张三", "age": 30},
            "2": {"id": 2, "name": "李四", "age": 25}
        },
        "orders": {
            "1": {"id": 1, "user_id": 1, "total": 100.0},
            "2": {"id": 2, "user_id": 2, "total": 200.0}
        }
    }
    
    table_data = database.get(table, {})
    record = table_data.get(id)
    
    if record:
        return json.dumps(record, ensure_ascii=False, indent=2)
    else:
        return json.dumps({"error": f"Record not found: {table}/{id}"})
```

### 3.4 资源列表

提供资源列表让 Client 知道有哪些资源可用。

```python
# FastMCP 会自动生成资源列表
# 但你也可以手动控制

@mcp.resource("catalog://resources")
def list_available_resources() -> str:
    """
    可用资源目录
    
    Returns:
        资源列表（JSON 格式）
    """
    resources = [
        {
            "uri": "config://app",
            "name": "应用配置",
            "description": "应用的配置信息"
        },
        {
            "uri": "system://info",
            "name": "系统信息",
            "description": "服务器系统信息"
        },
        {
            "uri_template": "user://{user_id}/profile",
            "name": "用户资料",
            "description": "获取指定用户的资料",
            "parameters": ["user_id"]
        }
    ]
    
    return json.dumps(resources, ensure_ascii=False, indent=2)
```

### 3.5 实战示例：知识库资源

```python
from pathlib import Path
from typing import List
import json

mcp = FastMCP("Knowledge Base Server")

# 知识库目录
KNOWLEDGE_BASE_DIR = Path("./knowledge_base")

@mcp.resource("kb://index")
def get_knowledge_base_index() -> str:
    """
    知识库索引
    
    Returns:
        所有文档的列表
    """
    if not KNOWLEDGE_BASE_DIR.exists():
        return json.dumps({"error": "Knowledge base not found"})
    
    documents = []
    for file_path in KNOWLEDGE_BASE_DIR.rglob("*.md"):
        documents.append({
            "id": file_path.stem,
            "title": file_path.stem.replace("-", " ").title(),
            "path": str(file_path.relative_to(KNOWLEDGE_BASE_DIR)),
            "size": file_path.stat().st_size
        })
    
    return json.dumps(documents, ensure_ascii=False, indent=2)

@mcp.resource("kb://doc/{doc_id}")
def get_knowledge_document(doc_id: str) -> str:
    """
    获取知识库文档
    
    Args:
        doc_id: 文档 ID
    
    Returns:
        文档内容（Markdown 格式）
    """
    # 查找文档
    for file_path in KNOWLEDGE_BASE_DIR.rglob(f"{doc_id}.md"):
        try:
            content = file_path.read_text(encoding="utf-8")
            return content
        except Exception as e:
            return f"Error reading document: {e}"
    
    return f"Document not found: {doc_id}"

@mcp.resource("kb://search/{keyword}")
def search_knowledge_base(keyword: str) -> str:
    """
    搜索知识库
    
    Args:
        keyword: 搜索关键词
    
    Returns:
        搜索结果（JSON 格式）
    """
    results = []
    
    for file_path in KNOWLEDGE_BASE_DIR.rglob("*.md"):
        try:
            content = file_path.read_text(encoding="utf-8")
            if keyword.lower() in content.lower():
                # 提取包含关键词的片段
                lines = content.split('\n')
                matching_lines = [
                    line for line in lines
                    if keyword.lower() in line.lower()
                ]
                
                results.append({
                    "doc_id": file_path.stem,
                    "title": file_path.stem.replace("-", " ").title(),
                    "matches": len(matching_lines),
                    "preview": matching_lines[0] if matching_lines else ""
                })
        except:
            continue
    
    return json.dumps(results, ensure_ascii=False, indent=2)
```

---

## 4. 实现提示（Prompts）

### 4.1 提示的概念

提示（Prompts）是可复用的 LLM 提示模板，帮助标准化常见任务。

#### 提示的价值

```
场景 1：代码审查
─────────────────
不使用提示：
用户每次都要输入完整的审查要求

使用提示：
@code-review <code>
自动生成标准化的审查提示

场景 2：文档生成
─────────────────
不使用提示：
用户需要描述文档格式和要求

使用提示：
@generate-api-doc <function>
自动生成符合规范的 API 文档
```

### 4.2 基础提示实现

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Prompt Server")

@mcp.prompt()
def code_review(code: str, language: str = "python") -> str:
    """
    代码审查提示
    
    Args:
        code: 要审查的代码
        language: 编程语言
    
    Returns:
        审查提示
    """
    return f"""请审查以下 {language} 代码，关注以下方面：

1. 代码质量和可读性
2. 潜在的 bug 和错误
3. 性能问题
4. 安全隐患
5. 最佳实践建议

代码：
```{language}
{code}
```

请提供详细的审查意见和改进建议。"""

@mcp.prompt()
def summarize_text(text: str, max_words: int = 100) -> str:
    """
    文本总结提示
    
    Args:
        text: 要总结的文本
        max_words: 最大字数
    
    Returns:
        总结提示
    """
    return f"""请用不超过 {max_words} 个字总结以下文本的核心内容：

{text}

总结应该：
- 简洁明了
- 保留关键信息
- 使用中文"""

@mcp.prompt()
def translate_text(text: str, target_language: str = "English") -> str:
    """
    翻译提示
    
    Args:
        text: 要翻译的文本
        target_language: 目标语言
    
    Returns:
        翻译提示
    """
    return f"""请将以下文本翻译成 {target_language}：

{text}

翻译要求：
- 准确传达原意
- 符合目标语言习惯
- 保持专业术语的准确性"""
```

### 4.3 高级提示模板

#### 结构化提示

```python
@mcp.prompt()
def generate_api_documentation(
    function_name: str,
    parameters: str,
    return_type: str,
    description: str = ""
) -> str:
    """
    生成 API 文档提示
    
    Args:
        function_name: 函数名称
        parameters: 参数列表
        return_type: 返回类型
        description: 函数描述
    
    Returns:
        API 文档生成提示
    """
    return f"""请为以下 API 函数生成完整的文档：

函数名称：{function_name}
参数：{parameters}
返回类型：{return_type}
{f'描述：{description}' if description else ''}

文档应包括：
1. 函数概述
2. 参数说明（每个参数的类型、是否必需、默认值、说明）
3. 返回值说明
4. 使用示例（至少 2 个）
5. 注意事项
6. 相关函数链接

请使用 Markdown 格式，遵循 Google 风格指南。"""

@mcp.prompt()
def debug_error(
    error_message: str,
    code_context: str,
    language: str = "python"
) -> str:
    """
    调试错误提示
    
    Args:
        error_message: 错误信息
        code_context: 相关代码
        language: 编程语言
    
    Returns:
        调试提示
    """
    return f"""我遇到了一个 {language} 错误，请帮我分析和解决：

错误信息：
```
{error_message}
```

相关代码：
```{language}
{code_context}
```

请提供：
1. 错误原因分析
2. 可能的解决方案（至少 2 个）
3. 修复后的代码示例
4. 如何避免类似错误的建议"""
```

#### 多步骤提示

```python
@mcp.prompt()
def plan_project(
    project_name: str,
    requirements: str,
    tech_stack: str = ""
) -> str:
    """
    项目规划提示
    
    Args:
        project_name: 项目名称
        requirements: 项目需求
        tech_stack: 技术栈
    
    Returns:
        项目规划提示
    """
    tech_info = f"\n技术栈：{tech_stack}" if tech_stack else ""
    
    return f"""请为以下项目制定详细的开发计划：

项目名称：{project_name}
项目需求：
{requirements}{tech_info}

请提供：

## 1. 项目概述
- 项目目标
- 核心功能
- 预期成果

## 2. 技术架构
- 系统架构设计
- 技术选型理由
- 关键技术点

## 3. 开发计划
- 阶段划分
- 每个阶段的任务
- 时间估算

## 4. 风险评估
- 潜在风险
- 应对策略

## 5. 资源需求
- 人力需求
- 工具和服务
- 预算估算

请使用 Markdown 格式，内容要详细具体。"""
```

### 4.4 提示参数验证

```python
from pydantic import BaseModel, Field

class CodeReviewParams(BaseModel):
    """代码审查参数"""
    code: str = Field(..., min_length=1, description="要审查的代码")
    language: str = Field(
        default="python",
        pattern="^(python|javascript|typescript|java|go|rust)$",
        description="编程语言"
    )
    focus_areas: list[str] = Field(
        default=["quality", "security", "performance"],
        description="关注领域"
    )

@mcp.prompt()
def advanced_code_review(params: CodeReviewParams) -> str:
    """
    高级代码审查提示
    
    Args:
        params: 审查参数
    
    Returns:
        审查提示
    """
    focus_descriptions = {
        "quality": "代码质量和可读性",
        "security": "安全隐患",
        "performance": "性能优化",
        "maintainability": "可维护性",
        "testing": "测试覆盖"
    }
    
    focus_list = "\n".join([
        f"{i+1}. {focus_descriptions.get(area, area)}"
        for i, area in enumerate(params.focus_areas)
    ])
    
    return f"""请审查以下 {params.language} 代码，重点关注：

{focus_list}

代码：
```{params.language}
{params.code}
```

请提供详细的审查意见和改进建议。"""
```

### 4.5 实战示例：内容创作提示集

```python
mcp = FastMCP("Content Creation Prompts")

@mcp.prompt()
def write_blog_post(
    topic: str,
    target_audience: str,
    tone: str = "professional",
    word_count: int = 1000
) -> str:
    """
    博客文章写作提示
    
    Args:
        topic: 文章主题
        target_audience: 目标读者
        tone: 写作风格
        word_count: 目标字数
    
    Returns:
        写作提示
    """
    return f"""请撰写一篇关于「{topic}」的博客文章。

目标读者：{target_audience}
写作风格：{tone}
目标字数：约 {word_count} 字

文章结构：
1. 引人入胜的标题
2. 简短的引言（100-150 字）
3. 主体内容（分 3-5 个小节）
4. 实用的建议或总结
5. 行动号召

要求：
- 内容要有深度和见解
- 使用具体的例子和数据
- 语言要清晰易懂
- 适当使用小标题和列表
- SEO 友好"""

@mcp.prompt()
def create_social_media_post(
    content: str,
    platform: str,
    hashtags: int = 3
) -> str:
    """
    社交媒体帖子创作提示
    
    Args:
        content: 核心内容
        platform: 社交平台
        hashtags: 标签数量
    
    Returns:
        创作提示
    """
    platform_specs = {
        "twitter": "280 字符以内",
        "linkedin": "专业正式，1300 字符以内",
        "instagram": "简洁吸引人，2200 字符以内",
        "facebook": "轻松友好，无严格限制"
    }
    
    spec = platform_specs.get(platform.lower(), "适合平台特点")
    
    return f"""请为 {platform} 创作一条帖子：

核心内容：
{content}

要求：
- {spec}
- 包含 {hashtags} 个相关标签
- 吸引目标受众
- 鼓励互动（点赞、评论、分享）
- 符合平台风格

请提供：
1. 帖子正文
2. 推荐的标签
3. 最佳发布时间建议"""

@mcp.prompt()
def generate_email_template(
    purpose: str,
    recipient_type: str,
    tone: str = "professional"
) -> str:
    """
    邮件模板生成提示
    
    Args:
        purpose: 邮件目的
        recipient_type: 收件人类型
        tone: 语气
    
    Returns:
        邮件模板生成提示
    """
    return f"""请生成一个邮件模板：

邮件目的：{purpose}
收件人类型：{recipient_type}
语气：{tone}

模板应包括：
1. 主题行（简洁有力，50 字符以内）
2. 开头问候
3. 正文（清晰表达目的）
4. 行动号召（如果适用）
5. 结尾和签名

请提供：
- 完整的邮件模板
- 可自定义的变量说明
- 使用建议"""
```

---

## 5. 生命周期与上下文管理

### 5.1 Server 生命周期

MCP Server 有明确的生命周期阶段。

#### 生命周期阶段

```
1. 启动（Startup）
   ↓
   初始化资源（数据库连接、缓存等）
   
2. 运行（Running）
   ↓
   处理请求
   
3. 关闭（Shutdown）
   ↓
   清理资源
```

#### 使用 Lifespan 管理生命周期

```python
from contextlib import asynccontextmanager
from mcp.server.fastmcp import FastMCP

@asynccontextmanager
async def lifespan(app):
    """Server 生命周期管理"""
    # 启动时执行
    print("Server starting...")
    
    # 初始化资源
    db = await connect_database()
    cache = {}
    
    # 将资源存储到上下文
    yield {"db": db, "cache": cache}
    
    # 关闭时执行
    print("Server shutting down...")
    await db.close()

mcp = FastMCP("My Server", lifespan=lifespan)

@mcp.tool()
async def query_data(query: str, ctx) -> str:
    """查询数据"""
    # 访问生命周期资源
    db = ctx.lifespan_context["db"]
    result = await db.query(query)
    return str(result)
```

### 5.2 上下文（Context）

上下文提供对请求相关信息和生命周期资源的访问。

#### 访问上下文

```python
from mcp.server.fastmcp import Context

@mcp.tool()
def tool_with_context(param: str, ctx: Context) -> str:
    """使用上下文的工具"""
    
    # 访问生命周期资源
    cache = ctx.lifespan_context.get("cache", {})
    
    # 检查缓存
    if param in cache:
        return f"From cache: {cache[param]}"
    
    # 计算结果
    result = process(param)
    
    # 存入缓存
    cache[param] = result
    
    return f"Computed: {result}"
```

### 5.3 实战示例：带缓存的搜索 Server

```python
from contextlib import asynccontextmanager
from mcp.server.fastmcp import FastMCP, Context
import httpx
import json
from datetime import datetime, timedelta

@asynccontextmanager
async def lifespan(app):
    """生命周期管理"""
    print("🚀 Search Server starting...")
    
    # 初始化缓存和 HTTP 客户端
    cache = {}
    http_client = httpx.AsyncClient()
    stats = {
        "requests": 0,
        "cache_hits": 0,
        "cache_misses": 0
    }
    
    yield {
        "cache": cache,
        "http_client": http_client,
        "stats": stats
    }
    
    # 清理
    await http_client.aclose()
    print(f"📊 Final stats: {stats}")
    print("👋 Search Server shutting down...")

mcp = FastMCP("Search Server", lifespan=lifespan)

@mcp.tool()
async def search_web(query: str, ctx: Context) -> str:
    """
    搜索网络
    
    Args:
        query: 搜索关键词
        ctx: 上下文
    
    Returns:
        搜索结果
    """
    cache = ctx.lifespan_context["cache"]
    http_client = ctx.lifespan_context["http_client"]
    stats = ctx.lifespan_context["stats"]
    
    stats["requests"] += 1
    
    # 检查缓存
    cache_key = f"search:{query}"
    if cache_key in cache:
        cached_data, cached_time = cache[cache_key]
        
        # 缓存有效期 5 分钟
        if datetime.now() - cached_time < timedelta(minutes=5):
            stats["cache_hits"] += 1
            return f"[Cached] {cached_data}"
    
    # 缓存未命中，执行搜索
    stats["cache_misses"] += 1
    
    try:
        response = await http_client.get(
            "https://api.example.com/search",
            params={"q": query}
        )
        result = response.json()
        
        # 存入缓存
        cache[cache_key] = (json.dumps(result), datetime.now())
        
        return json.dumps(result, indent=2)
        
    except Exception as e:
        return f"Search error: {e}"

@mcp.tool()
def get_stats(ctx: Context) -> str:
    """
    获取统计信息
    
    Args:
        ctx: 上下文
    
    Returns:
        统计信息
    """
    stats = ctx.lifespan_context["stats"]
    cache = ctx.lifespan_context["cache"]
    
    hit_rate = (
        stats["cache_hits"] / stats["requests"] * 100
        if stats["requests"] > 0
        else 0
    )
    
    return json.dumps({
        "total_requests": stats["requests"],
        "cache_hits": stats["cache_hits"],
        "cache_misses": stats["cache_misses"],
        "hit_rate": f"{hit_rate:.2f}%",
        "cache_size": len(cache)
    }, indent=2)

@mcp.tool()
def clear_cache(ctx: Context) -> str:
    """
    清空缓存
    
    Args:
        ctx: 上下文
    
    Returns:
        操作结果
    """
    cache = ctx.lifespan_context["cache"]
    size = len(cache)
    cache.clear()
    return f"Cleared {size} cache entries"
```

---

## 6. 测试与调试

### 6.1 单元测试

#### 测试工具函数

```python
# tests/test_tools.py

import pytest
from my_mcp_server.server import mcp

@pytest.mark.asyncio
async def test_hello_tool():
    """测试 hello 工具"""
    # 直接调用工具函数
    result = mcp._tools["hello"].fn(name="Alice")
    assert result == "Hello, Alice!"

@pytest.mark.asyncio
async def test_add_tool():
    """测试 add 工具"""
    result = mcp._tools["add"].fn(a=5, b=3)
    assert result == "8"

@pytest.mark.asyncio
async def test_search_tool_with_cache():
    """测试带缓存的搜索工具"""
    # 模拟上下文
    class MockContext:
        lifespan_context = {
            "cache": {},
            "stats": {"requests": 0, "cache_hits": 0, "cache_misses": 0}
        }
    
    ctx = MockContext()
    
    # 第一次调用（缓存未命中）
    result1 = await search_web("test", ctx)
    assert ctx.lifespan_context["stats"]["cache_misses"] == 1
    
    # 第二次调用（缓存命中）
    result2 = await search_web("test", ctx)
    assert ctx.lifespan_context["stats"]["cache_hits"] == 1
```

### 6.2 集成测试

```python
# tests/test_integration.py

import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

@pytest.mark.asyncio
async def test_server_integration():
    """集成测试：完整的 Server 交互"""
    
    server_params = StdioServerParameters(
        command="uv",
        args=["run", "python", "-m", "my_mcp_server"]
    )
    
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # 初始化
            await session.initialize()
            
            # 测试工具列表
            tools = await session.list_tools()
            assert len(tools.tools) > 0
            
            # 测试工具调用
            result = await session.call_tool("hello", {"name": "Test"})
            assert "Hello, Test" in result.content[0].text
            
            # 测试资源读取
            resources = await session.list_resources()
            if resources.resources:
                resource = await session.read_resource(
                    resources.resources[0].uri
                )
                assert resource.contents
```

### 6.3 调试技巧

#### 启用调试日志

```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

mcp = FastMCP("My Server", log_level="DEBUG")
```

#### 使用 MCP Inspector

```bash
# 启动 Inspector
npx @modelcontextprotocol/inspector

# 配置 Server
{
  "command": "uv",
  "args": ["run", "python", "-m", "my_mcp_server"],
  "env": {
    "DEBUG": "1"
  }
}
```

#### 添加调试输出

```python
@mcp.tool()
def debug_tool(param: str) -> str:
    """带调试输出的工具"""
    import sys
    
    # 输出到 stderr（不会干扰 MCP 协议）
    print(f"DEBUG: Received param: {param}", file=sys.stderr)
    
    result = process(param)
    
    print(f"DEBUG: Returning: {result}", file=sys.stderr)
    
    return result
```

---

## 7. 最佳实践与项目实战

### 7.1 设计最佳实践

#### 1. 工具设计原则

```python
# ✅ 好的设计
@mcp.tool()
def get_user(user_id: str) -> str:
    """
    获取用户信息
    
    Args:
        user_id: 用户 ID
    
    Returns:
        用户信息（JSON 格式）
    """
    pass

# ❌ 避免的设计
@mcp.tool()
def do_something(data: dict) -> str:
    """做某事"""  # 不清楚的描述
    pass
```

#### 2. 错误处理原则

```python
@mcp.tool()
async def safe_api_call(url: str) -> str:
    """安全的 API 调用"""
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(url, timeout=10.0)
            response.raise_for_status()
            return response.text
    except httpx.TimeoutException:
        return json.dumps({"error": "Request timeout"})
    except httpx.HTTPError as e:
        return json.dumps({"error": f"HTTP error: {e}"})
    except Exception as e:
        logging.error(f"Unexpected error: {e}")
        return json.dumps({"error": "Internal server error"})
```

#### 3. 性能优化

```python
# 使用连接池
@asynccontextmanager
async def lifespan(app):
    http_client = httpx.AsyncClient(
        limits=httpx.Limits(max_connections=100),
        timeout=httpx.Timeout(10.0)
    )
    yield {"http_client": http_client}
    await http_client.aclose()

# 使用缓存
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_computation(param: str) -> str:
    """昂贵的计算（带缓存）"""
    return compute(param)
```

### 7.2 完整项目示例：待办事项管理 Server

```python
# todo_server.py

from contextlib import asynccontextmanager
from mcp.server.fastmcp import FastMCP, Context
from datetime import datetime
from typing import List, Dict
import json

@asynccontextmanager
async def lifespan(app):
    """生命周期管理"""
    print("📝 Todo Server starting...")
    
    # 初始化待办事项列表
    todos: List[Dict] = []
    next_id = 1
    
    yield {"todos": todos, "next_id": [next_id]}
    
    print(f"📊 Total todos created: {len(todos)}")
    print("👋 Todo Server shutting down...")

mcp = FastMCP("Todo Server", lifespan=lifespan)

@mcp.tool()
def add_todo(title: str, description: str = "", ctx: Context = None) -> str:
    """
    添加待办事项
    
    Args:
        title: 标题
        description: 描述
        ctx: 上下文
    
    Returns:
        新建的待办事项
    """
    todos = ctx.lifespan_context["todos"]
    next_id = ctx.lifespan_context["next_id"]
    
    todo = {
        "id": next_id[0],
        "title": title,
        "description": description,
        "completed": False,
        "created_at": datetime.now().isoformat()
    }
    
    todos.append(todo)
    next_id[0] += 1
    
    return json.dumps(todo, ensure_ascii=False, indent=2)

@mcp.tool()
def list_todos(status: str = "all", ctx: Context = None) -> str:
    """
    列出待办事项
    
    Args:
        status: 状态过滤（all/active/completed）
        ctx: 上下文
    
    Returns:
        待办事项列表
    """
    todos = ctx.lifespan_context["todos"]
    
    if status == "active":
        filtered = [t for t in todos if not t["completed"]]
    elif status == "completed":
        filtered = [t for t in todos if t["completed"]]
    else:
        filtered = todos
    
    return json.dumps(filtered, ensure_ascii=False, indent=2)

@mcp.tool()
def complete_todo(todo_id: int, ctx: Context = None) -> str:
    """
    完成待办事项
    
    Args:
        todo_id: 待办事项 ID
        ctx: 上下文
    
    Returns:
        操作结果
    """
    todos = ctx.lifespan_context["todos"]
    
    for todo in todos:
        if todo["id"] == todo_id:
            todo["completed"] = True
            todo["completed_at"] = datetime.now().isoformat()
            return json.dumps(todo, ensure_ascii=False, indent=2)
    
    return json.dumps({"error": f"Todo {todo_id} not found"})

@mcp.tool()
def delete_todo(todo_id: int, ctx: Context = None) -> str:
    """
    删除待办事项
    
    Args:
        todo_id: 待办事项 ID
        ctx: 上下文
    
    Returns:
        操作结果
    """
    todos = ctx.lifespan_context["todos"]
    
    for i, todo in enumerate(todos):
        if todo["id"] == todo_id:
            deleted = todos.pop(i)
            return json.dumps({
                "message": "Todo deleted",
                "todo": deleted
            }, ensure_ascii=False, indent=2)
    
    return json.dumps({"error": f"Todo {todo_id} not found"})

@mcp.resource("todos://stats")
def get_todo_stats(ctx: Context) -> str:
    """
    待办事项统计
    
    Returns:
        统计信息
    """
    todos = ctx.lifespan_context["todos"]
    
    total = len(todos)
    completed = sum(1 for t in todos if t["completed"])
    active = total - completed
    
    stats = {
        "total": total,
        "active": active,
        "completed": completed,
        "completion_rate": f"{completed/total*100:.1f}%" if total > 0 else "0%"
    }
    
    return json.dumps(stats, indent=2)

def main():
    """启动 Server"""
    mcp.run()

if __name__ == "__main__":
    main()
```

### 7.3 阶段 3 完成检查清单

- [ ] 能够创建基本的 MCP Server
- [ ] 理解工具、资源、提示的区别和用途
- [ ] 掌握工具的参数验证和错误处理
- [ ] 能够实现静态和动态资源
- [ ] 能够创建实用的提示模板
- [ ] 理解生命周期和上下文管理
- [ ] 能够编写测试用例
- [ ] 掌握调试技巧
- [ ] 能够独立完成一个完整的 MCP Server 项目

### 7.4 下一步

恭喜完成阶段 3！现在你已经：

✅ 掌握了 MCP Server 的开发技能  
✅ 能够实现工具、资源和提示  
✅ 理解了生命周期和上下文管理  
✅ 学会了测试和调试方法  
✅ 了解了最佳实践  

**接下来**，你可以：

👉 **阶段 4：MCP Client 开发与集成**
- 开发自定义 MCP Client
- 集成到现有应用
- 多 Server 管理

👉 **实战项目**
- 开发实用的 MCP Server
- 发布到社区
- 与他人协作

---

## 📚 附录

### A. 常用代码模板

#### 基础 Server 模板

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My Server")

@mcp.tool()
def my_tool(param: str) -> str:
    """工具描述"""
    return f"Result: {param}"

@mcp.resource("my://resource")
def my_resource() -> str:
    """资源描述"""
    return "Resource content"

@mcp.prompt()
def my_prompt(input: str) -> str:
    """提示描述"""
    return f"Prompt for: {input}"

def main():
    mcp.run()

if __name__ == "__main__":
    main()
```

### B. 推荐资源

**官方文档**：
- MCP 规范：https://modelcontextprotocol.io/specification
- Python SDK：https://github.com/modelcontextprotocol/python-sdk
- TypeScript SDK：https://github.com/modelcontextprotocol/typescript-sdk

**示例项目**：
- 官方示例：https://github.com/modelcontextprotocol/servers
- 社区项目：搜索 "mcp-server" on GitHub

**开发工具**：
- MCP Inspector：调试 MCP Server
- uv：Python 包管理器
- pnpm：Node.js 包管理器

---

**祝你开发愉快！🚀**

> 本文档最后更新：2025年11月
> 
> 如有任何问题或建议，欢迎反馈！
