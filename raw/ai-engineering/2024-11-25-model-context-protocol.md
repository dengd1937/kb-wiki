# Model Context Protocol 技术文档（Model Context Protocol Documentation）

> Source: https://modelcontextprotocol.io/docs/getting-started/intro
> Collected: 2026-05-21
> Published: 2024-11-25
> Full text: https://modelcontextprotocol.io/docs/getting-started/intro (+ architecture, transports, resources, prompts, tools, sampling)

## 文章信息

- **作者**：MCP Project / Anthropic
- **载体**：MCP 官方文档站
- **发布日期**：2024-11-25
- **性质**：官方技术文档

---

## MCP 简介

MCP（Model Context Protocol）是一种开源标准，用于将 AI 应用连接到外部系统。通过 MCP，Claude 或 ChatGPT 等 AI 应用可以连接到数据源（例如本地文件、数据库）、工具（例如搜索引擎、计算器）和工作流（例如专门的提示词）——使它们能够访问关键信息并执行任务。

可以把 MCP 想象成 AI 应用的 USB-C 接口。正如 USB-C 提供了一种标准化的方式来连接电子设备，MCP 提供了一种标准化的方式来将 AI 应用连接到外部系统。

### MCP 能实现什么

- Agent 可以访问你的 Google 日历和 Notion，充当更加个性化的 AI 助手。
- Claude Code 可以使用 Figma 设计稿生成完整的 Web 应用。
- 企业聊天机器人可以连接到组织内的多个数据库，让用户通过聊天分析数据。
- AI 模型可以在 Blender 上创建 3D 设计并使用 3D 打印机打印出来。

### 为什么 MCP 很重要

根据你在生态系统中的位置不同，MCP 可以带来一系列好处：

- **开发者**：MCP 减少了构建或集成 AI 应用或 Agent 时的开发时间和复杂度。
- **AI 应用或 Agent**：MCP 提供了对数据源、工具和应用生态系统的访问，从而增强功能并改善终端用户体验。
- **终端用户**：MCP 使更强大的 AI 应用或 Agent 成为可能，它们可以访问你的数据并在必要时代表你执行操作。

### 广泛的生态支持

MCP 是一个开放协议，受到广泛的客户端和服务器支持。Claude 和 ChatGPT 等 AI 助手，Visual Studio Code、Cursor、MCPJam 等开发工具以及许多其他工具都支持 MCP——使构建一次、到处集成成为可能。

## 核心架构

Model Context Protocol (MCP) 建立在灵活、可扩展的架构之上，实现 LLM 应用与集成之间的无缝通信。本文档介绍核心架构组件和概念。

### 概述

MCP 遵循客户端-服务器架构，其中：

- **Host（宿主）** 是 LLM 应用（如 Claude Desktop 或 IDE），负责发起连接
- **Client（客户端）** 在宿主应用内部维护与服务器的 1:1 连接
- **Server（服务器）** 向客户端提供上下文、工具和提示词

### 核心组件

#### 协议层

协议层处理消息组帧、请求/响应链接以及高层通信模式。

```
class Protocol<Request, Notification, Result> {
    // 处理传入请求
    setRequestHandler<T>(schema: T, handler: (request: T, extra: RequestHandlerExtra) => Promise<Result>): void

    // 处理传入通知
    setNotificationHandler<T>(schema: T, handler: (notification: T) => Promise<void>): void

    // 发送请求并等待响应
    request<T>(request: Request, schema: T, options?: RequestOptions): Promise<T>

    // 发送单向通知
    notification(notification: Notification): Promise<void>
}
```

关键类包括：

- `Protocol`
- `Client`
- `Server`

#### 传输层

传输层处理客户端和服务器之间的实际通信。MCP 支持多种传输机制：

1. **Stdio 传输**

   - 使用标准输入/输出进行通信
   - 适用于本地进程
2. **Streamable HTTP 传输**

   - 使用 HTTP 并可选 Server-Sent Events 进行流式传输
   - 通过 HTTP POST 发送客户端到服务器的消息

所有传输都使用 JSON-RPC 2.0 来交换消息。有关 Model Context Protocol 消息格式的详细信息，请参阅规范。

#### 消息类型

MCP 有以下主要消息类型：

1. **Requests（请求）** 期望对方的响应：

   ```
   interface Request {
     method: string;
     params?: { ... };
   }
   ```
2. **Results（结果）** 是请求的成功响应：

   ```
   interface Result {
     [key: string]: unknown;
   }
   ```
3. **Errors（错误）** 表示请求失败：

   ```
   interface Error {
     code: number;
     message: string;
     data?: unknown;
   }
   ```
4. **Notifications（通知）** 是不期望响应的单向消息：

   ```
   interface Notification {
     method: string;
     params?: { ... };
   }
   ```

### 连接生命周期

#### 1. 初始化

1. 客户端发送 `initialize` 请求，附带协议版本和能力
2. 服务器响应其协议版本和能力
3. 客户端发送 `initialized` 通知作为确认
4. 正常消息交换开始

#### 2. 消息交换

初始化完成后，支持以下模式：

- **请求-响应**：客户端或服务器发送请求，另一方响应
- **通知**：任一方发送单向消息

#### 3. 终止

任一方都可以终止连接：

