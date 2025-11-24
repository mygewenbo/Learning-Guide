# Spring AI 完整学习计划

> **最后更新时间**: 2025年11月  
> **Spring AI 版本**: v1.0.3 / v1.1.0-M1  
> **文档来源**: Spring官方文档 + Context7最新资料

---

## 📋 目录

1. [Spring AI 简介](#1-spring-ai-简介)
2. [核心架构](#2-核心架构)
3. [核心组件详解](#3-核心组件详解)
4. [AI 模型提供商](#4-ai-模型提供商)
5. [向量存储与 RAG](#5-向量存储与-rag)
6. [高级功能](#6-高级功能)
7. [实战应用场景](#7-实战应用场景)
8. [学习路线图](#8-学习路线图)

---

## 1. Spring AI 简介

### 1.1 什么是 Spring AI？

**Spring AI** 是 Spring 生态系统中的全新框架，旨在简化 AI 驱动应用程序的开发。它提供了一套 Spring 友好的 API 和抽象层，使开发者能够：

- 🤖 **集成多种 AI 模型**: 支持 OpenAI、Azure、AWS Bedrock、Ollama、Google Vertex AI 等 15+ 提供商
- 🔍 **构建 RAG 应用**: 检索增强生成（Retrieval-Augmented Generation）
- 💾 **向量数据库管理**: 支持 Chroma、Pinecone、Weaviate、PostgreSQL 等 15+ 向量存储
- 🛠️ **函数调用**: 让 AI 模型调用外部工具和 API
- 🎯 **提示工程**: 内置多种提示模式（CoT、Few-Shot、Tree of Thoughts）
- 💬 **对话记忆**: 多轮对话的上下文管理

### 1.2 核心设计原则

Spring AI 遵循 Spring 生态系统的经典设计理念：

1. **便携性（Portability）**: 统一 API，轻松切换不同 AI 提供商
2. **模块化（Modularity）**: 松耦合组件设计，按需组合
3. **自动配置（Auto-configuration）**: Spring Boot 风格的零配置体验
4. **可扩展性（Extensibility）**: 支持自定义实现和扩展
5. **企业级（Production-ready）**: 内置监控、错误处理和安全机制

### 1.3 主要应用场景

- ✅ **智能客服系统**: 基于企业知识库的问答机器人
- ✅ **文档问答（RAG）**: 上传PDF/Word，智能检索并回答问题
- ✅ **代码生成助手**: AI辅助编程和代码审查
- ✅ **内容创作**: 自动生成文章、摘要、翻译
- ✅ **数据分析**: 自然语言查询数据库
- ✅ **多模态应用**: 图像理解、视频分析

---

## 2. 核心架构

### 2.1 整体架构图“


```
┌───────────────────────────────────────────────────────────────┐
│                  Spring AI Application Layer                  │
├───────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐    │
│  │  ChatClient  │  │   Advisors   │  │   Retrieval     │    │
│  │   (Fluent)   │  │ (Middleware) │  │   Augmenter     │    │
│  └──────┬───────┘  └──────┬───────┘  └────────┬────────┘    │
│         │                 │                    │              │
│         └─────────────────┴────────────────────┘              │
│                           │                                    │
├───────────────────────────┼────────────────────────────────────┤
│                           ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              ChatModel Interface (核心接口)             │  │
│  │        • call(Prompt)  • stream(Prompt)                 │  │
│  └─────────────────────────────────────────────────────────┘  │
│                           │                                    │
│         ┌─────────────────┼─────────────────┐                │
│         ▼                 ▼                 ▼                 │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐         │
│  │  OpenAI    │    │   Azure    │    │  Bedrock   │         │
│  │ ChatModel  │    │ ChatModel  │    │ ChatModel  │         │
│  └────────────┘    └────────────┘    └────────────┘         │
│                                                                │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐         │
│  │  Ollama    │    │ Vertex AI  │    │  DeepSeek  │         │
│  │ ChatModel  │    │ ChatModel  │    │ ChatModel  │         │
│  └────────────┘    └────────────┘    └────────────┘         │
│                                                                │
├────────────────────────────────────────────────────────────────┤
│                  Supporting Components                         │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────┐  │
│  │ EmbeddingModel   │  │  VectorStore     │  │ Document  │  │
│  │  (嵌入模型)      │  │  (向量数据库)    │  │  (文档)   │  │
│  └──────────────────┘  └──────────────────┘  └───────────┘  │
│                                                                │
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────┐  │
│  │  ToolCallback    │  │ PromptTemplate   │  │  Memory   │  │
│  │  (函数调用)      │  │  (提示模板)      │  │  (记忆)   │  │
│  └──────────────────┘  └──────────────────┘  └───────────┘  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 分层架构说明

#### 📱 应用层（Application Layer）
- **ChatClient**: 流式 API，提供最简洁的使用体验
- **Advisors**: 中间件机制，用于请求/响应拦截和增强
- **Retrieval Augmenter**: RAG（检索增强生成）实现

#### 🔌 模型抽象层（Model Abstraction Layer）
- **ChatModel**: 核心接口，定义统一的聊天交互契约
- **StreamingChatModel**: 支持流式响应
- **EmbeddingModel**: 文本嵌入向量化

#### 🏭 模型实现层（Model Implementation Layer）
支持 **15+ 主流 AI 提供商**：
- **OpenAI**: GPT-4, GPT-3.5-Turbo
- **Azure OpenAI**: 企业级部署
- **AWS Bedrock**: Claude, Llama, Titan
- **Google Vertex AI**: Gemini, PaLM
- **Ollama**: 本地开源模型
- **DeepSeek, MiniMax**: 国产模型

#### 🧰 工具层（Utility Layer）
- **Vector Stores**: 15+ 向量数据库支持
- **Document Processing**: 文档加载、分割、转换
- **Function Calling**: 工具和 API 集成
- **Prompt Engineering**: 提示模板和工程化

---

## 3. 核心组件详解

### 3.1 ChatModel - 聊天模型接口

#### 3.1.1 核心接口

```java
public interface ChatModel extends Model<Prompt, ChatResponse> {
    
    // 简单字符串交互
    default String call(String message) {...}
    
    // 标准 Prompt 交互
    ChatResponse call(Prompt prompt);
    
    // 流式响应
    Flux<ChatResponse> stream(Prompt prompt);
}
```

#### 3.1.2 请求响应结构

**Prompt（输入）**:
```java
public class Prompt implements ModelRequest<List<Message>> {
    private final List<Message> messages;      // 消息列表
    private ChatOptions modelOptions;          // 模型配置
}
```

**ChatResponse（输出）**:
```java
public class ChatResponse implements ModelResponse<Generation> {
    private final ChatResponseMetadata metadata;
    private final List<Generation> generations;
}
```

#### 3.1.3 使用示例

```java
@Service
public class ChatService {
    @Autowired
    private ChatModel chatModel;
    
    // 简单调用
    public String simpleChat(String message) {
        return chatModel.call(message);
    }
    
    // 高级调用
    public String advancedChat(String question) {
        Prompt prompt = new Prompt(
            List.of(
                new SystemMessage("你是一个Java专家"),
                new UserMessage(question)
            ),
            ChatOptions.builder()
                .temperature(0.7)
                .maxTokens(500)
                .build()
        );
        return chatModel.call(prompt)
            .getResult()
            .getOutput()
            .getContent();
    }
    
    // 流式响应
    public Flux<String> streamingChat(String message) {
        return chatModel.stream(new Prompt(message))
            .map(response -> response.getResult()
                .getOutput().getContent());
    }
}
```

### 3.2 ChatClient - 流式 API

#### 3.2.1 为什么需要 ChatClient？

`ChatClient` 提供更现代、更流畅的 API 设计：

```java
// 传统方式
Prompt prompt = new Prompt(new UserMessage("解释量子计算"));
ChatResponse response = chatModel.call(prompt);
String content = response.getResult().getOutput().getContent();

// ChatClient 方式
String content = ChatClient.create(chatModel)
    .prompt()
    .user("解释量子计算")
    .call()
    .content();
```

#### 3.2.2 核心功能

**1. 构建器模式配置**:
```java
ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultSystem("你是一个AI助手")
    .defaultAdvisors(
        new MessageChatMemoryAdvisor(chatMemory),
        new QuestionAnswerAdvisor(vectorStore)
    )
    .build();
```

**2. 参数化提示**:
```java
String response = chatClient.prompt()
    .user(u -> u.text("用{language}解释{topic}")
        .param("language", "中文")
        .param("topic", "微服务架构"))
    .call()
    .content();
```

**3. 多模态支持**:
```java
String response = chatClient.prompt()
    .user(u -> u.text("描述这张图片")
        .media(MimeTypeUtils.IMAGE_PNG, 
               new ClassPathResource("/image.png")))
    .call()
    .content();
```

**4. 流式内容**:
```java
Flux<String> stream = chatClient.prompt()
    .user("写一篇关于AI的文章")
    .stream()
    .content();

stream.subscribe(System.out::print);
```

### 3.3 Function Calling - 函数调用

#### 3.3.1 核心概念

函数调用允许 AI 模型调用外部工具和 API：

```
User: "北京今天天气怎么样？"
  ↓
AI Model: 识别需要调用 weatherFunction
  ↓
Function Call: weatherFunction("北京", "C")
  ↓
Function Result: {"temp": 25, "condition": "晴"}
  ↓
AI Model: "北京今天天气晴朗，温度25摄氏度"
```

#### 3.3.2 实现方式

**方式1: @Tool 注解**:
```java
@Service
public class WeatherService {
    
    @Tool(description = "获取指定城市的天气")
    public String getWeather(
        @ToolParam(description = "城市名称") String city) {
        
        // 调用实际的天气API
        return String.format("%s: 晴朗, 25°C", city);
    }
}

// 使用
String response = ChatClient.create(chatModel)
    .prompt("北京天气怎么样？")
    .tools(new WeatherService())
    .call()
    .content();
```

**方式2: FunctionToolCallback**:
```java
public record WeatherRequest(String location, String unit) {}
public record WeatherResponse(double temp, String unit) {}

ToolCallback toolCallback = FunctionToolCallback.builder(
    "currentWeather", 
    new Function<WeatherRequest, WeatherResponse>() {
        public WeatherResponse apply(WeatherRequest req) {
            return new WeatherResponse(25.0, "C");
        }
    })
    .description("获取指定位置的天气")
    .inputType(WeatherRequest.class)
    .build();
```

**方式3: Spring Bean 注册**:
```java
@Bean
@Description("获取天气信息")
public Function<WeatherRequest, WeatherResponse> weatherFunction() {
    return new WeatherService();
}

// 使用时指定 bean 名称
String response = chatClient.prompt("上海天气如何？")
    .toolNames("weatherFunction")
    .call()
    .content();
```

### 3.4 Advisors - 中间件机制

#### 3.4.1 核心接口

```java
// 基础接口
public interface Advisor extends Ordered {
    String getName();
}

// 同步调用顾问
public interface CallAdvisor extends Advisor {
    ChatClientResponse adviseCall(
        ChatClientRequest request, 
        CallAdvisorChain chain);
}

// 流式调用顾问
public interface StreamAdvisor extends Advisor {
    Flux<ChatClientResponse> adviseStream(
        ChatClientRequest request, 
        StreamAdvisorChain chain);
}
```

#### 3.4.2 内置 Advisors

**1. MessageChatMemoryAdvisor - 对话记忆**:
```java
ChatMemory chatMemory = new InMemoryChatMemory();

ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(
        MessageChatMemoryAdvisor.builder(chatMemory)
            .maxHistorySize(10)
            .build()
    )
    .build();

// 对话会自动记住历史
chatClient.prompt().user("我叫张三").call().content();
chatClient.prompt().user("我叫什么？").call().content(); 
// 返回: "您叫张三"
```

**2. QuestionAnswerAdvisor - RAG增强**:
```java
QuestionAnswerAdvisor ragAdvisor = QuestionAnswerAdvisor
    .builder(vectorStore)
    .searchRequest(SearchRequest.builder()
        .topK(5)
        .similarityThreshold(0.7)
        .build())
    .build();

ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(ragAdvisor)
    .build();

// 自动检索相关文档并增强回答
String answer = chatClient.prompt()
    .user("Spring Boot的自动配置原理？")
    .call()
    .content();
```

**3. SimpleLoggerAdvisor - 日志记录**:
```java
public class SimpleLoggerAdvisor implements CallAdvisor {
    
    @Override
    public ChatClientResponse adviseCall(
            ChatClientRequest request, 
            CallAdvisorChain chain) {
        
        log.info("Request: {}", request);
        ChatClientResponse response = chain.nextCall(request);
        log.info("Response: {}", response);
        return response;
    }
}
```

---
## 4. AI 模型提供商

### 4.1 OpenAI

#### 配置
**Maven依赖**:
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-openai</artifactId>
</dependency>
```

**application.properties**:
```properties
spring.ai.openai.api-key=${OPENAI_API_KEY}
spring.ai.openai.base-url=https://api.openai.com
spring.ai.openai.chat.options.model=gpt-4
spring.ai.openai.chat.options.temperature=0.7
spring.ai.openai.chat.options.max-tokens=500
```

#### 支持的模型
- **GPT-4**: `gpt-4`, `gpt-4-turbo`, `gpt-4o`
- **GPT-3.5**: `gpt-3.5-turbo`
- **Embeddings**: `text-embedding-ada-002`, `text-embedding-3-small/large`

### 4.2 Ollama（本地部署）

#### 优势
- ✅ 完全本地运行，数据隐私保证
- ✅ 无API费用
- ✅ 支持多种开源模型（Llama, Mistral, CodeLlama等）
- ✅ 适合开发测试

#### 配置
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-ollama</artifactId>
</dependency>
```

```properties
spring.ai.ollama.base-url=http://localhost:11434
spring.ai.ollama.chat.options.model=llama3
spring.ai.ollama.chat.options.temperature=0.7
```

#### 启动Ollama
```bash
# 下载并运行
ollama pull llama3
ollama serve

# Docker方式
docker run -d -p 11434:11434 ollama/ollama
docker exec -it ollama ollama pull llama3
```

### 4.3 主流模型提供商对比

| 提供商 | 优势 | 适用场景 |
|-------|------|---------|
| **OpenAI** | 能力最强，生态最好 | 生产环境，商业应用 |
| **Azure OpenAI** | 企业合规，SLA保障 | 企业级应用 |
| **AWS Bedrock** | Claude优秀，多模型选择 | AWS生态，多样化需求 |
| **Google Vertex AI** | Gemini多模态能力强 | 多模态应用 |
| **Ollama** | 免费，本地部署 | 开发测试，隐私要求高 |
| **DeepSeek** | 中文能力强，性价比高 | 中文应用，成本敏感 |

---
## 5. 向量存储与 RAG

### 5.1 核心概念

**向量存储（Vector Store）** 是构建RAG应用的核心组件，用于存储和检索文本的向量嵌入。

#### 工作流程
```
文档 → EmbeddingModel → 向量(float[]) → VectorStore (存储)
                                              ↓
查询 → EmbeddingModel → 查询向量 → 相似度搜索 → 返回相关文档
```

### 5.2 VectorStore 接口

```java
public interface VectorStore {
    // 添加文档
    void add(List<Document> documents);
    
    // 简单搜索
    List<Document> similaritySearch(String query);
    
    // 高级搜索
    List<Document> similaritySearch(SearchRequest request);
    
    // 删除文档
    void delete(List<String> documentIds);
}
```

### 5.3 Document 模型

```java
Document doc = Document.builder()
    .text("Spring AI是一个AI开发框架")
    .metadata("source", "官方文档")
    .metadata("author", "Spring团队")
    .metadata("date", LocalDate.now())
    .build();
```

### 5.4 EmbeddingModel - 嵌入模型

#### 配置示例（OpenAI）
```java
@Bean
public EmbeddingModel embeddingModel() {
    return new OpenAiEmbeddingModel(
        OpenAiApi.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .build()
    );
}
```

#### 使用示例
```java
@Service
public class DocumentService {
    @Autowired
    private VectorStore vectorStore;
    @Autowired
    private EmbeddingModel embeddingModel;
    
    public void addDocuments(List<String> texts) {
        List<Document> documents = texts.stream()
            .map(text -> Document.builder()
                .text(text)
                .metadata("source", "user_upload")
                .build())
            .toList();
        
        vectorStore.add(documents);
    }
    
    public List<Document> search(String query, int topK) {
        SearchRequest request = SearchRequest.builder()
            .query(query)
            .topK(topK)
            .similarityThreshold(0.7)
            .build();
        
        return vectorStore.similaritySearch(request);
    }
}
```

### 5.5 支持的向量数据库

| 数据库 | 类型 | 特点 |
|--------|------|------|
| **Chroma** | 独立向量数据库 | 开源，易用 |
| **Pinecone** | 云向量数据库 | 托管服务，高性能 |
| **Weaviate** | 开源向量数据库 | 功能丰富 |
| **PgVector** | PostgreSQL扩展 | 利用现有PostgreSQL |
| **Redis** | 内存数据库 | 高性能，适合缓存 |
| **Milvus** | 企业级向量数据库 | 大规模数据 |
| **Elasticsearch** | 搜索引擎 | 混合搜索能力 |

### 5.6 完整 RAG 示例

```java
@Service
public class RAGService {
    @Autowired
    private ChatClient chatClient;
    @Autowired
    private VectorStore vectorStore;
    
    // 1. 加载文档到向量数据库
    public void loadDocuments(Resource pdfFile) {
        // 读取PDF
        PagePdfDocumentReader reader = 
            new PagePdfDocumentReader(pdfFile);
        List<Document> documents = reader.get();
        
        // 文本分割
        TokenTextSplitter splitter = new TokenTextSplitter();
        List<Document> chunks = splitter.apply(documents);
        
        // 存储到向量数据库
        vectorStore.add(chunks);
    }
    
    // 2. RAG 问答
    public String ask(String question) {
        // 使用 QuestionAnswerAdvisor 自动处理 RAG
        return chatClient.prompt()
            .user(question)
            .call()
            .content();
    }
    
    // 3. 手动 RAG 流程
    public String manualRAG(String question) {
        // 检索相关文档
        List<Document> docs = vectorStore.similaritySearch(
            SearchRequest.builder()
                .query(question)
                .topK(3)
                .similarityThreshold(0.7)
                .build()
        );
        
        // 组装上下文
        String context = docs.stream()
            .map(Document::getText)
            .collect(Collectors.joining("\n\n"));
        
        // 生成回答
        return chatClient.prompt()
            .system("根据以下上下文回答问题：\n" + context)
            .user(question)
            .call()
            .content();
    }
}
```

---
## 6. 高级功能

### 6.1 提示工程模式

#### 6.1.1 Few-Shot Learning
```java
String prompt = """
    Q: 当我3岁时，我的伴侣是我年龄的3倍。现在我20岁，我的伴侣多大？
    A: 当我3岁时，伴侣是3 * 3 = 9岁。年龄差是6岁。
       现在我20岁，伴侣是20 + 6 = 26岁。答案是26。
    
    Q: 停车场有15辆车，又来了5辆，现在有多少辆？
    A:
    """;
```

#### 6.1.2 Chain of Thought（思维链）
```java
String prompt = """
    问题: 一个商店有100件商品，上午卖出30件，下午卖出25件，
    剩余商品又进货20件。现在有多少件商品？
    
    让我们一步步思考：
    """;
```

### 6.2 多模态应用

#### 图像理解
```java
String response = chatClient.prompt()
    .user(u -> u.text("这张图片中有什么？")
        .media(MimeTypeUtils.IMAGE_PNG, 
               new ClassPathResource("/image.png")))
    .call()
    .content();
```

#### 视频分析
```java
String response = chatClient.prompt()
    .user(u -> u.text("描述这个视频的内容")
        .media(Media.Format.VIDEO_MP4, 
               new ClassPathResource("/video.mp4")))
    .call()
    .content();
```

### 6.3 提示缓存（Bedrock）

降低成本和延迟：

```java
ChatResponse response = chatModel.call(
    new Prompt(
        List.of(
            new SystemMessage("你是专家..."), // 会被缓存
            new UserMessage("问题")
        ),
        BedrockChatOptions.builder()
            .cacheOptions(BedrockCacheOptions.builder()
                .strategy(BedrockCacheStrategy.SYSTEM_AND_TOOLS)
                .build())
            .build()
    )
);
```

---
## 7. 实战应用场景

### 7.1 智能客服系统

```java
@RestController
@RequestMapping("/api/support")
public class CustomerSupportController {
    
    @Autowired
    private ChatClient chatClient;
    
    @PostMapping("/chat")
    public String chat(
        @RequestParam String conversationId,
        @RequestParam String message) {
        
        return chatClient.prompt()
            .advisors(advisor -> advisor
                .param(ChatMemory.CONVERSATION_ID, conversationId))
            .user(message)
            .call()
            .content();
    }
}
```

### 7.2 文档问答系统

```java
@RestController
@RequestMapping("/api/docs")
public class DocumentQAController {
    
    @Autowired
    private VectorStore vectorStore;
    @Autowired
    private ChatClient chatClient;
    
    @PostMapping("/upload")
    public String uploadDocument(@RequestParam MultipartFile file) {
        Resource resource = file.getResource();
        PagePdfDocumentReader reader = new PagePdfDocumentReader(resource);
        TokenTextSplitter splitter = new TokenTextSplitter();
        
        List<Document> documents = splitter.apply(reader.get());
        vectorStore.add(documents);
        
        return "Uploaded " + documents.size() + " chunks";
    }
    
    @GetMapping("/ask")
    public String ask(@RequestParam String question) {
        return chatClient.prompt()
            .user(question)
            .call()
            .content();
    }
}
```

### 7.3 代码生成助手

```java
@Service
public class CodeAssistant {
    
    @Autowired
    private ChatClient chatClient;
    
    public String generateCode(String requirement) {
        return chatClient.prompt()
            .system("你是Java代码生成专家")
            .user(u -> u.text("""
                请生成{language}代码实现以下需求：
                {requirement}
                
                要求：
                1. 使用Spring Boot
                2. 包含注释
                3. 遵循最佳实践
                """)
                .param("language", "Java")
                .param("requirement", requirement))
            .call()
            .content();
    }
    
    public String reviewCode(String code) {
        return chatClient.prompt()
            .system("你是代码审查专家")
            .user("请审查以下代码并提供改进建议：\n" + code)
            .call()
            .content();
    }
}
```

---
## 8. 学习路线图

### 阶段一：基础入门（1-2周）

**目标**: 掌握Spring AI基本概念和使用

- [ ] 了解Spring AI核心概念
- [ ] 配置第一个ChatModel（推荐Ollama）
- [ ] 实现简单的聊天功能
- [ ] 学习ChatClient的流式API
- [ ] 理解Prompt和ChatResponse结构

**实践项目**: 简单的AI聊天Web应用

### 阶段二：进阶功能（2-3周）

**目标**: 掌握函数调用和RAG

- [ ] 实现函数调用（@Tool注解）
- [ ] 配置向量数据库（PgVector或Chroma）
- [ ] 学习Document和EmbeddingModel
- [ ] 构建简单的RAG应用
- [ ] 使用Advisors实现对话记忆

**实践项目**: 基于企业文档的智能问答系统

### 阶段三：高级应用（3-4周）

**目标**: 生产级应用开发

- [ ] 多模态应用开发
- [ ] 提示工程优化
- [ ] 自定义Advisor开发
- [ ] 性能优化和缓存
- [ ] 错误处理和重试机制
- [ ] 监控和日志

**实践项目**: 企业级智能客服系统

### 阶段四：专家级（持续）

- [ ] 深入源码理解架构
- [ ] 贡献开源社区
- [ ] 优化向量检索算法
- [ ] 研究最新AI模型集成
- [ ] 分享经验和最佳实践

---
## 9. 最佳实践

### 9.1 配置管理

```properties
# 使用环境变量管理敏感信息
spring.ai.openai.api-key=${OPENAI_API_KEY}

# 设置合理的超时
spring.ai.openai.chat.options.timeout=60s

# 控制token使用
spring.ai.openai.chat.options.max-tokens=1000
```

### 9.2 错误处理

```java
@Service
public class RobustChatService {
    
    @Autowired
    private ChatModel chatModel;
    
    @Retryable(maxAttempts = 3, backoff = @Backoff(delay = 1000))
    public String chat(String message) {
        try {
            return chatModel.call(message);
        } catch (Exception e) {
            log.error("Chat failed: {}", e.getMessage());
            return "抱歉，服务暂时不可用";
        }
    }
}
```

### 9.3 性能优化

1. **使用流式响应**: 提升用户体验
2. **向量缓存**: 避免重复嵌入
3. **批量处理**: 批量添加文档
4. **合理分割**: 控制chunk大小（200-500 tokens）

### 9.4 安全建议

- ✅ API密钥使用环境变量
- ✅ 实施速率限制
- ✅ 输入验证和清理
- ✅ 敏感信息过滤
- ✅ 日志脱敏

---
## 10. 学习资源

### 官方资源
- **官方文档**: https://docs.spring.io/spring-ai/reference/
- **GitHub仓库**: https://github.com/spring-projects/spring-ai
- **示例项目**: https://github.com/spring-projects/spring-ai-examples

### 推荐学习路径
1. 先学习Ollama本地部署（免费）
2. 再体验OpenAI的强大能力
3. 构建一个完整的RAG项目
4. 深入研究源码和架构

### 社区支持
- Spring AI官方论坛
- Stack Overflow (#spring-ai)
- GitHub Discussions

---
## 总结

Spring AI 是构建企业级AI应用的强大框架，它将复杂的AI集成变得简单。通过统一的API、丰富的模型支持和完善的工具生态，开发者可以快速构建智能客服、文档问答、代码助手等各类AI应用。

**关键要点**:
- 统一API设计，轻松切换模型
- RAG是核心应用场景
- Function Calling扩展AI能力
- Advisors提供灵活的中间件机制
- 遵循Spring生态最佳实践

**下一步行动**:
1. 安装Ollama并运行第一个示例
2. 构建一个简单的RAG应用
3. 加入Spring AI社区交流

---
*最后更新: 2025年11月*  
*版本: Spring AI v1.0.3 / v1.1.0-M1*