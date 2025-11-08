# ValueCell 前端架构文档

> 本文档详细介绍 ValueCell 前端的技术架构、设计模式和核心实现

## 📚 技术栈概览

### 核心框架

```
├── React 19.2.0              # 最新 React 版本
├── React Router v7.9.4       # 文件路由系统
├── TypeScript 5.9.3          # 类型安全
└── Vite 7.1.12              # 构建工具
```

### 状态管理

```
├── Zustand 5.0.8            # 轻量级状态管理
└── TanStack Query 5.90.5    # 服务端状态管理
```

### UI 组件库

```
├── Radix UI                 # 无障碍组件原语
├── Tailwind CSS v4          # 原子化 CSS
├── Lucide React             # 图标库
└── Shadcn/ui                # 组件系统
```

### 数据可视化

```
└── ECharts 6.0.0            # 图表库
```

### 实用工具

```
├── Dayjs                    # 日期处理
├── Zod 4.1.12               # 运行时验证
└── best-effort-json-parser  # 容错 JSON 解析
```

---

## 🗂️ 项目结构

```
frontend/src/
├── app/                    # 📄 页面路由（React Router v7 文件路由）
│   ├── agent/             # Agent 聊天相关页面
│   │   ├── chat.tsx       # Agent 聊天主页面
│   │   ├── config.tsx     # Agent 配置页面
│   │   └── components/    # Agent 专用组件
│   ├── home/              # 首页和股票详情
│   │   ├── _layout.tsx    # 布局组件
│   │   ├── home.tsx       # 首页
│   │   └── stock.tsx      # 股票详情页
│   ├── market/            # 市场/Agent 市场
│   ├── setting/           # 设置页面
│   └── redirect-to-home.tsx
│
├── api/                   # 🌐 API 客户端层
│   ├── agent.ts           # Agent 相关 API
│   ├── conversation.ts    # 对话管理 API
│   ├── stock.ts           # 股票数据 API
│   ├── strategy.ts        # 策略 API
│   └── setting.ts         # 设置 API
│
├── components/            # 🎨 组件库
│   ├── ui/                # Shadcn/ui 基础组件
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   ├── input.tsx
│   │   └── ...
│   └── valuecell/         # 业务组件
│       ├── agent-avatar.tsx
│       ├── app-sidebar.tsx
│       ├── charts/         # 图表组件
│       ├── renderer/       # 消息渲染器
│       └── ...
│
├── store/                 # 💾 Zustand 全局状态
│   ├── agent-store.ts     # Agent 对话状态
│   └── settings-store.ts  # 应用设置状态
│
├── lib/                   # 🔧 核心工具库
│   ├── api-client.ts      # HTTP 客户端
│   ├── sse-client.ts      # Server-Sent Events 客户端
│   ├── agent-store.ts     # Agent 状态管理逻辑
│   └── utils.ts           # 通用工具函数
│
├── hooks/                 # 🪝 自定义 Hooks
│   ├── use-sse.ts         # SSE 连接 Hook
│   ├── use-mobile.ts      # 移动端检测
│   ├── use-debounce.ts    # 防抖
│   └── use-chart-resize.ts # 图表自适应
│
├── types/                 # 📝 TypeScript 类型定义
│   ├── agent.ts           # Agent 事件类型
│   ├── conversation.ts    # 对话类型
│   ├── stock.ts           # 股票数据类型
│   └── strategy.ts        # 策略类型
│
├── constants/             # 📌 常量定义
│   ├── agent.ts           # Agent 相关常量
│   ├── api.ts             # API 端点
│   └── stock.ts           # 股票常量
│
└── provider/              # 🔌 Context Providers
    └── multi-section-provider.tsx
```

---

## 🚦 路由系统 (React Router v7)

### 文件路由配置

文件路径：`src/routes.ts`

```typescript
export default [
  // 根路径重定向到 /home
  index("app/redirect-to-home.tsx"),

  // /home/* - 首页和股票详情
  ...prefix("/home", [
    layout("app/home/_layout.tsx", [
      index("app/home/home.tsx"),                      // /home
      route("/stock/:stockId", "app/home/stock.tsx"),  // /home/stock/AAPL
    ]),
  ]),

  // /market - Agent 市场
  route("/market", "app/market/agents.tsx"),

  // /agent/* - Agent 聊天和配置
  ...prefix("/agent", [
    route("/:agentName", "app/agent/chat.tsx"),              // /agent/ResearchAgent
    route("/:agentName/config", "app/agent/config.tsx"),     // /agent/ResearchAgent/config
  ]),

  // /setting/* - 设置页面
  ...prefix("/setting", [
    layout("app/setting/_layout.tsx", [
      index("app/setting/general.tsx"),            // /setting
      route("/memory", "app/setting/memory.tsx"),  // /setting/memory
    ]),
  ]),

  // 测试路由
  route("/test", "app/test.tsx"),
] satisfies RouteConfig;
```