- 通过 `close()` 干净地关闭
- 传输断开连接
- 错误条件

### 错误处理

MCP 定义了以下标准错误码：

```
enum ErrorCode {
  // 标准 JSON-RPC 错误码
  ParseError = -32700,
  InvalidRequest = -32600,
  MethodNotFound = -32601,
  InvalidParams = -32602,
  InternalError = -32603,
}
```

SDK 和应用程序可以定义 -32000 以上的自定义错误码。

错误通过以下方式传播：

- 请求的错误响应
- 传输上的错误事件
- 协议级错误处理器

### 实现示例

以下是实现 MCP 服务器的基本示例：

```
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    resources: {}
  }
});

// 处理请求
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "example://resource",
        name: "Example Resource"
      }
    ]
  };
});

// 连接传输
const transport = new StdioServerTransport();
await server.connect(transport);
```

### 最佳实践

#### 传输选择

1. **本地通信**

   - 对本地进程使用 stdio 传输
   - 对同机通信效率高
   - 简单的进程管理
2. **远程通信**

   - 在需要 HTTP 兼容性的场景下使用 Streamable HTTP
   - 考虑安全影响，包括认证和授权

#### 消息处理

1. **请求处理**

   - 彻底验证输入
   - 使用类型安全的 schema
   - 优雅地处理错误
   - 实现超时
2. **进度报告**

   - 对长时间操作使用进度令牌
   - 增量报告进度
   - 在已知情况下包含总进度
3. **错误管理**

   - 使用适当的错误码
   - 包含有用的错误消息
   - 错误时清理资源

### 安全考虑

1. **传输安全**

   - 远程连接使用 TLS
   - 验证连接来源
   - 需要时实现认证
2. **消息验证**

   - 验证所有传入消息
   - 清理输入
   - 检查消息大小限制
   - 验证 JSON-RPC 格式
3. **资源保护**

   - 实现访问控制
   - 验证资源路径
   - 监控资源使用
   - 对请求进行速率限制
4. **错误处理**

   - 不要泄露敏感信息
   - 记录安全相关的错误
   - 实现适当的清理
   - 处理 DoS 场景

### 调试和监控

1. **日志**

   - 记录协议事件
   - 跟踪消息流
   - 监控性能
   - 记录错误
2. **诊断**

   - 实现健康检查
   - 监控连接状态
   - 跟踪资源使用
   - 分析性能
3. **测试**

   - 测试不同传输
   - 验证错误处理
   - 检查边缘情况
   - 对服务器进行负载测试

## 传输层

Model Context Protocol (MCP) 中的传输层为客户端和服务器之间的通信提供了基础。传输层处理消息发送和接收的底层机制。

### 消息格式

MCP 使用 JSON-RPC 2.0 作为线路格式。传输层负责将 MCP 协议消息转换为 JSON-RPC 格式进行传输，并将接收到的 JSON-RPC 消息转换回 MCP 协议消息。

使用的 JSON-RPC 消息有三种类型：

#### 请求

```
{
  jsonrpc: "2.0",
  id: number | string,
  method: string,
  params?: object
}
```

#### 响应

```
{
  jsonrpc: "2.0",
  id: number | string,
  result?: object,
  error?: {
    code: number,
    message: string,
    data?: unknown
  }
}
```

#### 通知

```
{
  jsonrpc: "2.0",
  method: string,
  params?: object
}
```

### 内置传输类型

MCP 目前定义了两种标准传输机制：

#### 标准输入/输出（stdio）

stdio 传输通过标准输入和输出流实现通信。这对于本地集成和命令行工具特别有用。

在以下情况下使用 stdio：

- 构建命令行工具
- 实现本地集成
- 需要简单的进程通信
- 使用 shell 脚本工作

```
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

#### Streamable HTTP

Streamable HTTP 传输使用 HTTP POST 请求进行客户端到服务器的通信，并可选使用 Server-Sent Events (SSE) 流进行服务器到客户端的通信。

在以下情况下使用 Streamable HTTP：

- 构建 Web 集成
- 需要 HTTP 上的客户端-服务器通信
- 需要有状态会话
- 支持多个并发客户端
- 实现可恢复连接

##### 工作原理

1. **客户端到服务器通信**：客户端到服务器的每条 JSON-RPC 消息都作为新的 HTTP POST 请求发送到 MCP 端点
2. **服务器响应**：服务器可以通过以下方式响应：
   - 单个 JSON 响应（`Content-Type: application/json`）
   - SSE 流（`Content-Type: text/event-stream`）用于多条消息
3. **服务器到客户端通信**：服务器可以通过以下方式向客户端发送请求/通知：
   - 由客户端请求发起的 SSE 流
   - 通过 HTTP GET 请求到 MCP 端点的 SSE 流

```
import express from "express";

const app = express();

const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

// MCP 端点同时处理 POST 和 GET
app.post("/mcp", async (req, res) => {
  // 处理 JSON-RPC 请求
  const response = await server.handleRequest(req.body);

  // 返回单个响应或 SSE 流
  if (needsStreaming) {
    res.setHeader("Content-Type", "text/event-stream");
    // 发送 SSE 事件...
  } else {
    res.json(response);
  }
});

app.get("/mcp", (req, res) => {
  // 可选：支持服务器发起的 SSE 流
  res.setHeader("Content-Type", "text/event-stream");
  // 发送服务器通知/请求...
});

