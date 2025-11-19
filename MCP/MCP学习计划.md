# MCP 学习计划（Model Context Protocol）

---

## 一、学习目标与整体路线

- **总目标**
  - 理解 MCP 的整体设计思想与协议规范。
  - 能够独立开发 MCP Server（提供 tools / resources / prompts）。
  - 能够编写 MCP Client / Host，将 MCP 能力接入到自己的 Agent、应用或 IDE 中。
  - 掌握常见传输方式（stdio / SSE 等）、生命周期管理、上下文管理和安全性实践。
  - 最终完成一个「可在实际项目中使用」的 MCP 实战项目。

- **整体用时建议**
  - 有后端 / AI 基础：约 2–3 周（每天 2–3 小时）。
  - 零 MCP 经验但有编程经验：约 3–4 周。

- **学习节奏**
  - 阶段 0：前置知识与环境准备。
  - 阶段 1：MCP 整体认知与核心概念。
  - 阶段 2：协议规范、消息模型与传输层。
  - 阶段 3：MCP Server 开发（Python / TypeScript 实战）。
  - 阶段 4：MCP Client / Host 集成与工具调用链。
  - 阶段 5：生产级能力（生命周期、状态、日志、安全、测试）。
  - 阶段 6：综合实战项目与进阶生态学习。

---

## 二、前置知识与环境准备（阶段 0）

- **核心前置知识**
  - **LLM 与工具调用基础**：
    - 知道什么是「让大模型调用工具 / 函数」以及为什么需要标准协议。
  - **基础后端知识**：
    - JSON、HTTP、异步编程基本概念。
  - **至少一种开发语言**：
    - 推荐 Python 或 TypeScript（官方 SDK 支持好）。

- **环境准备**
  - 安装一门主力语言环境：
    - Python 3.10+，或 Node.js 18+。
  - 安装包管理工具：
    - Python：`uv` / `pip`。
    - Node：`pnpm` / `npm` / `yarn`。
  - 准备一个 LLM 服务：
    - 任一兼容 OpenAI 接口的服务（如 DeepSeek 等），用于演示 function calling + MCP。

- **建议阅读（快速扫一遍，后续分阶段再回看）**
  - MCP 官方 Introduction（了解 MCP 是什么、解决什么问题）。
  - 一篇中文入门文章（如「Model Context Protocol(MCP) 编程极速入门」）。

---

## 三、MCP 总览与核心概念（阶段 1）

> 目标：能用自己的话解释 MCP 是什么、包含哪些核心概念、典型架构长什么样。

- **1. MCP 是什么**
  - 为 LLM / AI 应用提供一个统一的「接口层」，把外部数据源、工具、服务标准化给大模型使用。
  - 类比：给 AI 做一个「USB-C 口」，任何支持 MCP 的 server 都能被任意支持 MCP 的 host 使用。

- **2. 核心角色**
  - **Host / Client（宿主）**：
    - 例如 IDE、聊天应用、Agent 框架、你的自定义应用。
    - 负责：
      - 连接 MCP Servers。
      - 发现它们提供的 tools / resources / prompts。
      - 决定何时调用哪个 tool，把结果回传给 LLM 或用户。
  - **MCP Server**：
    - 提供实际能力的服务进程。
    - 对外暴露：
      - `tools`（可调用函数）。
      - `resources`（可读资源）。
      - `prompts`（可复用提示模板）。

- **3. 核心能力类型**
  - **Tools**：
    - 可带参数的函数，通常有明确的输入 schema 和输出。
    - 典型示例：`web_search(query: str)`、`run_sql(query: str)`、`get_user_profile(id: string)`。
  - **Resources**：
    - 可通过 URI 访问的内容，分为静态资源和带通配符模板。
    - 示例：
      - 静态：`echo://static`
      - 模板：`greeting://{name}`
  - **Prompts**：
    - 可参数化的提示模板，方便在 host 中复用，例如：
      - `翻译专家(target_language)`。