### 路由特点

- ✅ **文件系统路由** - 直观易维护，遵循约定优于配置
- ✅ **嵌套布局** - 通过 `layout` 实现共享布局
- ✅ **动态参数** - 支持 `:agentName`, `:stockId` 等参数
- ✅ **路由分组** - 使用 `prefix` 进行逻辑分组

### 路由映射表

| 路由路径 | 文件位置 | 说明 |
|---------|---------|------|
| `/` | `app/redirect-to-home.tsx` | 重定向到首页 |
| `/home` | `app/home/home.tsx` | 首页 |
| `/home/stock/:stockId` | `app/home/stock.tsx` | 股票详情页 |
| `/market` | `app/market/agents.tsx` | Agent 市场 |
| `/agent/:agentName` | `app/agent/chat.tsx` | Agent 聊天页面 |
| `/agent/:agentName/config` | `app/agent/config.tsx` | Agent 配置页面 |
| `/setting` | `app/setting/general.tsx` | 通用设置 |
| `/setting/memory` | `app/setting/memory.tsx` | 记忆设置 |

---

## 💾 状态管理架构

### 1. Zustand Store - Agent 对话状态

文件路径：`src/store/agent-store.ts`

```typescript
interface AgentStoreState {
  // 所有对话数据存储
  agentStore: AgentConversationsStore;

  // 当前活动对话 ID
  curConversationId: string;

  // 更新单条 SSE 消息到 store
  dispatchAgentStore: (action: SSEData) => void;

  // 批量更新历史消息
  dispatchAgentStoreHistory: (
    conversationId: string,
    history: SSEData[],
    clearHistory?: boolean
  ) => void;

  // 切换当前活动对话
  setCurConversationId: (conversationId: string) => void;

  // 重置整个 store
  resetStore: () => void;
}
```

### Agent Store 数据结构

```
AgentConversationsStore
└── [conversationId]           # 对话级别
    └── threads
        └── [threadId]         # 线程级别
            └── tasks
                └── [taskId]   # 任务级别
                    └── items
                        └── [itemId]  # 消息项级别
                            ├── role
                            ├── content
                            ├── component_type
                            └── metadata
```

**层级说明：**

- **Conversation**: 一个完整的对话会话
- **Thread**: 对话中的消息链
- **Task**: Agent 执行的单个任务
- **Item**: 最小的渲染单元（一条消息/组件）

### 2. 设置状态管理

文件路径：`src/store/settings-store.ts`

```typescript
interface SettingsStoreState {
  // 应用设置对象
  settings: Settings;

  // 更新设置（支持部分更新）
  updateSettings: (updates: Partial<Settings>) => void;
}

interface Settings {
  theme: 'light' | 'dark' | 'system';
  language: string;
  stockQuotesColor: 'red-up-green-down' | 'green-up-red-down';
  // ... 更多设置项
}
```

### Store 使用示例

```typescript
// 组件中使用 Zustand store
import { useAgentStore } from '@/store/agent-store';

function ChatComponent() {
  // 选择性订阅状态（性能优化）
  const { dispatchAgentStore, curConversationId } = useAgentStore(
    useShallow((s) => ({
      dispatchAgentStore: s.dispatchAgentStore,
      curConversationId: s.curConversationId
    }))
  );

  // 使用状态和方法
  const handleNewMessage = (data: SSEData) => {
    dispatchAgentStore(data);
  };

  // ...
}
```

---

## 🌐 实时通信架构 - SSE (Server-Sent Events)

### SSE 客户端实现

文件路径：`src/lib/sse-client.ts`

