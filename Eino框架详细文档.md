# Eino 框架详细文档

## 目录

- [1. 框架概述](#1-框架概述)
- [2. 核心概念](#2-核心概念)
- [3. 组件系统](#3-组件系统)
- [4. 编排能力](#4-编排能力)
- [5. 流式处理](#5-流式处理)
- [6. 回调机制](#6-回调机制)
- [7. ADK（Agent Development Kit）](#7-adkagent-development-kit)
- [8. 扩展生态](#8-扩展生态)
- [9. 开发工具](#9-开发工具)
- [10. 快速开始](#10-快速开始)
- [11. 最佳实践](#11-最佳实践)
- [12. 架构设计](#12-架构设计)

---

## 1. 框架概述

### 1.1 什么是 Eino

**Eino ['aino]**（谐音 "I know"）是由 CloudWeGo 团队开发的用 Go 语言编写的大型语言模型（LLM）应用开发框架。它从 LangChain、LlamaIndex 等优秀框架中汲取灵感，结合前沿研究与实际应用经验，提供了一个强调简洁性、可扩展性、可靠性与有效性的 LLM 应用开发平台。

### 1.2 核心价值

Eino 为开发者提供以下核心价值：

1. **丰富的组件抽象与实现**
   - 精心整理的组件抽象（ChatModel、Tool、ChatTemplate、Retriever 等）
   - 每个抽象都有多个可开箱即用的实现
   - 组件可轻松复用与组合

2. **强大的编排框架**
   - 自动处理类型检查、流式处理、并发管理
   - 支持切面注入和选项赋值
   - 提供 Chain、Graph、Workflow 三种编排方式

3. **简洁明了的 API 设计**
   - 符合 Go 语言编程惯例
   - 注重 API 的简洁性和清晰度
   - 易于学习和使用

4. **不断扩充的最佳实践**
   - 集成 Flow（如 ReAct Agent）
   - 丰富的示例代码
   - 实际应用场景的参考实现

5. **完整的开发工具链**
   - 从可视化开发到在线追踪
   - 覆盖整个开发生命周期
   - 支持调试和评估

### 1.3 技术特点

- **类型安全**：充分利用 Go 泛型，在编译时确保类型正确
- **流式优先**：原生支持流式处理，自动处理流的拼接、复制、合并
- **并发安全**：内置并发管理，确保状态安全访问
- **可扩展性**：灵活的组件系统和回调机制
- **生产就绪**：经过实际应用验证，性能可靠

### 1.4 适用场景

Eino 适用于以下 AI 应用开发场景：

- **智能对话系统**：构建基于 LLM 的聊天机器人
- **智能体（Agent）应用**：开发具有工具调用能力的 AI Agent
- **RAG（检索增强生成）系统**：构建知识库问答系统
- **多智能体系统**：实现协作型多 Agent 应用
- **工作流自动化**：使用 LLM 自动化复杂业务流程

---

## 2. 核心概念

### 2.1 组件（Component）

组件是 Eino 框架的基本构建块，每个组件都有明确的输入输出类型和功能职责。

#### 组件类型

```go
const (
    ComponentOfPrompt      Component = "ChatTemplate"
    ComponentOfChatModel   Component = "ChatModel"
    ComponentOfEmbedding   Component = "Embedding"
    ComponentOfIndexer     Component = "Indexer"
    ComponentOfRetriever   Component = "Retriever"
    ComponentOfLoader      Component = "Loader"
    ComponentOfTransformer Component = "DocumentTransformer"
    ComponentOfTool        Component = "Tool"
)
```

#### 组件接口特点

1. **明确的输入输出类型**：每个组件接口都定义了明确的输入和输出类型
2. **标准化的选项机制**：通过 Option 模式配置组件行为
3. **流式处理支持**：组件原生支持流式输入输出
4. **透明的实现细节**：使用时只需关注接口，不关心实现

#### 组件可组合性

- 组件实现可以嵌套使用
- 复杂的业务逻辑可以封装为新组件
- 从外部看，实现细节完全透明

### 2.2 编排（Orchestration）

编排是将多个组件连接起来，形成有向数据流的过程。Eino 提供三种编排方式：

| 编排方式 | 特点 | 适用场景 |
|---------|------|---------|
| **Chain** | 简单的链式有向图，只能向前推进 | 简单的顺序处理流程 |
| **Graph** | 循环或非循环有向图，功能强大且灵活 | 复杂的业务逻辑，需要条件分支 |
| **Workflow** | 非循环图，支持在结构体字段级别进行数据映射 | 需要精细数据映射的场景 |

### 2.3 节点（Node）和边（Edge）

- **节点（Node）**：图中的组件实例，执行具体的处理逻辑
- **边（Edge）**：连接节点的数据流通道
- **分支（Branch）**：根据条件动态选择执行路径
- **状态（State）**：在图执行过程中共享的全局状态

### 2.4 流式处理（Streaming）

Eino 原生支持流式处理，提供四种流式范式：

| 流处理范式 | 输入类型 | 输出类型 | 说明 |
|-----------|---------|---------|------|
| **Invoke** | 非流式 I | 非流式 O | 标准的同步调用 |
| **Stream** | 非流式 I | 流式 StreamReader[O] | 接收完整输入，流式输出 |
| **Collect** | 流式 StreamReader[I] | 非流式 O | 流式输入，等待完整输出 |
| **Transform** | 流式 StreamReader[I] | 流式 StreamReader[O] | 端到端流式处理 |

### 2.5 回调（Callbacks）

回调机制用于处理横切面关注点，如日志、追踪、指标统计等。

#### 回调时机

```go
const (
    TimingOnStart                   // 组件开始执行
    TimingOnEnd                     // 组件结束执行
    TimingOnError                   // 组件执行出错
    TimingOnStartWithStreamInput    // 开始执行（流式输入）
    TimingOnEndWithStreamOutput     // 结束执行（流式输出）
)
```

### 2.6 Message 消息

Message 是 Eino 中最核心的数据结构，用于表示 LLM 的输入输出。

#### 消息角色类型

```go
const (
    Assistant   RoleType = "assistant"  // 助手消息（LLM 返回）
    User        RoleType = "user"       // 用户消息
    System      RoleType = "system"     // 系统消息（设定提示）
    Tool        RoleType = "tool"       // 工具消息（工具执行结果）
)
```

---

## 3. 组件系统

### 3.1 ChatModel（对话模型）

ChatModel 是与大语言模型交互的核心组件。

#### 接口定义

```go
type BaseChatModel interface {
    Generate(ctx context.Context, input []*schema.Message, opts ...Option) (*schema.Message, error)
    Stream(ctx context.Context, input []*schema.Message, opts ...Option) (*schema.StreamReader[*schema.Message], error)
}

type ToolCallingChatModel interface {
    BaseChatModel
    WithTools(tools []*schema.ToolInfo) (ToolCallingChatModel, error)
}
```

#### 主要功能

- **Generate**：同步生成完整的响应消息
- **Stream**：流式生成响应，实时输出消息块
- **WithTools**：绑定工具，支持 Tool Calling

#### 使用示例

```go
// 创建 ChatModel 实例
model, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
    BaseURL: openAPIBaseURL,
    APIKey:  openAPIAK,
    Model:   "gpt-4",
})

// 同步调用
message, _ := model.Generate(ctx, []*schema.Message{
    schema.SystemMessage("you are a helpful assistant."),
    schema.UserMessage("what is AI?"),
})

// 流式调用
stream, _ := model.Stream(ctx, messages)
for {
    msg, err := stream.Recv()
    if err == io.EOF {
        break
    }
    fmt.Print(msg.Content)
}
```

### 3.2 Tool（工具）

Tool 组件用于扩展 LLM 的能力，让 LLM 可以调用外部工具。

#### 接口定义

```go
type BaseTool interface {
    Info(ctx context.Context) (*schema.ToolInfo, error)
}

type InvokableTool interface {
    BaseTool
    InvokableRun(ctx context.Context, argumentsInJSON string, opts ...Option) (string, error)
}

type StreamableTool interface {
    BaseTool
    StreamableRun(ctx context.Context, argumentsInJSON string, opts ...Option) (*schema.StreamReader[string], error)
}
```

#### 工具信息

ToolInfo 包含工具的元数据，用于 LLM 理解工具功能：

- **Name**：工具名称
- **Desc**：工具描述
- **ParamsOneOf**：参数的 JSON Schema 定义

#### 使用示例

```go
// 自定义工具
type WeatherTool struct{}

func (t *WeatherTool) Info(ctx context.Context) (*schema.ToolInfo, error) {
    return &schema.ToolInfo{
        Name: "get_weather",
        Desc: "获取指定城市的天气信息",
        ParamsOneOf: schema.NewParamsOneOfByParams(map[string]*schema.ParameterInfo{
            "city": {
                Type:     "string",
                Desc:     "城市名称",
                Required: true,
            },
        }),
    }, nil
}

func (t *WeatherTool) InvokableRun(ctx context.Context, argumentsInJSON string, opts ...Option) (string, error) {
    // 解析参数
    var args struct {
        City string `json:"city"`
    }
    json.Unmarshal([]byte(argumentsInJSON), &args)
    
    // 执行工具逻辑
    weather := getWeather(args.City)
    return weather, nil
}
```

### 3.3 ChatTemplate（对话模板）

ChatTemplate 用于将变量渲染成消息列表。

#### 模板格式类型

```go
const (
    FString    FormatType = 0  // Python f-string 格式
    GoTemplate FormatType = 1  // Go text/template 格式
    Jinja2     FormatType = 2  // Jinja2 模板格式
)
```

#### 使用示例

```go
// 使用 FString 格式
template := prompt.FromMessages(
    schema.FString,
    schema.SystemMessage("You are a {role}."),
    schema.UserMessage("{input}"),
)

// 渲染消息
messages, _ := template.Format(ctx, map[string]any{
    "role":  "helpful assistant",
    "input": "What is AI?",
})
```

### 3.4 Retriever（检索器）

Retriever 用于从知识库中检索相关文档。

#### 接口定义

```go
type Retriever interface {
    Retrieve(ctx context.Context, query string, opts ...Option) ([]*schema.Document, error)
}
```

#### 使用场景

- RAG（检索增强生成）系统
- 知识库问答
- 文档搜索

### 3.5 Document Loader（文档加载器）

Document Loader 用于从各种数据源加载文档。

#### 支持的数据源

- 本地文件
- Web URL
- Amazon S3
- 数据库
- API 接口

### 3.6 Embedding（嵌入模型）

Embedding 用于将文本转换为向量表示。

#### 接口定义

```go
type Embedder interface {
    EmbedStrings(ctx context.Context, texts []string, opts ...Option) ([][]float64, error)
}
```

### 3.7 Indexer（索引器）

Indexer 用于将文档存储到向量数据库。

#### 功能

- 文档向量化
- 存储到向量数据库
- 支持批量索引

### 3.8 Lambda（自定义函数）

Lambda 允许在编排中使用自定义函数。

#### 使用示例

```go
// 创建 Lambda 节点
lambda := compose.InvokableLambda(func(ctx context.Context, input string) (string, error) {
    // 自定义处理逻辑
    return strings.ToUpper(input), nil
})

// 在 Chain 中使用
chain.AppendLambda(lambda)
```

---

## 4. 编排能力

### 4.1 Chain（链式编排）

Chain 是最简单的编排方式，适用于线性的处理流程。

#### 特点

- 简单直观，易于理解
- 只能向前推进，不支持循环
- 支持条件分支和并行执行

#### 基本使用

```go
// 创建 Chain
chain := compose.NewChain[map[string]any, *schema.Message]()

// 添加节点
chain.
    AppendChatTemplate(template).
    AppendChatModel(model)

// 编译
runnable, _ := chain.Compile(ctx)

// 执行
output, _ := runnable.Invoke(ctx, map[string]any{
    "query": "What's your name?",
})
```

#### 高级特性

##### 1. 条件分支

```go
// 定义分支条件
branchCond := func(ctx context.Context, input map[string]any) (string, error) {
    if input["type"] == "A" {
        return "branch_a", nil
    }
    return "branch_b", nil
}

// 创建分支
branch := compose.NewChainBranch(branchCond).
    AddLambda("branch_a", lambdaA).
    AddLambda("branch_b", lambdaB)

// 添加到 Chain
chain.AppendBranch(branch)
```

##### 2. 并行执行

```go
// 创建并行节点
parallel := compose.NewParallel()
parallel.
    AddLambda("task1", lambda1).
    AddLambda("task2", lambda2)

// 添加到 Chain
chain.AppendParallel(parallel)
```

### 4.2 Graph（图编排）

Graph 是最灵活的编排方式，支持复杂的业务逻辑。

#### 特点

- 支持循环和非循环图
- 支持动态分支
- 支持全局状态管理
- 最灵活，可实现任意复杂逻辑

#### 基本使用

```go
// 创建 Graph
graph := compose.NewGraph[map[string]any, *schema.Message]()

// 添加节点
_ = graph.AddChatTemplateNode("template", chatTemplate)
_ = graph.AddChatModelNode("model", chatModel)
_ = graph.AddToolsNode("tools", toolsNode)
_ = graph.AddLambdaNode("converter", lambda)

// 添加边
_ = graph.AddEdge(compose.START, "template")
_ = graph.AddEdge("template", "model")
_ = graph.AddEdge("tools", "converter")
_ = graph.AddEdge("converter", compose.END)

// 添加条件分支
_ = graph.AddBranch("model", branchFunc)

// 编译
runnable, _ := graph.Compile(ctx)

// 执行
output, _ := runnable.Invoke(ctx, input)
```

#### 节点类型

1. **START**：图的起始节点
2. **END**：图的结束节点
3. **普通节点**：执行具体逻辑的组件节点
4. **Lambda 节点**：自定义函数节点

#### 分支（Branch）

分支允许根据运行时条件动态选择执行路径：

```go
// 定义分支函数
branchFunc := func(ctx context.Context, msg *schema.Message) (string, error) {
    if len(msg.ToolCalls) > 0 {
        return "tools", nil  // 有工具调用，执行工具
    }
    return compose.END, nil  // 无工具调用，结束
}

// 添加分支
graph.AddBranch("model", branchFunc)
```

#### 状态管理（State）

Graph 支持全局状态，在图执行过程中共享数据：

```go
// 定义状态类型
type GraphState struct {
    Messages    []*schema.Message
    Iterations  int
    MaxIter     int
}

// 创建带状态的 Graph
graph := compose.NewGraphWithState[GraphState]()

// 在节点中访问和修改状态
stateReader := func(ctx context.Context, state *GraphState) (*schema.Message, error) {
    // 读取状态
    messages := state.Messages
    
    // 修改状态
    state.Iterations++
    
    return messages[len(messages)-1], nil
}

// 添加状态处理节点
graph.AddStateHandlerNode("reader", stateReader)
```

### 4.3 Workflow（工作流）

Workflow 支持在结构体字段级别进行精细的数据映射。

#### 特点

- 支持字段级别的数据映射
- 类型安全的数据转换
- 适用于复杂的数据流转场景

#### 基本使用

```go
// 定义输入输出类型
type Input1 struct {
    Query string
}

type Output1 struct {
    Result string
}

type Input2 struct {
    Data string
}

// 创建 Workflow
wf := compose.NewWorkflow[[]*schema.Message, *schema.Message]()

// 添加节点并映射字段
wf.AddChatModelNode("model", chatModel).
    AddInput(compose.START)

wf.AddLambdaNode("lambda1", lambda1).
    AddInput("model", compose.MapFields("Content", "Input"))

wf.AddLambdaNode("lambda2", lambda2).
    AddInput("lambda1", compose.MapFields("Result", "Data"))

wf.End().AddInput("lambda2")

// 编译和执行
runnable, _ := wf.Compile(ctx)
output, _ := runnable.Invoke(ctx, messages)
```

#### 字段映射

MapFields 函数用于在节点间映射字段：

```go
// 将 source 节点的 OutputField 映射到目标节点的 InputField
AddInput("source", MapFields("OutputField", "InputField"))
```

---

## 5. 流式处理

### 5.1 流式处理的重要性

在 LLM 应用中，流式处理至关重要：

1. **实时响应**：用户可以立即看到 LLM 的输出
2. **降低延迟感知**：逐步输出比等待完整响应体验更好
3. **资源效率**：不需要等待完整响应再处理
4. **复杂编排**：多个组件需要协同处理流式数据

### 5.2 流式处理机制

#### StreamReader

StreamReader 是 Eino 的核心流式接口：

```go
type StreamReader[T any] interface {
    Recv() (T, error)  // 接收下一个数据块
    Close()            // 关闭流
}
```

#### 自动流处理

Eino 自动处理以下流操作：

1. **拼接（Concatenate）**
   - 当下游节点只接受非流式输入时
   - 自动将流式输出拼接成完整数据

2. **装箱（Box）**
   - 当需要流式输出但上游是非流式时
   - 自动将非流式数据装箱为流

3. **合并（Merge）**
   - 当多个流汇聚到一个节点时
   - 自动合并多个流

4. **复制（Copy）**
   - 当流需要分发到多个下游节点时
   - 自动复制流到各个分支

### 5.3 流式范式使用

#### Invoke（非流式）

```go
// 标准的非流式调用
output, err := runnable.Invoke(ctx, input)
```

#### Stream（流式输出）

```go
// 接收非流式输入，返回流式输出
streamReader, err := runnable.Stream(ctx, input)

for {
    chunk, err := streamReader.Recv()
    if err == io.EOF {
        break
    }
    // 处理 chunk
    fmt.Print(chunk.Content)
}
streamReader.Close()
```

#### Collect（流式输入）

```go
// 接收流式输入，返回非流式输出
inputStream := createInputStream()
output, err := runnable.Collect(ctx, inputStream)
```

#### Transform（端到端流式）

```go
// 接收流式输入，返回流式输出
inputStream := createInputStream()
outputStream, err := runnable.Transform(ctx, inputStream)

for {
    chunk, err := outputStream.Recv()
    if err == io.EOF {
        break
    }
    // 处理 chunk
}
outputStream.Close()
```

### 5.4 流式处理最佳实践

1. **优先使用流式**：在交互式应用中使用 Stream 提升用户体验
2. **正确关闭流**：使用 defer 确保流被正确关闭
3. **错误处理**：区分 io.EOF 和真正的错误
4. **缓冲处理**：对于高频流，考虑使用缓冲

```go
// 最佳实践示例
streamReader, err := runnable.Stream(ctx, input)
if err != nil {
    return err
}
defer streamReader.Close()

for {
    chunk, err := streamReader.Recv()
    if err == io.EOF {
        break
    }
    if err != nil {
        return err
    }
    
    // 处理 chunk
    processChunk(chunk)
}
```

---

## 6. 回调机制

### 6.1 回调概述

回调机制用于在组件执行的不同阶段插入自定义逻辑，主要用于：

- 日志记录
- 性能追踪
- 指标统计
- 调试信息
- 错误监控

### 6.2 回调接口

```go
type Handler interface {
    OnStart(ctx context.Context, info *RunInfo, input CallbackInput) context.Context
    OnEnd(ctx context.Context, info *RunInfo, output CallbackOutput) context.Context
    OnError(ctx context.Context, info *RunInfo, err error) context.Context
    OnStartWithStreamInput(ctx context.Context, info *RunInfo, input *schema.StreamReader[CallbackInput]) context.Context
    OnEndWithStreamOutput(ctx context.Context, info *RunInfo, output *schema.StreamReader[CallbackOutput]) context.Context
}
```

### 6.3 RunInfo

RunInfo 包含组件执行的元信息：

```go
type RunInfo struct {
    Name      string      // 组件名称
    Type      string      // 组件类型
    Component Component   // 组件类别
}
```

### 6.4 创建回调处理器

#### 使用 HandlerBuilder

```go
handler := callbacks.NewHandlerBuilder().
    OnStartFn(func(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
        log.Printf("开始执行 %s: %v", info.Name, input)
        return ctx
    }).
    OnEndFn(func(ctx context.Context, info *RunInfo, output CallbackOutput) context.Context {
        log.Printf("执行完成 %s: %v", info.Name, output)
        return ctx
    }).
    OnErrorFn(func(ctx context.Context, info *RunInfo, err error) context.Context {
        log.Printf("执行错误 %s: %v", info.Name, err)
        return ctx
    }).
    Build()
```

#### 实现完整接口

```go
type MyHandler struct{}

func (h *MyHandler) OnStart(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
    // 自定义开始逻辑
    return ctx
}

func (h *MyHandler) OnEnd(ctx context.Context, info *RunInfo, output CallbackOutput) context.Context {
    // 自定义结束逻辑
    return ctx
}

func (h *MyHandler) OnError(ctx context.Context, info *RunInfo, err error) context.Context {
    // 自定义错误处理
    return ctx
}

func (h *MyHandler) OnStartWithStreamInput(ctx context.Context, info *RunInfo, input *schema.StreamReader[CallbackInput]) context.Context {
    // 流式输入处理
    return ctx
}

func (h *MyHandler) OnEndWithStreamOutput(ctx context.Context, info *RunInfo, output *schema.StreamReader[CallbackOutput]) context.Context {
    // 流式输出处理
    return ctx
}
```

### 6.5 使用回调

#### 全局回调

```go
// 设置全局回调，对所有节点生效
callbacks.AppendGlobalHandlers(handler1, handler2)
```

#### 运行时回调

```go
// 所有节点
runnable.Invoke(ctx, input, compose.WithCallbacks(handler))

// 指定组件类型
runnable.Invoke(ctx, input, compose.WithChatModelCallbacks(handler))

// 指定节点
runnable.Invoke(ctx, input, 
    compose.WithCallbacks(handler).DesignateNode("node_1"))
```

### 6.6 回调最佳实践

1. **避免阻塞**：回调函数应快速返回，避免阻塞主流程
2. **错误处理**：回调内部错误不应影响主流程
3. **性能考虑**：频繁调用的回调要注意性能
4. **链式传递**：可以通过 context 在回调间传递数据

```go
// 通过 context 传递追踪 ID
handler := callbacks.NewHandlerBuilder().
    OnStartFn(func(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
        traceID := uuid.New().String()
        ctx = context.WithValue(ctx, "trace_id", traceID)
        log.Printf("[%s] 开始执行 %s", traceID, info.Name)
        return ctx
    }).
    OnEndFn(func(ctx context.Context, info *RunInfo, output CallbackOutput) context.Context {
        traceID := ctx.Value("trace_id").(string)
        log.Printf("[%s] 执行完成 %s", traceID, info.Name)
        return ctx
    }).
    Build()
```

---

## 7. ADK（Agent Development Kit）

### 7.1 ADK 概述

ADK（Agent Development Kit）是 Eino 提供的高级智能体开发工具包，包含预构建的智能体模式和工具。

### 7.2 ReAct Agent

ReAct（Reasoning and Acting）是最常用的智能体模式。

#### 工作原理

1. **Reasoning（推理）**：LLM 分析问题，决定是否需要使用工具
2. **Acting（行动）**：如果需要，调用相应的工具
3. **Observation（观察）**：工具执行结果返回给 LLM
4. **循环**：重复上述过程直到得出最终答案

#### 使用 ReAct Agent

```go
import "github.com/cloudwego/eino/flow/agent/react"

// 创建 ChatModel
chatModel, _ := openai.NewChatModel(ctx, config)

// 创建工具
tools := []tool.InvokableTool{
    weatherTool,
    calculatorTool,
}

// 创建 ReAct Agent
agent, _ := react.NewReActAgent(ctx, &react.Config{
    Model:    chatModel,
    Tools:    tools,
    MaxSteps: 10,  // 最大迭代次数
})

// 使用 Agent
output, _ := agent.Invoke(ctx, []*schema.Message{
    schema.UserMessage("北京今天天气怎么样？"),
})

fmt.Println(output.Content)
```

#### ReAct 配置选项

```go
type Config struct {
    Model       model.ToolCallingChatModel  // 必需：支持工具调用的模型
    Tools       []tool.InvokableTool        // 必需：可用的工具列表
    MaxSteps    int                         // 可选：最大迭代次数，默认 10
    SystemMsg   string                      // 可选：系统提示
}
```

### 7.3 Plan-Execute Agent

Plan-Execute 模式将任务分解为计划和执行两个阶段。

#### 工作原理

1. **Planning（计划）**：LLM 制定解决问题的步骤
2. **Execution（执行）**：按步骤执行，可能使用工具
3. **Verification（验证）**：检查每步结果，必要时重新规划

#### 使用示例

```go
import "github.com/cloudwego/eino/adk/prebuilt/planexecute"

// 创建 Plan-Execute Agent
agent, _ := planexecute.NewPlanExecuteAgent(ctx, &planexecute.Config{
    Planner:  plannerModel,
    Executor: executorModel,
    Tools:    tools,
})
```

### 7.4 Supervisor Agent

Supervisor 模式用于协调多个子 Agent。

#### 使用场景

- 复杂任务分解给不同的专家 Agent
- 多 Agent 协作
- 任务路由和调度

#### 使用示例

```go
import "github.com/cloudwego/eino/flow/agent/multiagent/host"

// 定义子 Agent
agents := map[string]compose.Runnable{
    "researcher": researcherAgent,
    "writer":     writerAgent,
    "reviewer":   reviewerAgent,
}

// 创建 Supervisor
supervisor, _ := host.NewSupervisor(ctx, &host.Config{
    Supervisor: supervisorModel,
    Agents:     agents,
})
```

### 7.5 Agent 工具

#### AgentTool

AgentTool 允许将一个 Agent 封装为 Tool，供其他 Agent 使用：

```go
// 将 Agent 封装为 Tool
agentTool := adk.AgentToTool(
    "research_agent",
    "执行深度研究的专家",
    researchAgent,
)

// 在另一个 Agent 中使用
mainAgent, _ := react.NewReActAgent(ctx, &react.Config{
    Model: model,
    Tools: []tool.InvokableTool{
        agentTool,  // 使用 Agent 作为工具
        otherTools,
    },
})
```

---

## 8. 扩展生态

### 8.1 EinoExt 组件实现

EinoExt 提供了大量组件的官方实现。

#### ChatModel 实现

| 提供商 | 实现 |
|--------|------|
| OpenAI | `eino-ext/components/model/openai` |
| Anthropic (Claude) | `eino-ext/components/model/anthropic` |
| Google (Gemini) | `eino-ext/components/model/google` |
| 字节火山引擎 (Ark) | `eino-ext/components/model/ark` |
| Ollama | `eino-ext/components/model/ollama` |

#### Tool 实现

| 工具 | 实现 |
|------|------|
| Google Search | `eino-ext/components/tool/googlesearch` |
| DuckDuckGo Search | `eino-ext/components/tool/duckduckgo` |
| Weather API | `eino-ext/components/tool/weather` |
| Web Scraper | `eino-ext/components/tool/webscraper` |

#### Retriever 实现

| 向量数据库 | 实现 |
|-----------|------|
| Elasticsearch | `eino-ext/components/retriever/elasticsearch` |
| 火山引擎 VikingDB | `eino-ext/components/retriever/vikingdb` |
| Chroma | `eino-ext/components/retriever/chroma` |
| Milvus | `eino-ext/components/retriever/milvus` |

#### Document Loader 实现

| 数据源 | 实现 |
|--------|------|
| Local File | `eino-ext/components/document/loader/file` |
| Web URL | `eino-ext/components/document/loader/web` |
| Amazon S3 | `eino-ext/components/document/loader/s3` |
| PDF | `eino-ext/components/document/loader/pdf` |

#### Embedding 实现

| 提供商 | 实现 |
|--------|------|
| OpenAI | `eino-ext/components/embedding/openai` |
| 字节火山引擎 (Ark) | `eino-ext/components/embedding/ark` |
| BGE | `eino-ext/components/embedding/bge` |

### 8.2 回调处理器实现

#### Langfuse

Langfuse 是一个开源的 LLM 追踪平台。

```go
import "github.com/cloudwego/eino-ext/callbacks/langfuse"

// 创建 Langfuse 回调处理器
handler, _ := langfuse.NewHandler(&langfuse.Config{
    PublicKey:  langfusePublicKey,
    SecretKey:  langfuseSecretKey,
    BaseURL:    langfuseBaseURL,
})

// 使用回调
callbacks.AppendGlobalHandlers(handler)
```

#### LangSmith

LangSmith 是 LangChain 的追踪平台。

```go
import "github.com/cloudwego/eino-ext/callbacks/langsmith"

// 创建 LangSmith 回调处理器
handler, _ := langsmith.NewHandler(&langsmith.Config{
    APIKey:     langsmithAPIKey,
    ProjectID:  projectID,
})

callbacks.AppendGlobalHandlers(handler)
```

#### APMPlus

APMPlus 是字节跳动的性能监控平台。

```go
import "github.com/cloudwego/eino-ext/callbacks/apmplus"

// 创建 APMPlus 回调处理器
handler, _ := apmplus.NewHandler(&apmplus.Config{
    ServiceName: "my-ai-service",
})

callbacks.AppendGlobalHandlers(handler)
```

#### CozeLoop

CozeLoop 是字节跳动的 AI 应用追踪平台。

```go
import "github.com/cloudwego/eino-ext/callbacks/cozeloop"

// 创建 CozeLoop 客户端
client, _ := cozeloop.NewClient(
    cozeloop.WithAPIToken(apiToken),
    cozeloop.WithWorkspaceID(workspaceID),
)

// 创建回调处理器
handler := cozeloop.NewLoopHandler(client)

callbacks.AppendGlobalHandlers(handler)
```

---

## 9. 开发工具

### 9.1 Eino DevOps

Eino DevOps 是可视化开发和调试工具。

#### 主要功能

1. **可视化图编辑**
   - 拖拽式创建节点和连接
   - 实时预览图结构
   - 自动生成代码

2. **可视化调试**
   - 单步执行
   - 查看中间状态
   - 性能分析

3. **配置管理**
   - 组件配置可视化
   - 环境变量管理
   - 版本控制

#### 使用方式

```go
import "github.com/cloudwego/eino-ext/devops"

// 启用 DevOps 服务
devops.Serve(&devops.Config{
    Port:      8080,
    EnableUI:  true,
})
```

### 9.2 追踪工具

通过回调处理器集成第三方追踪工具：

- **Langfuse**：开源追踪平台
- **LangSmith**：LangChain 官方追踪
- **APMPlus**：性能监控
- **CozeLoop**：字节内部追踪平台

### 9.3 评估工具

```go
import "github.com/cloudwego/eino-ext/devops/evaluator"

// 创建评估器
evaluator := evaluator.New(&evaluator.Config{
    TestSet:   testData,
    Metrics:   []string{"accuracy", "relevance"},
})

// 运行评估
results, _ := evaluator.Evaluate(ctx, runnable)
```

---

## 10. 快速开始

### 10.1 安装

```bash
# 安装核心框架
go get github.com/cloudwego/eino

# 安装扩展组件
go get github.com/cloudwego/eino-ext
```

### 10.2 简单示例：调用 ChatModel

```go
package main

import (
    "context"
    "fmt"
    "log"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/schema"
)

func main() {
    ctx := context.Background()
    
    // 创建 ChatModel
    model, err := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        APIKey: "your-api-key",
        Model:  "gpt-4",
    })
    if err != nil {
        log.Fatal(err)
    }
    
    // 调用模型
    message, err := model.Generate(ctx, []*schema.Message{
        schema.SystemMessage("你是一个有帮助的助手。"),
        schema.UserMessage("什么是人工智能？"),
    })
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println(message.Content)
}
```

### 10.3 Chain 示例

```go
package main

import (
    "context"
    "log"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/components/prompt"
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/schema"
)

func main() {
    ctx := context.Background()
    
    // 创建模型
    model, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        APIKey: "your-api-key",
        Model:  "gpt-4",
    })
    
    // 创建模板
    template := prompt.FromMessages(
        schema.FString,
        schema.SystemMessage("你是一个{role}。"),
        schema.UserMessage("{input}"),
    )
    
    // 创建 Chain
    chain := compose.NewChain[map[string]any, *schema.Message]()
    chain.
        AppendChatTemplate(template).
        AppendChatModel(model)
    
    // 编译
    runnable, _ := chain.Compile(ctx)
    
    // 执行
    output, _ := runnable.Invoke(ctx, map[string]any{
        "role":  "诗人",
        "input": "写一首关于春天的诗",
    })
    
    log.Println(output.Content)
}
```

### 10.4 ReAct Agent 示例

```go
package main

import (
    "context"
    "log"

    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino/components/tool"
    "github.com/cloudwego/eino/flow/agent/react"
    "github.com/cloudwego/eino/schema"
)

// 定义天气工具
type WeatherTool struct{}

func (t *WeatherTool) Info(ctx context.Context) (*schema.ToolInfo, error) {
    return &schema.ToolInfo{
        Name: "get_weather",
        Desc: "获取指定城市的天气",
        ParamsOneOf: schema.NewParamsOneOfByParams(map[string]*schema.ParameterInfo{
            "city": {
                Type:     "string",
                Desc:     "城市名称",
                Required: true,
            },
        }),
    }, nil
}

func (t *WeatherTool) InvokableRun(ctx context.Context, argumentsInJSON string, opts ...tool.Option) (string, error) {
    // 模拟获取天气
    return "北京今天晴，气温 20-30°C", nil
}

func main() {
    ctx := context.Background()
    
    // 创建模型
    model, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        APIKey: "your-api-key",
        Model:  "gpt-4",
    })
    
    // 创建工具
    tools := []tool.InvokableTool{
        &WeatherTool{},
    }
    
    // 创建 ReAct Agent
    agent, _ := react.NewReActAgent(ctx, &react.Config{
        Model:    model,
        Tools:    tools,
        MaxSteps: 10,
    })
    
    // 使用 Agent
    output, _ := agent.Invoke(ctx, []*schema.Message{
        schema.UserMessage("北京今天天气怎么样？"),
    })
    
    log.Println(output.Content)
}
```

### 10.5 RAG 示例

```go
package main

import (
    "context"
    "log"

    "github.com/cloudwego/eino-ext/components/embedding/openai"
    "github.com/cloudwego/eino-ext/components/indexer/elasticsearch"
    "github.com/cloudwego/eino-ext/components/model/openai"
    "github.com/cloudwego/eino-ext/components/retriever/elasticsearch"
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/schema"
)

func main() {
    ctx := context.Background()
    
    // 创建 Embedding
    embedder, _ := openai.NewEmbedding(ctx, &openai.EmbeddingConfig{
        APIKey: "your-api-key",
        Model:  "text-embedding-ada-002",
    })
    
    // 创建 Retriever
    retriever, _ := elasticsearch.NewRetriever(ctx, &elasticsearch.Config{
        Addresses: []string{"http://localhost:9200"},
        Index:     "knowledge_base",
        Embedding: embedder,
    })
    
    // 创建 ChatModel
    model, _ := openai.NewChatModel(ctx, &openai.ChatModelConfig{
        APIKey: "your-api-key",
        Model:  "gpt-4",
    })
    
    // 创建 RAG Chain
    ragChain := compose.NewChain[string, *schema.Message]()
    
    // 添加检索节点
    ragChain.AppendLambda(compose.InvokableLambda(
        func(ctx context.Context, query string) (string, error) {
            // 检索相关文档
            docs, _ := retriever.Retrieve(ctx, query)
            
            // 构建上下文
            context := ""
            for _, doc := range docs {
                context += doc.Content + "\n"
            }
            
            return context, nil
        },
    ))
    
    // 添加模型节点
    ragChain.AppendLambda(compose.InvokableLambda(
        func(ctx context.Context, context string) ([]*schema.Message, error) {
            return []*schema.Message{
                schema.SystemMessage("基于以下上下文回答问题：\n" + context),
                schema.UserMessage(ctx.Value("query").(string)),
            }, nil
        },
    ))
    
    ragChain.AppendChatModel(model)
    
    // 编译和使用
    runnable, _ := ragChain.Compile(ctx)
    output, _ := runnable.Invoke(ctx, "什么是 Eino？")
    
    log.Println(output.Content)
}
```

---

## 11. 最佳实践

### 11.1 组件选择

#### 选择合适的 ChatModel

- **GPT-4**：复杂推理任务，需要高质量输出
- **GPT-3.5**：一般任务，成本敏感
- **Claude**：长文本处理，需要更安全的输出
- **本地模型（Ollama）**：数据敏感，需要本地部署

#### 选择合适的编排方式

- **Chain**：简单的顺序流程
- **Graph**：需要条件分支或循环
- **Workflow**：复杂的数据映射

### 11.2 性能优化

#### 1. 使用流式处理

```go
// ❌ 不推荐：等待完整响应
output, _ := runnable.Invoke(ctx, input)

// ✅ 推荐：使用流式处理
stream, _ := runnable.Stream(ctx, input)
for {
    chunk, err := stream.Recv()
    if err == io.EOF {
        break
    }
    // 立即处理 chunk
    processChunk(chunk)
}
```

#### 2. 并行执行

```go
// 使用 Parallel 并行执行独立任务
parallel := compose.NewParallel()
parallel.
    AddLambda("task1", task1).
    AddLambda("task2", task2).
    AddLambda("task3", task3)

chain.AppendParallel(parallel)
```

#### 3. 缓存策略

```go
// 实现缓存中间件
type CachedModel struct {
    model model.BaseChatModel
    cache map[string]*schema.Message
}

func (m *CachedModel) Generate(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.Message, error) {
    key := hashMessages(input)
    if cached, ok := m.cache[key]; ok {
        return cached, nil
    }
    
    output, err := m.model.Generate(ctx, input, opts...)
    if err == nil {
        m.cache[key] = output
    }
    return output, err
}
```

### 11.3 错误处理

#### 1. 重试机制

```go
// 使用重试装饰器
type RetryableModel struct {
    model      model.BaseChatModel
    maxRetries int
}

func (m *RetryableModel) Generate(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.Message, error) {
    var lastErr error
    for i := 0; i < m.maxRetries; i++ {
        output, err := m.model.Generate(ctx, input, opts...)
        if err == nil {
            return output, nil
        }
        lastErr = err
        time.Sleep(time.Second * time.Duration(i+1))
    }
    return nil, lastErr
}
```

#### 2. 优雅降级

```go
// 主模型失败时使用备用模型
func GenerateWithFallback(ctx context.Context, primary, fallback model.BaseChatModel, input []*schema.Message) (*schema.Message, error) {
    output, err := primary.Generate(ctx, input)
    if err != nil {
        log.Printf("主模型失败: %v，使用备用模型", err)
        return fallback.Generate(ctx, input)
    }
    return output, nil
}
```

### 11.4 安全性

#### 1. API Key 管理

```go
// ❌ 不推荐：硬编码 API Key
apiKey := "sk-xxxxx"

// ✅ 推荐：从环境变量读取
apiKey := os.Getenv("OPENAI_API_KEY")
if apiKey == "" {
    log.Fatal("OPENAI_API_KEY 未设置")
}
```

#### 2. 输入验证

```go
// 验证用户输入
func ValidateInput(input string) error {
    if len(input) > 10000 {
        return errors.New("输入过长")
    }
    if containsMalicious(input) {
        return errors.New("输入包含非法内容")
    }
    return nil
}
```

#### 3. 输出过滤

```go
// 过滤敏感输出
func FilterOutput(output string) string {
    // 移除敏感信息
    filtered := removePII(output)
    filtered = removeOffensiveContent(filtered)
    return filtered
}
```

### 11.5 测试

#### 1. Mock 组件

```go
// Mock ChatModel 用于测试
type MockChatModel struct {
    Response *schema.Message
    Err      error
}

func (m *MockChatModel) Generate(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.Message, error) {
    return m.Response, m.Err
}

func (m *MockChatModel) Stream(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.StreamReader[*schema.Message], error) {
    // Mock 实现
    return nil, nil
}

// 测试中使用
func TestMyChain(t *testing.T) {
    mockModel := &MockChatModel{
        Response: schema.AssistantMessage("测试响应"),
    }
    
    chain := compose.NewChain[string, *schema.Message]()
    chain.AppendChatModel(mockModel)
    
    runnable, _ := chain.Compile(context.Background())
    output, err := runnable.Invoke(context.Background(), "测试输入")
    
    assert.NoError(t, err)
    assert.Equal(t, "测试响应", output.Content)
}
```

#### 2. 集成测试

```go
func TestIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("跳过集成测试")
    }
    
    // 使用真实组件进行集成测试
    model, _ := openai.NewChatModel(context.Background(), config)
    // ... 测试完整流程
}
```

### 11.6 监控和可观测性

#### 1. 日志记录

```go
// 使用结构化日志
handler := callbacks.NewHandlerBuilder().
    OnStartFn(func(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
        log.WithFields(log.Fields{
            "component": info.Component,
            "name":      info.Name,
            "input":     input,
        }).Info("组件开始执行")
        return ctx
    }).
    Build()
```

#### 2. 指标收集

```go
// 收集执行时间指标
handler := callbacks.NewHandlerBuilder().
    OnStartFn(func(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
        startTime := time.Now()
        return context.WithValue(ctx, "start_time", startTime)
    }).
    OnEndFn(func(ctx context.Context, info *RunInfo, output CallbackOutput) context.Context {
        startTime := ctx.Value("start_time").(time.Time)
        duration := time.Since(startTime)
        metrics.RecordDuration(info.Component, duration)
        return ctx
    }).
    Build()
```

#### 3. 分布式追踪

```go
// 集成 OpenTelemetry
import "go.opentelemetry.io/otel"

handler := callbacks.NewHandlerBuilder().
    OnStartFn(func(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
        tracer := otel.Tracer("eino")
        ctx, span := tracer.Start(ctx, info.Name)
        return context.WithValue(ctx, "span", span)
    }).
    OnEndFn(func(ctx context.Context, info *RunInfo, output CallbackOutput) context.Context {
        span := ctx.Value("span").(trace.Span)
        span.End()
        return ctx
    }).
    Build()
```

---

## 12. 架构设计

### 12.1 整体架构

Eino 框架采用分层架构：

```
┌─────────────────────────────────────────┐
│         应用层 (Applications)            │
│  (ReAct Agent, RAG, Multi-Agent...)     │
├─────────────────────────────────────────┤
│       Flow 层 (Prebuilt Flows)          │
│  (Agent Patterns, Retrieval Flows...)   │
├─────────────────────────────────────────┤
│        编排层 (Orchestration)           │
│  (Chain, Graph, Workflow)               │
├─────────────────────────────────────────┤
│        组件层 (Components)               │
│  (ChatModel, Tool, Retriever...)        │
├─────────────────────────────────────────┤
│         基础层 (Foundation)              │
│  (Schema, Stream, Callbacks...)         │
└─────────────────────────────────────────┘
```

### 12.2 核心模块

#### 1. Schema 模块

定义核心数据类型：
- `Message`：消息类型
- `Document`：文档类型
- `ToolInfo`：工具信息
- `StreamReader`：流读取器

#### 2. Components 模块

定义组件接口：
- `model`：ChatModel 接口
- `tool`：Tool 接口
- `prompt`：ChatTemplate 接口
- `retriever`：Retriever 接口
- 等等

#### 3. Compose 模块

编排能力实现：
- `Chain`：链式编排
- `Graph`：图编排
- `Workflow`：工作流编排
- 流处理机制
- 节点管理

#### 4. Callbacks 模块

回调机制实现：
- `Handler` 接口
- `HandlerBuilder`
- 切面注入
- 全局回调

#### 5. Flow 模块

预构建流程：
- `react`：ReAct Agent
- `multiagent`：多智能体
- `retriever`：检索流程

#### 6. ADK 模块

高级开发工具：
- Agent 开发工具
- 预构建智能体模式
- Agent 工具封装

### 12.3 扩展机制

Eino 提供多种扩展机制：

#### 1. 自定义组件

实现组件接口即可创建自定义组件：

```go
type MyCustomModel struct {
    // 自定义字段
}

func (m *MyCustomModel) Generate(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.Message, error) {
    // 自定义实现
}

func (m *MyCustomModel) Stream(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.StreamReader[*schema.Message], error) {
    // 自定义实现
}
```

#### 2. 组件装饰器

使用装饰器模式扩展组件功能：

```go
type LoggingModel struct {
    inner model.BaseChatModel
}

func (m *LoggingModel) Generate(ctx context.Context, input []*schema.Message, opts ...model.Option) (*schema.Message, error) {
    log.Printf("调用模型: %v", input)
    output, err := m.inner.Generate(ctx, input, opts...)
    log.Printf("模型响应: %v", output)
    return output, err
}
```

#### 3. 自定义回调

实现 Handler 接口创建自定义回调：

```go
type MyHandler struct {
    // 自定义字段
}

func (h *MyHandler) OnStart(ctx context.Context, info *RunInfo, input CallbackInput) context.Context {
    // 自定义逻辑
    return ctx
}

// 实现其他回调方法...
```

### 12.4 依赖关系

```
eino (核心框架)
  ↑
  │ 依赖
  │
eino-ext (扩展实现)
  ↑
  │ 依赖
  │
eino-examples (示例代码)
```

### 12.5 并发模型

Eino 采用以下并发策略：

1. **无状态组件**：组件实例本身是无状态的，可以安全并发调用
2. **状态隔离**：每次调用使用独立的 context 和参数
3. **流式安全**：StreamReader 是线程安全的
4. **状态管理**：Graph 的 State 使用锁保护

### 12.6 性能考虑

#### 1. 零拷贝设计

- StreamReader 避免不必要的数据拷贝
- Message 使用指针传递

#### 2. 延迟初始化

- 组件在首次使用时初始化
- 连接池延迟创建

#### 3. 资源池化

```go
// 连接池示例
type ModelPool struct {
    pool chan model.BaseChatModel
}

func (p *ModelPool) Get() model.BaseChatModel {
    return <-p.pool
}

func (p *ModelPool) Put(m model.BaseChatModel) {
    p.pool <- m
}
```

---

## 附录

### A. 常见问题

#### Q1: Eino 与 LangChain 的区别？

**A:** Eino 是专为 Go 语言设计的，更符合 Go 的编程习惯：
- 更强的类型安全（利用泛型）
- 更好的并发支持
- 更清晰的 API 设计
- 更注重性能

#### Q2: 如何选择编排方式？

**A:** 
- 简单顺序流程用 **Chain**
- 需要条件分支或循环用 **Graph**
- 复杂数据映射用 **Workflow**

#### Q3: 流式处理是必须的吗？

**A:** 不是必须的，但强烈推荐：
- 提升用户体验（实时响应）
- 降低延迟感知
- 更高效的资源利用

#### Q4: 如何调试 Eino 应用？

**A:** 
1. 使用回调记录日志
2. 使用 Eino DevOps 可视化调试
3. 集成追踪工具（Langfuse、LangSmith）
4. 单元测试使用 Mock 组件

#### Q5: 支持哪些 LLM 提供商？

**A:** Eino 通过 EinoExt 支持：
- OpenAI (GPT-4, GPT-3.5)
- Anthropic (Claude)
- Google (Gemini)
- 字节火山引擎 (Ark)
- Ollama (本地部署)
- 其他兼容 OpenAI API 的提供商

### B. 术语表

| 术语 | 说明 |
|------|------|
| Component | 组件，Eino 的基本构建块 |
| Orchestration | 编排，将组件连接成数据流 |
| Node | 节点，图中的组件实例 |
| Edge | 边，连接节点的数据流通道 |
| Branch | 分支，条件路由机制 |
| State | 状态，图执行中的共享数据 |
| Stream | 流，流式数据处理 |
| Callback | 回调，切面处理机制 |
| Runnable | 可运行对象，编译后的图 |
| Lambda | 自定义函数节点 |

### C. 参考资源

#### 官方文档
- [Eino 用户手册](https://www.cloudwego.io/zh/docs/eino/)
- [Eino 快速开始](https://www.cloudwego.io/zh/docs/eino/quick_start/)
- [Eino GitHub](https://github.com/cloudwego/eino)

#### 扩展项目
- [Eino-Ext GitHub](https://github.com/cloudwego/eino-ext)
- [Eino-Examples GitHub](https://github.com/cloudwego/eino-examples)

#### 社区
- [CloudWeGo 社区](https://github.com/cloudwego/community)
- [Issues](https://github.com/cloudwego/eino/issues)
- 飞书用户群（见 README）

### D. 版本历史

Eino 遵循语义化版本规范。

**当前版本要求**：
- Go 1.18+
- kin-openapi v0.118.0（固定版本，兼容 Go 1.18）

### E. 贡献指南

欢迎为 Eino 贡献代码！

#### 贡献流程

1. Fork 项目
2. 创建特性分支
3. 提交代码
4. 创建 Pull Request

#### 代码规范

- 遵循 Go 编码规范
- 添加充分的测试
- 更新相关文档
- 使用有意义的提交信息

详见 [CONTRIBUTING.md](https://github.com/cloudwego/eino/blob/main/CONTRIBUTING.md)

---

## 结语

Eino 是一个强大、灵活且易用的 LLM 应用开发框架。通过其丰富的组件系统、强大的编排能力和完善的工具链，开发者可以快速构建生产级的 AI 应用。

希望这份文档能帮助你更好地理解和使用 Eino 框架。如有问题，欢迎在 GitHub 上提 Issue 或加入社区讨论。

**Happy Coding with Eino! 🚀**

---

*本文档由 CloudWeGo Eino 社区维护*  
*最后更新：2025年11月*