app.listen(3000);
```

##### 会话管理

Streamable HTTP 支持有状态会话，以在多个请求之间维护上下文：

1. **会话初始化**：服务器可以在初始化期间通过在 `Mcp-Session-Id` 头中包含会话 ID 来分配会话 ID
2. **会话持久化**：客户端必须在所有后续请求中使用 `Mcp-Session-Id` 头包含会话 ID
3. **会话终止**：可以通过发送带有会话 ID 的 HTTP DELETE 请求显式终止会话

会话流程示例：

```
// 服务器在初始化期间分配会话 ID
app.post("/mcp", (req, res) => {
  if (req.body.method === "initialize") {
    const sessionId = generateSecureId();
    res.setHeader("Mcp-Session-Id", sessionId);
    // 存储会话状态...
  }
  // 处理请求...
});

// 客户端在后续请求中包含会话 ID
fetch("/mcp", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Mcp-Session-Id": sessionId,
  },
  body: JSON.stringify(request),
});
```

##### 可恢复性和重传

为了支持恢复断开的连接，Streamable HTTP 提供：

1. **事件 ID**：服务器可以为 SSE 事件附加唯一 ID 用于跟踪
2. **从最后一个事件恢复**：客户端可以通过发送 `Last-Event-ID` 头来恢复
3. **消息重放**：服务器可以从断开点重放丢失的消息

这确保了即使在网络连接不稳定的情况下也能可靠地传递消息。

##### 安全考虑

在实现 Streamable HTTP 传输时，请遵循以下安全最佳实践：

1. **验证 Origin 头**：始终验证所有传入连接的 `Origin` 头以防止 DNS 重绑定攻击
2. **绑定到 localhost**：在本地运行时，仅绑定到 localhost（127.0.0.1）而不是所有网络接口（0.0.0.0）
3. **实现认证**：对所有连接使用适当的认证
4. **使用 HTTPS**：在生产部署中始终使用 TLS/HTTPS
5. **验证会话 ID**：确保会话 ID 具有密码学安全性并经过正确验证

没有这些保护措施，攻击者可能利用 DNS 重绑定从远程网站与本地 MCP 服务器交互。

#### Server-Sent Events (SSE) —— 已弃用

旧的 SSE 传输通过 HTTP POST 请求实现客户端到服务器的通信，同时实现服务器到客户端的流式传输。

以前在以下情况下使用：

- 仅需要服务器到客户端的流式传输
- 在受限网络中工作
- 实现简单更新

##### 旧版安全考虑

已弃用的 SSE 传输与 Streamable HTTP 有类似的安全考虑，特别是关于 DNS 重绑定攻击。在 Streamable HTTP 传输中使用 SSE 流时，也应应用这些相同的保护措施。

```
import express from "express";

const app = express();

const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

let transport: SSEServerTransport | null = null;

app.get("/sse", (req, res) => {
  transport = new SSEServerTransport("/messages", res);
  server.connect(transport);
});

app.post("/messages", (req, res) => {
  if (transport) {
    transport.handlePostMessage(req, res);
  }
});

app.listen(3000);
```

### 自定义传输

MCP 使得为特定需求实现自定义传输变得简单。任何传输实现只需要符合 Transport 接口：

你可以为以下场景实现自定义传输：

- 自定义网络协议
- 专门的通信通道
- 与现有系统的集成
- 性能优化

```
interface Transport {
  // 开始处理消息
  start(): Promise<void>;

  // 发送 JSON-RPC 消息
  send(message: JSONRPCMessage): Promise<void>;

  // 关闭连接
  close(): Promise<void>;

  // 回调
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
}
```

### 错误处理

传输实现应该处理各种错误场景：

1. 连接错误
2. 消息解析错误
3. 协议错误
4. 网络超时
5. 资源清理

错误处理示例：

```
class ExampleTransport implements Transport {
  async start() {
    try {
      // 连接逻辑
    } catch (error) {
      this.onerror?.(new Error(`Failed to connect: ${error}`));
      throw error;
    }
  }