```typescript
export class SSEClient {
  // 连接状态
  private readyState: SSEReadyState = SSEReadyState.CLOSED;

  // AbortController 用于取消连接
  private abortController: AbortController | null = null;

  // 事件处理器
  private handlers: SSEEventHandlers = {};

  constructor(options: SSEOptions, handlers?: SSEEventHandlers) {
    this.options = this.resolveOptions(options);
    this.handlers = handlers ?? {};
  }

  /**
   * 连接到 SSE 端点
   */
  async connect(body?: BodyInit): Promise<void> {
    // 防止重复连接
    if (this.readyState === SSEReadyState.CONNECTING ||
        this.readyState === SSEReadyState.OPEN) return;

    this.setReadyState(SSEReadyState.CONNECTING);
    this.abortController = new AbortController();

    try {
      const response = await fetch(this.options.url, {
        method: 'POST',
        headers: this.options.headers,
        body,
        signal: this.abortController.signal,
        ...this.options.fetchOptions,
      });

      if (!response.ok) {
        throw new Error(`SSE connection failed: ${response.statusText}`);
      }

      this.setReadyState(SSEReadyState.OPEN);
      this.handlers.onOpen?.();

      // 读取流式响应
      await this.readStream(response.body);
    } catch (error) {
      this.handleError(error);
    }
  }

  /**
   * 关闭连接
   */
  close(): void {
    this.abortController?.abort();
    this.setReadyState(SSEReadyState.CLOSED);
    this.handlers.onClose?.();
  }

  /**
   * 读取流式数据
   */
  private async readStream(body: ReadableStream): Promise<void> {
    const reader = body.getReader();
    const decoder = new TextDecoder();

    try {
      while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const text = decoder.decode(value, { stream: true });
        this.parseSSEData(text);
      }
    } finally {
      reader.releaseLock();
      this.close();
    }
  }

  /**
   * 解析 SSE 数据格式
   */
  private parseSSEData(text: string): void {
    const lines = text.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        try {
          const parsed = parse(data); // 容错 JSON 解析
          this.handlers.onData?.(parsed);
        } catch (error) {
          console.error('Failed to parse SSE data:', error);
        }
      }
    }
  }
}
```

### SSE 连接状态

```typescript
export enum SSEReadyState {
  CONNECTING = 0,  // 正在连接
  OPEN = 1,        // 连接已建立
  CLOSED = 2,      // 连接已关闭
}
```

### SSE 事件处理器接口

```typescript
export interface SSEEventHandlers {
  /** 接收到数据时调用 */
  onData?: (data: SSEData) => void;

  /** 连接建立时调用 */
  onOpen?: () => void;

  /** 发生错误时调用 */
  onError?: (error: Error) => void;

  /** 连接关闭时调用 */
  onClose?: () => void;

  /** 连接状态变化时调用 */
  onStateChange?: (state: SSEReadyState) => void;
}
```

### SSE 事件类型系统

文件路径：`src/types/agent.ts`

```typescript
export interface AgentEventMap {
  // ========== 生命周期事件 ==========
  conversation_started: Pick<BaseEventData, "conversation_id">;
  thread_started: AgentThreadStartedMessage;
  done: Pick<BaseEventData, "conversation_id" | "thread_id">;

  // ========== 消息流式传输 ==========
  message_chunk: AgentChunkMessage;     // 流式文本片段
  message: AgentChunkMessage;           // 完整消息

  // ========== 组件生成 ==========
  component_generator: AgentComponentMessage;

  // ========== 用户交互 ==========
  plan_require_user_input: AgentPlanRequireUserInputMessage;

  // ========== 工具调用生命周期 ==========
  tool_call_started: AgentToolCallMessage;
  tool_call_completed: AgentToolCallMessage;

  // ========== 推理过程 ==========
  reasoning: AgentReasoningMessage;
  reasoning_started: BaseEventData;
  reasoning_completed: BaseEventData;

  // ========== 错误处理 ==========
  plan_failed: AgentPlanFailedMessage;
  task_failed: AgentTaskFailedMessage;
  system_failed: AgentSystemFailedMessage;
}
```

### 基础事件数据结构

```typescript
interface BaseEventData {
  role: "user" | "agent" | "system";
  conversation_id: string;  // 顶层对话会话
  thread_id: string;        // 对话中的消息链
  task_id: string;          // 单个 agent 执行单元
  item_id: string;          // 最小粒度渲染级别
  metadata: Partial<{
    task_title: string;     // 任务标题
  }>;
}

// 消息类型
export type AgentChunkMessage = BaseEventData & {
  payload: {
    content: string;
  };
};

// 组件消息类型
export type AgentComponentMessage = BaseEventData & {
  payload: {
    component_type: AgentComponentType;
    content: string;
  };
};

// 工具调用消息类型
export type AgentToolCallMessage = BaseEventData & {
  payload: {
    tool_call_id: string;
    tool_name: string;
    tool_call_result?: string;
  };
};
```