- **4. 连接与传输**
  - MCP 定义是「协议 + 能力模型」，传输层可以是：
    - `stdio`（进程标准输入输出）。
    - `SSE`（Server-Sent Events）。
    - 其他（如 WebSocket）由具体实现扩展。

- **本阶段实践**
  - 用图画出：
    - 用户 / LLM ↔ Host ↔ MCP Server ↔ 外部系统 的整体架构图。
  - 用一段话概括 MCP 的价值并记录在笔记里。

---

## 四、协议规范与消息模型（阶段 2，核心技术）

> 目标：理解 MCP 的「线缆内部」，知道一次 tool 调用、resource 读取在协议层面发生了什么。

- **1. 会话生命周期**
  - 连接建立：
    - Host 启动 MCP Server 进程（stdio）或连接到远程端点（SSE 等）。
  - `initialize` / `initialized`：
    - 交换协议版本、能力（capabilities）、会话 ID 等。
  - 会话关闭：
    - Host 或 Server 主动结束连接，释放资源。

- **2. 能力发现（Capabilities & Listing）**
  - 关键交互：
    - `list_tools` / `list_resources` / `list_resource_templates` / `list_prompts`。
  - 需要掌握：
    - 返回结构中每个 tool / resource / prompt 的字段含义。
    - 特别是 tool 的 `input_schema`（JSON Schema）。

- **3. 请求 / 响应模型**
  - 底层通常基于 JSON-RPC 风格：
    - `request`：带有 `id`、`method`、`params`。
    - `response`：对应 `id` 的结果或错误。
    - `notification`：没有 `id` 的单向消息（如日志、进度）。
  - 典型调用链：
    - Host 发送 `call_tool` → Server 执行 → 返回结果内容块（可能是分片 / 流式）。

- **4. 错误处理与取消**
  - 错误类型：
    - 参数错误（schema 不匹配）。
    - 业务错误（如查询不到数据）。
    - 系统错误（超时、网络中断）。
  - 需要理解：
    - 如何在 Server 端抛出错误，并在协议层以结构化方式返回。
    - Host 如何在 UI 或给 LLM 的消息中良好呈现错误。

- **5. 日志与通知**
  - Server 可以向 Host 发送日志、进度等通知消息。
  - 用途：
    - 调试 MCP Server。
    - 在 UI 中展示执行进度、诊断信息。

- **本阶段实践建议**
  - 选一个官方示例，打开「调试日志」或在本地打印原始 JSON 消息：
    - 观察从 `initialize` → `list_tools` → `call_tool` 全流程的消息内容。
  - 用笔记总结：一次 tool 调用在消息层面发生了哪些步骤。

---

## 五、MCP Server 端开发（阶段 3，核心技术）

> 目标：能够用 Python / TypeScript 写出自己的 MCP Server，提供自定义 tools / resources / prompts。

### 1. Python：使用 FastMCP

- **关键概念**
  - `FastMCP` 应用对象：
    - 定义 server 名称、端口 / 传输方式（stdio / SSE 等）。
  - `@app.tool()` 装饰器：
    - 将一个 async 函数暴露为 MCP tool。
  - `@app.resource()` 装饰器：
    - 定义资源与资源模板。
  - 生命周期与上下文（`lifespan`、`Context`）：
    - 使用 `lifespan` 保持会话范围内的状态（如缓存、连接池）。

- **需要掌握的技术点**
  - 如何定义一个带参数的 tool，如：
    - `async def web_search(query: str) -> str`。
  - 如何通过 `Context` 访问生命周期上下文：
    - 如 `ctx.request_context.lifespan_context.histories` 实现缓存。
  - 如何定义和返回资源：
    - 静态资源 `echo://static`。
    - 模板资源 `greeting://{name}`。
  - 如何启动 Server：
    - `app.run(transport="stdio" 或 "sse")`。