  async send(message: JSONRPCMessage) {
    try {
      // 发送逻辑
    } catch (error) {
      this.onerror?.(new Error(`Failed to send message: ${error}`));
      throw error;
    }
  }
}
```

### 最佳实践

在实现或使用 MCP 传输时：

1. 正确处理连接生命周期
2. 实现适当的错误处理
3. 连接关闭时清理资源
4. 使用适当的超时
5. 发送前验证消息
6. 记录传输事件以便调试
7. 适当时实现重连逻辑
8. 处理消息队列中的背压
9. 监控连接健康
10. 实现适当的安全措施

### 安全考虑

在实现传输时：

- 实现适当的认证机制
- 验证客户端凭据
- 使用安全的令牌处理
- 实现授权检查

#### 数据安全

- 网络传输使用 TLS
- 加密敏感数据
- 验证消息完整性
- 实现消息大小限制
- 清理输入数据

#### 网络安全

- 实现速率限制
- 使用适当的超时
- 处理拒绝服务场景
- 监控异常模式
- 实现适当的防火墙规则
- 对于基于 HTTP 的传输（包括 Streamable HTTP），验证 Origin 头以防止 DNS 重绑定攻击
- 对于本地服务器，仅绑定到 localhost（127.0.0.1）而不是所有接口（0.0.0.0）

### 调试传输

调试传输问题的提示：

1. 启用调试日志
2. 监控消息流
3. 检查连接状态
4. 验证消息格式
5. 测试错误场景
6. 使用网络分析工具
7. 实现健康检查
8. 监控资源使用
9. 测试边缘情况
10. 使用适当的错误跟踪

### 向后兼容性

为了在不同协议版本之间保持兼容性：

#### 支持旧客户端的服务器

想要支持使用已弃用 HTTP+SSE 传输的客户端的服务器应该：

1. 在新的 MCP 端点旁边托管旧的 SSE 和 POST 端点
2. 在两个端点上处理初始化请求
3. 为每种传输类型维护独立的处理逻辑

#### 支持旧服务器的客户端

想要支持使用已弃用传输的服务器的客户端应该：

1. 接受可能使用任一传输的服务器 URL
2. 尝试使用适当的 `Accept` 头 POST 一个 `InitializeRequest`：
   - 如果成功，使用 Streamable HTTP 传输
   - 如果以 4xx 状态码失败，回退到旧版 SSE 传输
3. 发出 GET 请求，期望旧版服务器返回带有 `endpoint` 事件的 SSE 流

兼容性检测示例：

```
async function detectTransport(serverUrl: string): Promise<TransportType> {
  try {
    // 首先尝试 Streamable HTTP
    const response = await fetch(serverUrl, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Accept: "application/json, text/event-stream",
      },
      body: JSON.stringify({
        jsonrpc: "2.0",
        method: "initialize",
        params: {
          /* ... */
        },
      }),
    });

    if (response.ok) {
      return "streamable-http";
    }
  } catch (error) {
    // 回退到旧版 SSE
    const sseResponse = await fetch(serverUrl, {
      method: "GET",
      headers: { Accept: "text/event-stream" },
    });

    if (sseResponse.ok) {
      return "legacy-sse";
    }
  }

  throw new Error("Unsupported transport");
}
```

## 资源（Resources）

资源是 Model Context Protocol (MCP) 中的核心原语，允许服务器暴露数据和内容，供客户端读取并用作 LLM 交互的上下文。

### 概述

资源代表 MCP 服务器希望向客户端提供的任何类型的数据。这可以包括：

- 文件内容
- 数据库记录
- API 响应
- 实时系统数据
- 截图和图像
- 日志文件
- 以及更多

每个资源由唯一的 URI 标识，可以包含文本或二进制数据。

### 资源 URI

资源使用遵循以下格式的 URI 进行标识：

```
[protocol]://[host]/[path]
```

例如：

- `file:///home/user/documents/report.pdf`
- `postgres://database/customers/schema`
- `screen://localhost/display1`

协议和路径结构由 MCP 服务器实现定义。服务器可以定义自己的自定义 URI 方案。

### 资源类型

资源可以包含两种类型的内容：

#### 文本资源

文本资源包含 UTF-8 编码的文本数据。这些适用于：

- 源代码
- 配置文件
- 日志文件
- JSON/XML 数据
- 纯文本

#### 二进制资源

二进制资源包含以 base64 编码的原始二进制数据。这些适用于：

- 图像
- PDF
- 音频文件
- 视频文件
- 其他非文本格式

### 资源发现

客户端可以通过两种主要方法发现可用资源：

#### 直接资源

服务器通过 `resources/list` 请求暴露资源列表。每个资源包括：

```
{
  uri: string;           // 资源的唯一标识符
  name: string;          // 人类可读的名称
  description?: string;  // 可选描述
  mimeType?: string;     // 可选 MIME 类型
  size?: number;         // 可选大小（字节）
}
```

#### 资源模板

对于动态资源，服务器可以暴露 URI 模板，客户端可以使用这些模板构建有效的资源 URI：

```
{
  uriTemplate: string;   // 遵循 RFC 6570 的 URI 模板
  name: string;          // 此类型的人类可读名称
  description?: string;  // 可选描述
  mimeType?: string;     // 所有匹配资源的可选 MIME 类型
}
```

### 读取资源

要读取资源，客户端使用资源 URI 发出 `resources/read` 请求。

服务器以资源内容列表响应：

```
{
  contents: [
    {
      uri: string;        // 资源的 URI
      mimeType?: string;  // 可选 MIME 类型

      // 以下之一：
      text?: string;      // 文本资源
      blob?: string;      // 二进制资源（base64 编码）
    }
  ]
}
```

### 资源更新

MCP 通过两种机制支持资源的实时更新：

#### 列表变更

服务器可以通过 `notifications/resources/list_changed` 通知在可用资源列表发生变化时通知客户端。

#### 内容变更

客户端可以订阅特定资源的更新：

1. 客户端发送带有资源 URI 的 `resources/subscribe`
2. 当资源发生变化时，服务器发送 `notifications/resources/updated`
3. 客户端可以通过 `resources/read` 获取最新内容
4. 客户端可以通过 `resources/unsubscribe` 取消订阅

### 实现示例

以下是在 MCP 服务器中实现资源支持的简单示例：