### 自定义 SSE Hook

文件路径：`src/hooks/use-sse.ts`

```typescript
export const useSSE = (
  options: SSEOptions,
  handlers?: SSEEventHandlers
) => {
  const sseClient = useRef<SSEClient | null>(null);

  const connect = useCallback((body?: BodyInit) => {
    // 创建新的 SSE 客户端实例
    sseClient.current = new SSEClient(options, handlers);
    return sseClient.current.connect(body);
  }, [options, handlers]);

  const close = useCallback(() => {
    sseClient.current?.close();
    sseClient.current = null;
  }, []);

  // 组件卸载时自动关闭连接
  useEffect(() => {
    return () => {
      close();
    };
  }, [close]);

  return { connect, close };
};
```

---

## 🔄 数据流架构

### 完整数据流程图

```
┌─────────────┐
│  用户输入    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  API 请求   │ ← apiClient.post('/agents/stream', data)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ SSE 连接建立 │ ← sseClient.connect(body)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│后端流式返回  │ ← message_chunk, component_generator, ...
│  SSE 事件    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│SSEClient 解析│ ← parseSSEData(text)
│    事件      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ onData 回调  │ ← handlers.onData(parsedData)
│    触发      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│dispatchAgent │ ← updateAgentConversationsStore()
│Store 更新状态│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Zustand Store │ ← set(newState)
│    更新      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ React 组件  │ ← useAgentStore()
│ 自动重新渲染 │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ UI 实时更新 │ ← 显示流式消息/组件
│    显示     │
└─────────────┘
```

### 代码实现示例

```typescript
// Agent 聊天页面集成 SSE
function AgentChatPage() {
  const { dispatchAgentStore, curConversationId } = useAgentStore();
  const [isConnecting, setIsConnecting] = useState(false);

  // 初始化 SSE
  const { connect, close } = useSSE(
    {
      url: '/api/v1/agents/stream',
      timeout: 30000
    },
    {
      onData: (data: SSEData) => {
        // 接收 SSE 事件并更新到 Zustand store
        dispatchAgentStore(data);
      },
      onOpen: () => {
        setIsConnecting(false);
        console.log('SSE connected');
      },
      onError: (error) => {
        console.error('SSE Error:', error);
        setIsConnecting(false);
        toast.error('连接失败');
      },
      onClose: () => {
        setIsConnecting(false);
        console.log('SSE closed');
      }
    }
  );

  // 发送消息
  const sendMessage = async (message: string) => {
    setIsConnecting(true);

    try {
      await connect(JSON.stringify({
        agent_name: 'ResearchAgent',
        query: message,
        conversation_id: curConversationId || undefined
      }));
    } catch (error) {
      console.error('Failed to send message:', error);
      setIsConnecting(false);
    }
  };

  return (
    <div>
      {/* 聊天界面 */}
      <MessageList />
      <MessageInput onSend={sendMessage} disabled={isConnecting} />
    </div>
  );
}
```

---

## 🎨 组件渲染系统

### 消息渲染器架构

```
components/valuecell/renderer/
├── text-renderer.tsx        # 纯文本渲染
├── markdown-renderer.tsx    # Markdown 渲染（支持 GFM）
├── code-renderer.tsx        # 代码块渲染（语法高亮）
├── table-renderer.tsx       # 表格数据渲染
├── chart-renderer.tsx       # 图表渲染（ECharts）
├── stock-card-renderer.tsx  # 股票卡片
├── component-renderer.tsx   # 动态组件路由器
└── index.tsx                # 统一导出
```

### 组件类型系统

文件路径：`src/constants/agent.ts`