- **阶段练习**
  - 实现一个「简单搜索 + 缓存」的 MCP Server：
    - tool：`web_search(query)`，调用一个 HTTP API 并缓存结果。
    - 使用生命周期上下文存储查询历史，并在 Server 关闭时打印出来。

### 2. TypeScript：使用官方 TypeScript SDK（可选但推荐）

- **掌握点**
  - 初始化 MCP Server：定义名称与传输方式。
  - 使用 TypeScript 类型系统定义工具输入输出 schema。
  - 将 Node.js 生态中的库封装为 MCP tools（如访问数据库、调用外部 REST API）。

- **练习**
  - 用 TS 实现一个「Todo 管理」MCP Server：
    - tools：`add_todo`、`list_todos`、`complete_todo`。

### 3. 设计 MCP Server 时的通用原则

- 工具设计：
  - 输入 schema 要精确，避免让 LLM 自由发挥出错。
  - 工具要小而专一，强副作用操作要明确标注（如删除、写入）。
- 性能与可靠性：
  - 合理超时设置，避免阻塞整个 LLM 会话。
  - 尽量实现幂等接口，方便重试。
- 可观测性：
  - 为关键工具加日志。
  - 在错误返回中带上可诊断的信息（但注意不要泄露敏感数据）。

---

## 六、MCP Client / Host 集成（阶段 4，核心技术）

> 目标：不仅会写 Server，还能写「调用方」，把 MCP 能力串到大模型工作流中。

- **1. 标准 Client 生命周期**
  - 建立连接（例如通过 `stdio_client` 或 SSE 客户端）。
  - 创建 `ClientSession`，执行：
    - `initialize()`
    - `list_tools()` / `list_resources()` / `list_prompts()`。
  - 实际调用：
    - `call_tool(tool_name, args)`。
    - `read_resource(uri)`。
    - `get_prompt(name, arguments)`。

- **2. 与 LLM 工具调用集成**
  - 步骤：
    1. 从 MCP `list_tools()` 获取 tools 列表。
    2. 将每个 tool 映射为 LLM 的 function / tool 描述（包含 `name`、`description`、`input_schema`）。
    3. 调用 LLM Chat Completion API，带上 `tools` 描述。
    4. 解析返回的 `tool_calls`，根据 `name` 和 `arguments` 调用 MCP 的 `call_tool`。
    5. 将 MCP 结果以 `tool` 角色消息回传给 LLM，让其生成最终回答。

- **3. 同步 / 异步与流式**
  - 对 MCP Client 侧通常用异步（async/await）。
  - 工具执行可比较耗时（如爬虫、长任务）时：
    - 需要处理进度通知、超时与取消。

- **4. 在 IDE / Agent 中接入 MCP 的思路**
  - IDE（如 VS Code / Windsurf）：
    - 通过配置文件声明 MCP Servers（名称 + 命令 / URL）。
    - IDE 作为 Host，管理连接与工具调用。
  - 自建 Agent：
    - Agent = LLM + MCP Client + 对话管理逻辑。
    - MCP 提供工具层，Agent 负责何时调用、如何把结果编织进对话。

- **本阶段实践**
  - 写一个最小 MCP Client：
    - 能够连接你的 `web_search` Server。
    - 在命令行中输入自然语言问题 → 由 LLM 决定是否调用工具 → 输出最终回答。

---

## 七、Resources 与 Prompts 的高级用法（阶段 4–5）

> 目标：学会把静态 / 半静态知识与提示工程通过 MCP 标准化管理。

- **1. Resources 设计**
  - 场景：
    - 知识库片段、配置模板、固定说明文档、系统提示等。
  - 技术要点：
    - 静态资源：URI 固定，返回固定内容。
    - 模板资源：URI 中带参数（如 `greeting://{name}`），Server 根据参数动态返回内容。
    - 访问方式：Host 通过 `read_resource(uri)` 获取内容，并组合进 LLM 提示中。