```
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    resources: {}
  }
});

// 列出可用资源
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "file:///logs/app.log",
        name: "Application Logs",
        mimeType: "text/plain"
      }
    ]
  };
});

// 读取资源内容
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;

  if (uri === "file:///logs/app.log") {
    const logContents = await readLogFile();
    return {
      contents: [
        {
          uri,
          mimeType: "text/plain",
          text: logContents
        }
      ]
    };
  }

  throw new Error("Resource not found");
});
```

### 最佳实践

在实现资源支持时：

1. 使用清晰、描述性的资源名称和 URI
2. 包含有帮助的描述以引导 LLM 理解
3. 在已知时设置适当的 MIME 类型
4. 为动态内容实现资源模板
5. 对频繁变化的资源使用订阅
6. 优雅地处理错误并提供清晰的错误消息
7. 考虑对大型资源列表进行分页
8. 在适当时缓存资源内容
9. 在处理前验证 URI
10. 为你的自定义 URI 方案编写文档

### 安全考虑

在暴露资源时：

- 验证所有资源 URI
- 实现适当的访问控制
- 清理文件路径以防止目录遍历
- 谨慎处理二进制数据
- 考虑对资源读取进行速率限制
- 审计资源访问
- 在传输中加密敏感数据
- 验证 MIME 类型
- 为长时间运行的读取实现超时
- 适当地处理资源清理

## 提示（Prompts）

提示词使服务器能够定义可重用的提示模板和工作流，客户端可以轻松地将其呈现给用户和 LLM。它们提供了一种强大的方式来标准化和共享常见的 LLM 交互。

### 概述

MCP 中的提示词是预定义的模板，可以：

- 接受动态参数
- 包含来自资源的上下文
- 链接多个交互
- 引导特定工作流
- 作为 UI 元素呈现（如斜杠命令）

### 提示词结构

每个提示词定义为：

```
{
  name: string;              // 提示词的唯一标识符
  description?: string;      // 人类可读的描述
  arguments?: [              // 可选的参数列表
    {
      name: string;          // 参数标识符
      description?: string;  // 参数描述
      required?: boolean;    // 参数是否必需
    }
  ]
}
```

### 发现提示词

客户端可以通过发送 `prompts/list` 请求来发现可用提示词：

```
// 请求
{
  method: "prompts/list";
}

// 响应
{
  prompts: [
    {
      name: "analyze-code",
      description: "Analyze code for potential improvements",
      arguments: [
        {
          name: "language",
          description: "Programming language",
          required: true,
        },
      ],
    },
  ];
}
```

### 使用提示词

要使用提示词，客户端发出 `prompts/get` 请求：

```
// 请求
{
  method: "prompts/get",
  params: {
    name: "analyze-code",
    arguments: {
      language: "python"
    }
  }
}

// 响应
{
  description: "Analyze Python code for potential improvements",
  messages: [
    {
      role: "user",
      content: {
        type: "text",
        text: "Please analyze the following Python code for potential improvements:\n\n```python\ndef calculate_sum(numbers):\n    total = 0\n    for num in numbers:\n        total = total + num\n    return total\n\nresult = calculate_sum([1, 2, 3, 4, 5])\nprint(result)\n```"
      }
    }
  ]
}
```

### 动态提示词

提示词可以是动态的，并包含：

#### 嵌入资源上下文

```
{
  "name": "analyze-project",
  "description": "Analyze project logs and code",
  "arguments": [
    {
      "name": "timeframe",
      "description": "Time period to analyze logs",
      "required": true
    },
    {
      "name": "fileUri",
      "description": "URI of code file to review",
      "required": true
    }
  ]
}
```

在处理 `prompts/get` 请求时：

```
{
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "Analyze these system logs and the code file for any issues:"
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "logs://recent?timeframe=1h",
          "text": "[2024-03-14 15:32:11] ERROR: Connection timeout in network.py:127\n[2024-03-14 15:32:15] WARN: Retrying connection (attempt 2/3)\n[2024-03-14 15:32:20] ERROR: Max retries exceeded",
          "mimeType": "text/plain"
        }
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "file:///path/to/code.py",
          "text": "def connect_to_service(timeout=30):\n    retries = 3\n    for attempt in range(retries):\n        try:\n            return establish_connection(timeout)\n        except TimeoutError:\n            if attempt == retries - 1:\n                raise\n            time.sleep(5)\n\ndef establish_connection(timeout):\n    # Connection implementation\n    pass",
          "mimeType": "text/x-python"
        }
      }
    }
  ]
}
```

#### 多步工作流

```
const debugWorkflow = {
  name: "debug-error",
  async getMessages(error: string) {
    return [
      {
        role: "user",
        content: {
          type: "text",
          text: `Here's an error I'm seeing: ${error}`,
        },
      },
      {
        role: "assistant",
        content: {
          type: "text",
          text: "I'll help analyze this error. What have you tried so far?",
        },
      },
      {
        role: "user",
        content: {
          type: "text",
          text: "I've tried restarting the service, but the error persists.",
        },
      },
    ];
  },
};
```

### 实现示例

以下是在 MCP 服务器中实现提示词的完整示例：