```typescript
// Agent 组件类型定义
export const AGENT_COMPONENT_TYPE = [
  // 基础内容类型
  "text",
  "markdown",
  "code",

  // 数据展示
  "table",
  "chart",
  "json",

  // 业务组件
  "stock_card",
  "stock_list",
  "strategy_summary",
  "portfolio_view",

  // 交互组件
  "form",
  "button_group",

  // 工具组件
  "error",
  "loading",
  "divider",
] as const;

export type AgentComponentType = typeof AGENT_COMPONENT_TYPE[number];

// 区段组件类型（可折叠）
export const AGENT_SECTION_COMPONENT_TYPE = [
  "reasoning",      // 推理过程
  "tool_call",      // 工具调用
  "search_result",  // 搜索结果
] as const;

// 多区段组件类型
export const AGENT_MULTI_SECTION_COMPONENT_TYPE = [
  "analysis",       // 分析报告
  "research",       // 研究报告
] as const;
```

### 动态组件渲染逻辑

```typescript
// component-renderer.tsx
export function ComponentRenderer({ item }: { item: ChatItem }) {
  const { component_type, payload } = item;

  // 根据 component_type 动态渲染对应组件
  switch (component_type) {
    case "text":
      return <TextRenderer content={payload.content} />;

    case "markdown":
      return <MarkdownRenderer content={payload.content} />;

    case "code":
      return <CodeRenderer content={payload.content} />;

    case "table":
      return <TableRenderer data={JSON.parse(payload.content)} />;

    case "chart":
      return <ChartRenderer data={JSON.parse(payload.content)} />;

    case "stock_card":
      return <StockCardRenderer data={JSON.parse(payload.content)} />;

    case "strategy_summary":
      return <StrategySummaryRenderer data={JSON.parse(payload.content)} />;

    case "error":
      return <ErrorRenderer message={payload.content} />;

    default:
      // 未知类型降级到文本渲染
      return <TextRenderer content={payload.content} />;
  }
}
```

### Markdown 渲染器

```typescript
// markdown-renderer.tsx
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import rehypeRaw from 'rehype-raw';

export function MarkdownRenderer({ content }: { content: string }) {
  return (
    <ReactMarkdown
      remarkPlugins={[remarkGfm]}    // 支持 GitHub Flavored Markdown
      rehypePlugins={[rehypeRaw]}    // 支持原始 HTML
      components={{
        // 自定义组件渲染
        code: CodeBlock,
        table: Table,
        a: Link,
        img: Image,
      }}
    >
      {content}
    </ReactMarkdown>
  );
}
```

### 图表渲染器

```typescript
// chart-renderer.tsx
import * as echarts from 'echarts';
import { useEffect, useRef } from 'react';

export function ChartRenderer({ data }: { data: EChartsOption }) {
  const chartRef = useRef<HTMLDivElement>(null);
  const chartInstance = useRef<echarts.ECharts | null>(null);

  useEffect(() => {
    if (!chartRef.current) return;

    // 初始化 ECharts 实例
    chartInstance.current = echarts.init(chartRef.current);
    chartInstance.current.setOption(data);

    // 响应式调整
    const handleResize = () => {
      chartInstance.current?.resize();
    };
    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
      chartInstance.current?.dispose();
    };
  }, [data]);

  return <div ref={chartRef} style={{ width: '100%', height: '400px' }} />;
}
```

---

## 📡 API 客户端层

### API Client 实现

文件路径：`src/lib/api-client.ts`