- **2. Prompts 模板**
  - 用 MCP 管理可复用的 Prompt：
    - 例如：翻译专家、SQL 专家、代码审查专家。
  - 技术要点：
    - 定义 Prompt 名称、描述、参数列表。
    - Host 先 `list_prompts()` → `get_prompt(name, arguments)`，得到最终 prompt 文本，再交给 LLM。

- **3. 实践练习**
  - 在现有 Server 上增加：
    - 一个 `company://policies` 资源（返回公司规则）。
    - 一个 `review-expert` Prompt（输入代码片段，输出审查意见）。
  - 写一个简单 Client：
    - 列出所有 prompts，并让你从命令行选一个使用。

---

## 八、生产级能力：生命周期、状态、日志、安全（阶段 5）

> 目标：从 Demo 走向可长期运行的 MCP 服务。

- **1. 生命周期与状态管理**
  - 使用 `lifespan`：
    - 在 Server 启动时初始化共享资源（缓存、数据库连接池、HTTP 客户端等）。
    - 在 Server 关闭时统一清理资源、打印统计信息。
  - 使用上下文对象（如 `AppContext`）：
    - 保存跨 tool 调用共享的状态（如查询历史、会话配置）。

- **2. 缓存与性能优化**
  - 对高频但结果稳定的调用（如 web 搜索）进行缓存。
  - 在 MCP 上下文中记录历史，避免重复请求外部 API。

- **3. 日志与监控**
  - 为每一个 tool 调用打印：
    - 调用时间、参数概况、耗时、结果大小或状态码。
  - 规划：
    - 错误日志渠道（本地文件 / 监控系统）。

- **4. 安全性与权限控制**
  - 不要在 tool 中随意暴露敏感数据。
  - 为危险操作设计「显式」接口和确认机制（例如删除、转账一类操作）。
  - 使用环境变量 / 配置文件管理密钥（如 API Key），不要硬编码。

- **5. 测试与调试**
  - 单元测试：
    - 对工具内部逻辑做普通单测，不依赖 MCP 框架。
  - 集成测试：
    - 启动 MCP Server，通过 Client 发送真实的 `call_tool` 请求，验证协议层和业务逻辑都正常。

---

## 九、分阶段详细学习路径（按天 / 周）

> 可根据自己时间灵活调整，这里按「有一定编程基础」的情况给出一个参考节奏。

### 阶段 0：前置准备（0.5–1 天）

- **目标**：准备好环境，对 MCP 有初步感性认识。
- **任务**：
  - 安装 Python / Node 环境和包管理工具。
  - 配置一个可用的 LLM API Key（如 DeepSeek / 兼容 OpenAI 的服务）。
  - 快速浏览一次：
    - MCP 官方 Introduction。
    - 一篇中文入门（了解大概能做什么）。

### 阶段 1：总体认知与架构（1 天）

- **目标**：理解 MCP 的角色、能力类型和典型架构。
- **任务**：
  - 认真阅读：
    - MCP 简介文档（包括 Host / Server / Tools / Resources / Prompts）。
  - 用自己的语言写出：
    - MCP 是什么。
    - 为什么需要 MCP，而不是每个项目都自造一套工具调用协议。
  - 画出整体架构图并保存到笔记。

### 阶段 2：协议与消息流（1–2 天）

- **目标**：能看懂一次 tool 调用背后的消息流。
- **任务**：
  - 阅读 MCP 规范中与以下相关的部分：
    - 会话初始化（initialize）。
    - 列出 tools / resources / prompts。
    - 调用工具 / 读取资源的消息格式。
    - 错误与通知。
  - 跑一个官方 / 示例 MCP Server，在调试模式下：
    - 抓取并分析一次完整的调用链路消息。
  - 在笔记中总结每种消息的字段含义。

### 阶段 3：MCP Server 实战（Python / TypeScript）（2–3 天）