```
import { Server } from "@modelcontextprotocol/sdk/server";
import {
  ListPromptsRequestSchema,
  GetPromptRequestSchema
} from "@modelcontextprotocol/sdk/types";

const PROMPTS = {
  "git-commit": {
    name: "git-commit",
    description: "Generate a Git commit message",
    arguments: [
      {
        name: "changes",
        description: "Git diff or description of changes",
        required: true
      }
    ]
  },
  "explain-code": {
    name: "explain-code",
    description: "Explain how code works",
    arguments: [
      {
        name: "code",
        description: "Code to explain",
        required: true
      },
      {
        name: "language",
        description: "Programming language",
        required: false
      }
    ]
  }
};

const server = new Server({
  name: "example-prompts-server",
  version: "1.0.0"
}, {
  capabilities: {
    prompts: {}
  }
});

// 列出可用提示词
server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: Object.values(PROMPTS)
  };
});

// 获取特定提示词
server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  const prompt = PROMPTS[request.params.name];
  if (!prompt) {
    throw new Error(`Prompt not found: ${request.params.name}`);
  }

  if (request.params.name === "git-commit") {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Generate a concise but descriptive commit message for these changes:\n\n${request.params.arguments?.changes}`
          }
        }
      ]
    };
  }

  if (request.params.name === "explain-code") {
    const language = request.params.arguments?.language || "Unknown";
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Explain how this ${language} code works:\n\n${request.params.arguments?.code}`
          }
        }
      ]
    };
  }

  throw new Error("Prompt implementation not found");
});
```

### 最佳实践

在实现提示词时：

1. 使用清晰、描述性的提示词名称
2. 为提示词和参数提供详细描述
3. 验证所有必需参数
4. 优雅地处理缺失参数
5. 考虑提示词模板的版本管理
6. 在适当时缓存动态内容
7. 实现错误处理
8. 为预期的参数格式编写文档
9. 考虑提示词的可组合性
10. 使用各种输入测试提示词

### UI 集成

提示词可以在客户端 UI 中呈现为：

- 斜杠命令
- 快捷操作
- 上下文菜单项
- 命令面板条目
- 引导式工作流
- 交互式表单

### 更新和变更

服务器可以通知客户端关于提示词的变更：

1. 服务器能力：`prompts.listChanged`
2. 通知：`notifications/prompts/list_changed`
3. 客户端重新获取提示词列表

### 安全考虑

在实现提示词时：

- 验证所有参数
- 清理用户输入
- 考虑速率限制
- 实现访问控制
- 审计提示词使用
- 适当处理敏感数据
- 验证生成的内容
- 实现超时
- 考虑提示注入风险
- 编写安全要求文档

## 工具（Tools）

工具是 Model Context Protocol (MCP) 中的强大原语，使服务器能够向客户端暴露可执行功能。通过工具，LLM 可以与外部系统交互、执行计算并在现实世界中采取行动。

### 概述

MCP 中的工具允许服务器暴露可由客户端调用、供 LLM 执行操作的可执行函数。工具的关键方面包括：

- **发现**：客户端可以通过发送 `tools/list` 请求获取可用工具列表
- **调用**：使用 `tools/call` 请求调用工具，服务器执行请求的操作并返回结果
- **灵活性**：工具可以涵盖从简单计算到复杂 API 交互的各种功能

与资源类似，工具由唯一名称标识，可以包含描述以指导其使用。然而，与资源不同的是，工具代表可以修改状态或与外部系统交互的动态操作。

每个工具定义为以下结构：

```
{
  name: string;          // 工具的唯一标识符
  description?: string;  // 人类可读的描述
  inputSchema: {         // 工具参数的 JSON Schema
    type: "object",
    properties: { ... }  // 工具特定的参数
  },
  annotations?: {        // 关于工具行为的可选提示
    title?: string;      // 工具的人类可读标题
    readOnlyHint?: boolean;    // 如果为 true，工具不修改其环境
    destructiveHint?: boolean; // 如果为 true，工具可能执行破坏性更新
    idempotentHint?: boolean;  // 如果为 true，使用相同参数重复调用没有额外效果
    openWorldHint?: boolean;   // 如果为 true，工具与外部实体交互
  }
}
```

以下是在 MCP 服务器中实现基本工具的示例：

```
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// 定义可用工具
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [{
      name: "calculate_sum",
      description: "Add two numbers together",
      inputSchema: {
        type: "object",
        properties: {
          a: { type: "number" },
          b: { type: "number" }
        },
        required: ["a", "b"]
      }
    }]
  };
});

// 处理工具执行
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "calculate_sum") {
    const { a, b } = request.params.arguments;
    return {
      content: [
        {
          type: "text",
          text: String(a + b)
        }
      ]
    };
  }
  throw new Error("Tool not found");
});
```

以下是服务器可以提供的工具类型示例：

#### 系统操作

与本地系统交互的工具：

```
{
  name: "execute_command",
  description: "Run a shell command",
  inputSchema: {
    type: "object",
    properties: {
      command: { type: "string" },
      args: { type: "array", items: { type: "string" } }
    }
  }
}
```

#### API 集成

封装外部 API 的工具：

```
{
  name: "github_create_issue",
  description: "Create a GitHub issue",
  inputSchema: {
    type: "object",
    properties: {
      title: { type: "string" },
      body: { type: "string" },
      labels: { type: "array", items: { type: "string" } }
    }
  }
}
```

#### 数据处理

转换或分析数据的工具：

```
{
  name: "analyze_csv",
  description: "Analyze a CSV file",
  inputSchema: {
    type: "object",
    properties: {
      filepath: { type: "string" },
      operations: {
        type: "array",
        items: {
          enum: ["sum", "average", "count"]
        }
      }
    }
  }
}
```

### 最佳实践

在实现工具时：

1. 提供清晰、描述性的名称和描述
2. 为参数使用详细的 JSON Schema 定义
3. 在工具描述中包含示例以演示模型应如何使用它们
4. 实现适当的错误处理和验证
5. 对长时间操作使用进度报告
6. 保持工具操作专注和原子化
7. 为预期的返回值结构编写文档
8. 实现适当的超时
9. 考虑对资源密集型操作进行速率限制
10. 记录工具使用以便调试和监控

### 安全考虑

在暴露工具时：

#### 输入验证

- 根据模式验证所有参数
- 清理文件路径和系统命令
- 验证 URL 和外部标识符
- 检查参数大小和范围
- 防止命令注入

#### 访问控制

- 在需要的地方实现认证
- 使用适当的授权检查
- 审计工具使用
- 对请求进行速率限制
- 监控滥用

#### 错误处理

- 不要向客户端暴露内部错误
- 记录安全相关的错误
- 适当处理超时
- 错误后清理资源
- 验证返回值

### 动态工具发现

MCP 支持动态工具发现：

1. 客户端可以随时列出可用工具
2. 服务器可以使用 `notifications/tools/list_changed` 通知客户端工具变更
3. 工具可以在运行时添加或移除
4. 工具定义可以更新（但应谨慎进行）

### 错误处理

工具错误应在结果对象中报告，而不是作为 MCP 协议级错误。这允许 LLM 看到并可能处理错误。当工具遇到错误时：

1. 在结果中将 `isError` 设置为 `true`
2. 在 `content` 数组中包含错误详情

以下是工具的正确错误处理示例：

```
try {
  // 工具操作
  const result = performOperation();
  return {
    content: [
      {
        type: "text",
        text: `Operation successful: ${result}`
      }
    ]
  };
} catch (error) {
  return {
    isError: true,
    content: [
      {
        type: "text",
        text: `Error: ${error.message}`
      }
    ]
  };
}
```

这种方法允许 LLM 看到错误的发生，并可能采取纠正措施或请求人工干预。

### 工具注解

工具注解提供了关于工具行为的额外元数据，帮助客户端理解如何展示和管理工具。这些注解是描述工具性质和影响的提示，不应作为安全决策的依据。

#### 工具注解的目的

工具注解服务于以下几个关键目的：

1. 提供特定于 UX 的信息而不影响模型上下文
2. 帮助客户端正确分类和展示工具
3. 传达关于工具潜在副作用的信息
4. 协助开发直观的工具审批界面

#### 可用的工具注解

MCP 规范为工具定义了以下注解：

| 注解 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| `title` | string | - | 工具的人类可读标题，适用于 UI 显示 |
| `readOnlyHint` | boolean | false | 如果为 true，表示工具不修改其环境 |
| `destructiveHint` | boolean | true | 如果为 true，工具可能执行破坏性更新（仅在 `readOnlyHint` 为 false 时有意义） |
| `idempotentHint` | boolean | false | 如果为 true，使用相同参数重复调用工具没有额外效果（仅在 `readOnlyHint` 为 false 时有意义） |
| `openWorldHint` | boolean | true | 如果为 true，工具可能与外部实体的"开放世界"交互 |

#### 使用示例

以下是为不同场景定义带注解工具的方法：

```
// 一个只读搜索工具
{
  name: "web_search",
  description: "Search the web for information",
  inputSchema: {
    type: "object",
    properties: {
      query: { type: "string" }
    },
    required: ["query"]
  },
  annotations: {
    title: "Web Search",
    readOnlyHint: true,
    openWorldHint: true
  }
}

// 一个破坏性文件删除工具
{
  name: "delete_file",
  description: "Delete a file from the filesystem",
  inputSchema: {
    type: "object",
    properties: {
      path: { type: "string" }
    },
    required: ["path"]
  },
  annotations: {
    title: "Delete File",
    readOnlyHint: false,
    destructiveHint: true,
    idempotentHint: true,
    openWorldHint: false
  }
}

// 一个非破坏性数据库记录创建工具
{
  name: "create_record",
  description: "Create a new record in the database",
  inputSchema: {
    type: "object",
    properties: {
      table: { type: "string" },
      data: { type: "object" }
    },
    required: ["table", "data"]
  },
  annotations: {
    title: "Create Database Record",
    readOnlyHint: false,
    destructiveHint: false,
    idempotentHint: false,
    openWorldHint: false
  }
}
```

#### 在服务器实现中集成注解

```
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [{
      name: "calculate_sum",
      description: "Add two numbers together",
      inputSchema: {
        type: "object",
        properties: {
          a: { type: "number" },
          b: { type: "number" }
        },
        required: ["a", "b"]
      },
      annotations: {
        title: "Calculate Sum",
        readOnlyHint: true,
        openWorldHint: false
      }
    }]
  };
});
```

#### 工具注解的最佳实践

1. **准确描述副作用**：清楚地指出工具是否修改其环境以及这些修改是否是破坏性的。
2. **使用描述性标题**：提供清晰描述工具目的的人类友好标题。
3. **正确标记幂等性**：仅在重复调用相同参数确实没有额外效果时将工具标记为幂等。
4. **设置适当的开放/封闭世界提示**：指出工具是与封闭系统（如数据库）交互还是与开放系统（如 Web）交互。
5. **记住注解只是提示**：ToolAnnotations 中的所有属性都是提示，不保证提供对工具行为的忠实描述。客户端不应仅基于注解做出安全关键决策。

### 测试策略

MCP 工具的综合测试策略应涵盖：

- **功能测试**：验证工具在有效输入下正确执行，并适当处理无效输入
- **集成测试**：使用真实和模拟的依赖项测试工具与外部系统的交互
- **安全测试**：验证认证、授权、输入清理和速率限制
- **性能测试**：检查负载下的行为、超时处理和资源清理
- **错误处理**：确保工具通过 MCP 协议正确报告错误并清理资源

## 采样（Sampling）

采样是一项强大的 MCP 功能，允许服务器通过客户端请求 LLM 补全，实现复杂的 agent 行为同时维护安全和隐私。

### 采样如何工作

采样流程遵循以下步骤：

1. 服务器向客户端发送 `sampling/createMessage` 请求
2. 客户端审查请求并可以修改它
3. 客户端从 LLM 采样
4. 客户端审查补全结果
5. 客户端将结果返回给服务器

这种人在回路中的设计确保用户对 LLM 看到和生成的内容保持控制。

### 消息格式

采样请求使用标准化的消息格式：

### 请求参数

#### 消息

`messages` 数组包含要发送给 LLM 的对话历史。每条消息包含：

- `role`："user" 或 "assistant"
- `content`：消息内容，可以是：
  - 带有 `text` 字段的文本内容
  - 带有 `data`（base64）和 `mimeType` 字段的图像内容

#### 模型偏好

`modelPreferences` 对象允许服务器指定其模型选择偏好：

- `hints`：模型名称建议数组，客户端可用于选择适当的模型：
  - `name`：可以匹配完整或部分模型名称的字符串（例如 "claude-3"、"sonnet"）
  - 客户端可以将提示映射到不同提供商的等效模型
  - 多个提示按偏好顺序评估
- 优先级值（0-1 归一化）：
  - `costPriority`：最小化成本的重要性
  - `speedPriority`：低延迟响应的重要性
  - `intelligencePriority`：高级模型能力的重要性

客户端根据这些偏好及其可用模型做出最终的模型选择。

#### 系统提示词

可选的 `systemPrompt` 字段允许服务器请求特定的系统提示词。客户端可以修改或忽略此字段。

#### 上下文包含

`includeContext` 参数指定要包含的 MCP 上下文：

- `"none"`：不包含额外上下文
- `"thisServer"`：包含来自请求服务器的上下文
- `"allServers"`：包含来自所有已连接 MCP 服务器的上下文

客户端控制实际包含的上下文。

#### 采样参数

微调 LLM 采样的参数：

- `temperature`：控制随机性（0.0 到 1.0）
- `maxTokens`：最大生成令牌数
- `stopSequences`：停止生成的序列数组
- `metadata`：额外的提供商特定参数

### 响应格式

客户端返回一个补全结果：

### 请求示例

以下是从客户端请求采样的示例：

### 最佳实践

在实现采样时：

1. 始终提供清晰、结构良好的提示词
2. 适当处理文本和图像内容
3. 设置合理的令牌限制
4. 通过 `includeContext` 包含相关上下文
5. 在使用前验证响应
6. 优雅地处理错误
7. 考虑对采样请求进行速率限制
8. 为预期的采样行为编写文档
9. 使用各种模型参数进行测试
10. 监控采样成本

### 人在回路中的控制

采样在设计时就考虑了人工监督：

#### 对于提示词

- 客户端应向用户展示提议的提示词
- 用户应能够修改或拒绝提示词
- 系统提示词可以被过滤或修改
- 上下文包含由客户端控制

#### 对于补全

- 客户端应向用户展示补全结果
- 用户应能够修改或拒绝补全
- 客户端可以过滤或修改补全
- 用户控制使用哪个模型

### 安全考虑

在实现采样时：

- 验证所有消息内容
- 清理敏感信息
- 实现适当的速率限制
- 监控采样使用
- 在传输中加密数据
- 处理用户数据隐私
- 审计采样请求
- 控制成本暴露
- 实现超时
- 优雅地处理模型错误

### 常见模式

#### Agent 工作流

采样支持以下 agent 模式：

- 读取和分析资源
- 基于上下文做出决策
- 生成结构化数据
- 处理多步任务
- 提供交互式辅助

#### 上下文管理

上下文的最佳实践：

- 请求最少的必要上下文
- 清晰地构建上下文结构
- 处理上下文大小限制
- 根据需要更新上下文
- 清理过时的上下文

#### 错误处理

健壮的错误处理应该：

- 捕获采样失败
- 处理超时错误
- 管理速率限制
- 验证响应
- 提供回退行为
- 适当记录错误

### 限制

请注意以下限制：

- 采样依赖于客户端能力
- 用户控制采样行为
- 上下文大小有限制
- 可能适用速率限制
- 应考虑成本
- 模型可用性各异
- 响应时间各异
- 并非所有内容类型都受支持