```typescript
export class ApiClient {
  // 默认配置
  private config: RequestConfig = {
    requiresAuth: true,
    headers: {
      "Content-Type": "application/json",
    },
  };

  /**
   * GET 请求
   */
  async get<T>(endpoint: string, config?: RequestConfig): Promise<T> {
    return this.request<T>('GET', endpoint, undefined, config);
  }

  /**
   * POST 请求
   */
  async post<T>(
    endpoint: string,
    data?: unknown,
    config?: RequestConfig
  ): Promise<T> {
    return this.request<T>('POST', endpoint, data, config);
  }

  /**
   * PUT 请求
   */
  async put<T>(
    endpoint: string,
    data?: unknown,
    config?: RequestConfig
  ): Promise<T> {
    return this.request<T>('PUT', endpoint, data, config);
  }

  /**
   * DELETE 请求
   */
  async delete<T>(endpoint: string, config?: RequestConfig): Promise<T> {
    return this.request<T>('DELETE', endpoint, undefined, config);
  }

  /**
   * 统一请求方法
   */
  private async request<T>(
    method: string,
    endpoint: string,
    data?: unknown,
    config: RequestConfig = {},
  ): Promise<T> {
    const mergedConfig = { ...this.config, ...config };
    const url = getServerUrl(endpoint);

    const headers: HeadersInit = {
      ...mergedConfig.headers,
    };

    // 添加认证 token（如果需要）
    if (mergedConfig.requiresAuth) {
      const token = localStorage.getItem('authToken');
      if (token) {
        headers['Authorization'] = `Bearer ${token}`;
      }
    }

    const requestOptions: RequestInit = {
      method,
      headers,
      signal: config.signal,
    };

    if (data) {
      requestOptions.body = JSON.stringify(data);
    }

    try {
      const response = await fetch(url, requestOptions);
      return await this.handleResponse<T>(response);
    } catch (error) {
      if (error instanceof ApiError) throw error;
      throw new ApiError('Network error', 0, error);
    }
  }

  /**
   * 统一响应处理
   */
  private async handleResponse<T>(response: Response): Promise<T> {
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      const message =
        errorData.message ||
        errorData.detail ||
        response.statusText ||
        `HTTP ${response.status}`;

      // 401 未授权处理
      if (response.status === 401) {
        localStorage.removeItem("authToken");
        if (typeof window !== "undefined") {
          window.location.href = "/login";
        }
      }

      // 显示错误提示
      toast.error(message);

      throw new ApiError(message, response.status, errorData);
    }

    const contentType = response.headers.get("content-type");
    if (contentType?.includes("application/json")) {
      return response.json();
    }

    return response.text() as unknown as T;
  }
}

// 导出单例
export const apiClient = new ApiClient();
```

### API 模块示例

文件路径：`src/api/agent.ts`

```typescript
import { apiClient } from '@/lib/api-client';
import type { Agent, Conversation, SSEData } from '@/types/agent';

export const agentApi = {
  /**
   * 获取所有可用的 Agent
   */
  getAgents: (params?: { enabled_only?: boolean }) =>
    apiClient.get<Agent[]>('/agents', {
      requiresAuth: false,
    }),

  /**
   * 获取单个 Agent 详情
   */
  getAgent: (agentName: string) =>
    apiClient.get<Agent>(`/agents/${agentName}`),

  /**
   * 创建新对话
   */
  createConversation: (data: {
    agent_name: string;
    title?: string;
  }) =>
    apiClient.post<Conversation>('/conversations', data),

  /**
   * 获取对话历史
   */
  getConversationHistory: (conversationId: string) =>
    apiClient.get<SSEData[]>(`/conversations/${conversationId}/history`),

  /**
   * 获取所有对话列表
   */
  getConversations: (params?: {
    agent_name?: string;
    limit?: number;
    offset?: number;
  }) =>
    apiClient.get<Conversation[]>('/conversations', {
      requiresAuth: false,
    }),

  /**
   * 删除对话
   */
  deleteConversation: (conversationId: string) =>
    apiClient.delete(`/conversations/${conversationId}`),
};
```

---

## 🎯 核心设计模式

### 1. 分层架构

```
┌─────────────────────────────────┐
│   Presentation Layer            │  ← Components (React)
│   (UI Components)               │
├─────────────────────────────────┤
│   Business Logic Layer          │  ← Hooks, Stores (Zustand)
│   (State Management)            │
├─────────────────────────────────┤
│   Data Access Layer             │  ← API Client, Services
│   (API Integration)             │
├─────────────────────────────────┤
│   Network Layer                 │  ← SSE, HTTP (Fetch)
│   (Communication)               │
└─────────────────────────────────┘
```

**职责划分：**

- **Presentation Layer**: 负责 UI 渲染和用户交互
- **Business Logic Layer**: 负责业务逻辑和状态管理
- **Data Access Layer**: 负责数据获取和转换
- **Network Layer**: 负责网络通信

### 2. 单向数据流

```
User Action
    ↓
Event Handler
    ↓
Dispatch Action
    ↓
Update Store
    ↓
Component Re-render
    ↓
UI Update
```

### 3. 关注点分离

```typescript
// ✅ 好的实践：关注点分离
// UI 组件 - 只负责展示
function MessageItem({ message }: { message: ChatItem }) {
  return <div>{message.content}</div>;
}

// 业务逻辑 - Hook 封装
function useMessageList(conversationId: string) {
  const store = useAgentStore();
  const messages = store.agentStore[conversationId]?.threads || [];

  return { messages };
}

// 页面组件 - 组合使用
function ChatPage() {
  const { messages } = useMessageList(conversationId);

  return (
    <div>
      {messages.map(msg => <MessageItem key={msg.id} message={msg} />)}
    </div>
  );
}
```