- **目标**：能写出一个真实可用的 MCP Server。
- **任务（Python 方向示例）**：
  - 基础 Demo：
    - 用 FastMCP 写一个 `web-search` Server：
      - tool：`web_search(query: str)`，调用外部 web 搜索 API，返回结果摘要。
  - 增强：
    - 引入 `lifespan` 与 `AppContext`，为 web 搜索加缓存。
    - 在工具内部增加必要的错误处理和日志打印。
  - 资源与 Prompt：
    - 增加一个静态资源和一个模板资源。
    - 增加一个 Prompt 模板（例如翻译专家），并通过 Client 获取。

### 阶段 4：MCP Client / Host 与 LLM 集成（2–3 天）

- **目标**：把 MCP Server 真正「用起来」，让 LLM 通过 tools 调用你的能力。
- **任务**：
  - 实现一个 `MCPClient` 类：
    - 能：
      - 启动 / 连接 MCP Server（stdio）。
      - `initialize`。
      - `list_tools`、`list_resources`、`list_prompts`。
      - 调用 `call_tool` 并打印结果。
  - 集成 LLM：
    - 从 MCP `list_tools()` 结果构造 LLM 的 `tools` 描述。
    - 根据 LLM 返回的 `tool_calls` 调用 MCP 的 `call_tool`。
    - 将工具执行结果回传给 LLM，再获取最终回答。
  - 做一个 CLI Demo：
    - 输入问题 → 选择是否让 LLM 自动决定调用哪些 MCP 工具 → 打印结果。

### 阶段 5：生产级能力与工程化（3–4 天）

- **目标**：让 MCP Server 可以长期稳定运行在真实环境中。
- **任务**：
  - 生命周期 & 状态：
    - 使用 `lifespan` 管理数据库连接池或 HTTP Client。
    - 为常用查询做缓存并统计命中率。
  - 日志 & 监控：
    - 设计统一日志格式，记录每次 tool 调用的重要信息。
  - 安全 & 配置：
    - 用 `.env` / 配置文件管理 API Key、数据库连接信息。
    - 检查所有工具，确保不返回敏感数据。
  - 测试：
    - 为工具业务逻辑写单元测试。
    - 为典型调用路径写集成测试。

### 阶段 6：综合实战项目与生态扩展（持续）

- **目标**：在自己的业务场景中真正落地 MCP。
- **建议项目方向**：
  - **场景 1：企业内知识库问答**
    - MCP Server：
      - tools：全文检索、FAQ 查询、文档解析。
      - resources：公司制度、技术规范等静态资源。
    - Host：
      - 自建聊天应用或现有 IDE 插件。
  - **场景 2：开发者助手**
    - MCP Server：
      - tools：git 操作、代码搜索、运行测试、部署脚本调用。
      - resources：项目文档、API 文档。
    - Host：
      - IDE、中间件、Agent。

- **生态扩展方向**：
  - 熟悉官方 Python / TypeScript SDK 的更多特性。
  - 学习典型开源 MCP Servers 的实现方式（如数据库、云服务、LSP 等）。
  - 如果你常用的系统还没有 MCP Server，可以考虑自己封装一个。

---

## 十、如何持续迭代你的 MCP 能力

- **1. 持续关注协议与 SDK 更新**
  - MCP 仍在快速演进，关注官方文档与 SDK 仓库的变更日志。

- **2. 将 MCP 纳入日常开发工具箱**
  - 新项目需要对接外部系统时，优先考虑：
    - 能否通过 MCP 把这个能力对外暴露，方便复用与集成。

- **3. 积累自己的 MCP 模板与最佳实践**
  - 总结：
    - 常见工具类型的设计模式。
    - 通用的 Prompt 模板和资源组织方式。
    - 监控、日志、安全等方面的经验。

> 完成以上各阶段后，你将具备：
> 
> - 对 MCP 协议与核心概念的系统理解；
> - 编写 MCP Server 和 MCP Client / Host 的实战能力；
> - 在真实业务场景中利用 MCP 连接各类数据源和工具、构建 AI Agent 工作流的工程能力。