### 4. 组合优于继承

```typescript
// ✅ 使用组合模式
function Card({ children, header, footer }) {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// 使用
<Card
  header={<CardHeader title="Agent" />}
  footer={<CardActions />}
>
  <CardContent />
</Card>
```

---

## ⚡ 性能优化策略

### 1. 代码分割

```typescript
// React Router v7 自动按路由分割
// 每个路由文件会被打包成独立的 chunk

// 手动懒加载组件
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function Page() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### 2. 组件记忆化

```typescript
// 使用 React.memo 避免不必要的重渲染
const MessageItem = memo(function MessageItem({ message }: Props) {
  return <div>{message.content}</div>;
});

// 使用 useMemo 缓存计算结果
function ChatList({ messages }) {
  const sortedMessages = useMemo(() => {
    return messages.sort((a, b) => a.timestamp - b.timestamp);
  }, [messages]);

  return <div>{sortedMessages.map(...)}</div>;
}

// 使用 useCallback 缓存函数引用
function ChatInput() {
  const handleSend = useCallback((message: string) => {
    sendMessage(message);
  }, [sendMessage]);

  return <Input onSend={handleSend} />;
}
```

### 3. 虚拟滚动

```typescript
// 对于长列表使用虚拟滚动
import { useVirtualizer } from '@tanstack/react-virtual';

function MessageList({ messages }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: messages.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100,
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(item => (
          <div key={item.key} style={{ transform: `translateY(${item.start}px)` }}>
            <MessageItem message={messages[item.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 4. 防抖和节流

```typescript
// 防抖：用户停止输入后才执行
const debouncedSearch = useDebouncedCallback(
  (value: string) => {
    search(value);
  },
  500 // 延迟 500ms
);

// 节流：限制执行频率
const throttledScroll = useThrottledCallback(
  () => {
    handleScroll();
  },
  100 // 每 100ms 最多执行一次
);
```

### 5. 图片优化

```typescript
// 懒加载图片
<img
  src={lowResImage}
  data-src={highResImage}
  loading="lazy"
  alt="description"
/>

// 使用现代图片格式
<picture>
  <source srcSet="image.webp" type="image/webp" />
  <source srcSet="image.avif" type="image/avif" />
  <img src="image.jpg" alt="fallback" />
</picture>
```

---

## 🔐 类型安全体系

### 端到端类型安全

```typescript
// 1. API 响应类型定义
interface ApiResponse<T> {
  code: number;
  data: T;
  msg: string;
}

// 2. 业务数据类型
interface Agent {
  name: string;
  display_name: string;
  description: string;
  enabled: boolean;
  capabilities: {
    streaming: boolean;
    push_notifications: boolean;
  };
}

// 3. Zod Schema 验证
const agentSchema = z.object({
  name: z.string(),
  display_name: z.string(),
  description: z.string(),
  enabled: z.boolean(),
  capabilities: z.object({
    streaming: z.boolean(),
    push_notifications: z.boolean(),
  }),
});

// 4. 类型推导
type Agent = z.infer<typeof agentSchema>;

// 5. 运行时验证
function validateAgent(data: unknown): Agent {
  return agentSchema.parse(data);
}

// 6. API 调用时的类型安全
const agents = await apiClient.get<Agent[]>('/agents');
//    ^? agents: Agent[]
```

### 类型保护

```typescript
// 类型守卫
function isAgentMessage(message: unknown): message is AgentChunkMessage {
  return (
    typeof message === 'object' &&
    message !== null &&
    'role' in message &&
    'payload' in message &&
    typeof (message as any).payload.content === 'string'
  );
}

// 使用
if (isAgentMessage(data)) {
  // data 类型自动收窄为 AgentChunkMessage
  console.log(data.payload.content);
}
```

---

## 🎭 主题系统

### 主题配置

```typescript
// root.tsx
import { ThemeProvider } from 'next-themes';

export default function Root() {
  return (
    <ThemeProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      <App />
    </ThemeProvider>
  );
}
```

### 主题切换

```typescript
// 使用 next-themes
import { useTheme } from 'next-themes';

function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      Toggle Theme
    </button>
  );
}
```

### Tailwind 暗色模式

```typescript
// 使用 class 策略
<div className="bg-white dark:bg-gray-900">
  <p className="text-black dark:text-white">Content</p>
</div>
```

---

## 🧪 测试策略

### 单元测试

```typescript
// 使用 Vitest
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { MessageItem } from './MessageItem';

describe('MessageItem', () => {
  it('renders message content', () => {
    const message = {
      id: '1',
      content: 'Hello World',
      role: 'agent'
    };

    render(<MessageItem message={message} />);
    expect(screen.getByText('Hello World')).toBeInTheDocument();
  });
});
```

### 集成测试

```typescript
// 测试完整用户流程
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useAgentStore } from '@/store/agent-store';

describe('Agent Chat Flow', () => {
  it('should update store when receiving SSE data', () => {
    const { result } = renderHook(() => useAgentStore());

    act(() => {
      result.current.dispatchAgentStore({
        type: 'message_chunk',
        conversation_id: 'conv-1',
        thread_id: 'thread-1',
        task_id: 'task-1',
        item_id: 'item-1',
        payload: { content: 'Hello' }
      });
    });

    expect(result.current.agentStore['conv-1']).toBeDefined();
  });
});
```

---

## 📦 构建和部署

### 构建命令

```bash
# 开发模式
bun run dev

# 类型检查
bun run typecheck

# 代码检查和格式化
bun run check:fix

# 生产构建
bun run build

# 预览构建结果
bun run start
```

### 环境变量

```bash
# .env
VITE_API_BASE_URL=http://localhost:8000/api/v1
VITE_APP_NAME=ValueCell
VITE_APP_VERSION=0.1.0
```

### 构建优化

- ✅ Tree Shaking（自动移除未使用代码）
- ✅ Code Splitting（按路由分割）
- ✅ Asset Optimization（图片和字体优化）
- ✅ Minification（代码压缩）
- ✅ Source Maps（生产环境禁用）

---

## 🔧 开发工具

### 代码质量工具

```json
{
  "scripts": {
    "lint": "biome lint --diagnostic-level=error ./src",
    "format": "biome format --write ./src",
    "check": "biome check ./src",
    "check:fix": "biome check --write ./src"
  }
}
```

### VS Code 推荐扩展

- Biome (Code formatter and linter)
- Tailwind CSS IntelliSense
- TypeScript Vue Plugin (Volar)
- Error Lens

---

## 📚 最佳实践

### 1. 组件设计原则

- **单一职责**: 每个组件只做一件事
- **Props 接口清晰**: 使用 TypeScript 定义明确的 Props
- **可组合**: 支持通过组合创建复杂组件
- **可访问性**: 遵循 ARIA 规范

### 2. 状态管理原则

- **最小化状态**: 只存储必要的状态
- **派生状态计算**: 使用 `useMemo` 计算派生值
- **避免冗余**: 不在多个地方存储相同数据
- **状态归一化**: 使用 ID 索引而非嵌套对象

### 3. 性能优化原则

- **延迟加载**: 非关键资源延迟加载
- **代码分割**: 按需加载代码
- **记忆化**: 缓存计算结果和函数引用
- **虚拟化**: 长列表使用虚拟滚动

### 4. 安全原则

- **XSS 防护**: 避免使用 `dangerouslySetInnerHTML`
- **CSRF 防护**: API 请求包含 CSRF token
- **输入验证**: 使用 Zod 验证用户输入
- **敏感信息**: 不在前端存储敏感数据

---

## 🎓 学习资源

### 官方文档

- [React 19 文档](https://react.dev)
- [React Router v7 文档](https://reactrouter.com)
- [Zustand 文档](https://docs.pmnd.rs/zustand)
- [Tailwind CSS 文档](https://tailwindcss.com)
- [Radix UI 文档](https://www.radix-ui.com)

### 推荐阅读

- React 设计模式
- TypeScript 最佳实践
- Web 性能优化
- 前端架构设计

---

## 📝 总结

ValueCell 前端架构的核心特点：

✅ **现代化技术栈** - React 19 + React Router v7 + TypeScript
✅ **实时通信** - SSE 流式数据传输
✅ **类型安全** - 端到端 TypeScript + Zod 验证
✅ **状态管理** - Zustand 轻量高效
✅ **组件化设计** - Radix UI + Shadcn/ui
✅ **性能优化** - 代码分割 + 虚拟滚动 + 记忆化
✅ **开发体验** - 文件路由 + 类型推导 + 热更新

---

*文档更新时间: 2025-11-08*
*版本: 1.0.0*
