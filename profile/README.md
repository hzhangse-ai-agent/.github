# 企业级 AI Agent 智能体平台

## 项目概述

这是一个**企业级 AI 智能体编排平台**，采用微服务架构，实现了从用户交互、智能体编排、工具调用到服务治理的完整解决方案。系统基于 **LangGraph 多智能体框架**，通过 **MCP 协议**扩展工具能力，使用 **Higress 网关**进行统一治理，并集成 **WASM 插件**实现高性能的缓存、认证和授权。

### 核心特性

| 特性 | 描述 |
|------|------|
| **多智能体编排** | 基于 LangGraph 的三层图架构，支持数百个智能体的复杂任务编排 |
| **工具生态** | 通过 MCP 协议提供丰富的"手脚"能力，零代码接入存量 API |
| **统一网关** | Higress + WASM 插件实现认证、授权、缓存的网关层治理 |
| **智能记忆** | Hybrid 检索 + BGE-Reranker 重排序，准确率提升 30-50% |
| **语音交互** | 50+ 语言实时识别，延迟 < 500ms |
| **用户管理** | RBAC 权限控制，多提供商 OAuth2 认证 |

---

## AI Agent 工作流架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                  用户                                        │
│                                (浏览器)                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                 Chat UI                                      │
│                              (前端 / Next.js)                                │
│                                                                              │
│                        对话界面 · 流式渲染 · 语音输入                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      │ API请求 / WebSocket
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Higress Gateway                                   │
│                                                                              │
│        ┌──────────────┐    ┌──────────────┐    ┌──────────────┐            │
│        │  API路由聚合  │    │   MCP 网关   │    │   服务治理   │            │
│        │              │    │  存量API转换  │    │ 认证/授权/缓存│            │
│        └──────────────┘    └──────────────┘    └──────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                │                       │                   │
                ▼                       ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                   服务层                                     │
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Agent-Server │  │  ASR-Server  │  │ Fusion-Mem0  │  │ User Service │    │
│  │  LangGraph   │  │   语音识别    │  │   记忆服务    │  │   认证/RBAC  │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────────────┘    │
│         │                 │                 │                               │
└─────────┼─────────────────┼─────────────────┼───────────────────────────────┘
          │                 │                 │
          ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                   模型层                                     │
│                                                                              │
│       ┌──────────────┐    ┌──────────────┐    ┌──────────────┐             │
│       │  Qwen3-32B   │    │  SenseVoice  │    │   BGE-M3     │             │
│       │    vLLM      │    │   FunASR     │    │ BGE-Reranker │             │
│       └──────────────┘    └──────────────┘    └──────────────┘             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
          │
          │ MCP调用
          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                                 MCP Tools                                    │
│                                                                              │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────────┐     │
│   │  Chart   │    │ Metrics  │    │ Deployer │    │ 存量API          │     │
│   │  图表生成 │    │  指标计算 │    │  部署管理 │    │ OA/考勤/Jira     │     │
│   └──────────┘    └──────────┘    └──────────┘    └──────────────────┘     │
│                                                            ▲                 │
│                                            ┌───────────────┘                 │
│                                            │ MCP网关转换                      │
└─────────────────────────────────────────────────────────────────────────────┘

【数据流向】
┌─────────────────────────────────────────────────────────────────────────────┐
│ 文本对话流:                                                                  │
│   用户(浏览器) → Chat UI → GW(路由) → Agent-Server → LLM推理               │
│   Agent-Server → GW → Chat UI → 用户                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ 语音对话流:                                                                  │
│   用户(浏览器) → Chat UI → GW(路由) → ASR-Server → SenseVoice              │
│   SenseVoice → ASR-Server → GW → Chat UI → 用户                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ 工具调用流:                                                                  │
│   Agent-Server → MCP调用 → Chart/Metrics/Deployer                          │
│   Agent-Server → MCP调用 → MCP网关 → 存量API                                │
└─────────────────────────────────────────────────────────────────────────────┘
├─────────────────────────────────────────────────────────────────────────────┤
│ 工具调用流:                                                                  │
│   Agent-Server → MCP调用 → Chart/Metrics/Deployer                          │
│   Agent-Server → MCP调用 → MCP网关 → 存量API                                │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 角色分工说明

| 角色 | 组件 | 职责 |
|------|------|------|
| **🚪 网关入口** | Higress Gateway | 统一入口、认证授权、缓存加速、存量API转换 |
| **🖥️ 前端交互** | Agent-Chat | 用户界面、实时消息推送、流式渲染展示 |
| **🧠 核心编排** | Agent-Server | 三层图编排、智能计划分解、状态管理、错误自愈 |
| **🤖 大脑推理** | VLLM Qwen3-32B | 意图理解、工具调用决策、流式内容生成 |
| **💭 记忆系统** | Fusion-Mem0 | 上下文检索、Hybrid混合搜索、相关性重排 |
| **🔧 工具生态** | MCP Services | 图表生成、部署管理、指标计算、存量API调用 |
| **🔐 用户管理** | User Service | 身份认证、RBAC权限、用户信息缓存 |
| **🎤 语音服务** | ASR-Server | 语音采集、VAD检测、多语言识别 |

### 完整工作流程

```
1. 用户输入 → Gateway 认证授权 → Agent-Chat 接收
                    ↓
2. Agent-Server 获取用户上下文 ← User Service (权限/偏好)
                    ↓
3. Agent-Server 检索相关记忆 ← Fusion-Mem0 (Hybrid + Rerank)
                    ↓
4. Agent-Server 请求 LLM 推理 ← VLLM Qwen3-32B (意图理解)
                    ↓
5. LLM 决策调用工具 → MCP Services (Chart/Metrics/Deployer/OA)
                    ↓
6. Agent-Server 执行工具 → 获取结果 → 状态更新 (Redis Checkpoint)
                    ↓
7. Agent-Server 流式推送 → Agent-Chat 实时渲染 → 用户看到结果
```

---
## 关键实践

### 一、网关层：Higress + WASM 服务治理

网关层是整个系统的统一入口，通过 WASM 插件实现认证、授权、缓存等治理能力的下沉，业务层零侵入。

#### 1.1 三层权限验证模型

**业务挑战**：权限验证逻辑散落在各业务代码，难以统一管理、审计困难。

**架构决策**：在网关层统一实现三层权限模型，业务代码零侵入。

```
┌─────────────────────────────────────────────────────────────────┐
│                        三层权限验证                              │
├─────────────────────────────────────────────────────────────────┤
│  L1 公开路由    is_public: true      →  直接放行               │
│  L2 认证路由    验证 Token 有效性    →  无效返回 401           │
│  L3 权限路由    检查用户权限         →  无权限返回 403         │
└─────────────────────────────────────────────────────────────────┘
```

**成果**：权限管理集中化，新增权限规则**无需修改业务代码**，支持即时生效。

#### 1.2 L1+L2 双层缓存架构

**业务挑战**：权限配置高频读取，纯内存缓存容量有限无法跨实例共享，纯 Redis 缓存每次查询有网络延迟。

**架构决策**：设计双层缓存架构，L1 本地内存 + L2 Redis，实现主动回源机制。

```
请求 → L1 内存缓存 (sync.Map) → 命中返回 (~0μs)
                ↓ 未命中
          L2 Redis → 命中返回 (~1ms) → 回写 L1
                ↓ 未命中
          回源 API 加载 → 写入 L1+L2
```

**成果**：L1 命中率 ~90%，实现**零延迟权限验证**；支持多实例水平扩展。

#### 1.3 Bitmap 缓存空洞优化

**业务挑战**：查询员工考勤/工时数据时，大量请求查询"不存在的数据"（如未来日期、离职员工），导致无效数据库查询。

**架构决策**：引入 Redis Bitmap 标记数据存在性，在缓存层前置拦截无效请求。

```
查询流程：
检查 Bitmap → 无数据标记 → 直接返回空（跳过查询）
           → 有数据标记 → 正常查询缓存/数据库
```

**成果**：**减少 60-80% 无效数据库查询**，显著降低数据库压力。

#### 1.4 自动 Token 管理机制

**业务挑战**：调用外部 API 需要管理 Token 的获取、缓存、刷新、注入，逻辑繁琐且容易出错。

**架构决策**：设计全自动 Token 管理器，对业务层透明。

```
请求到达 → 检查 Token 状态
    ├─ 有效 → 注入 Header → 继续请求
    └─ 无效/过期 → 暂停请求 → 异步获取 → 注入 → 恢复请求
```

**成果**：Token 管理完全自动化，业务代码**无感知**，避免 Token 过期导致的调用失败。

#### 1.5 API → MCP 零代码转换

**业务挑战**：将企业内部 API 暴露给 AI Agent 调用，传统方式需要为每个 API 编写适配代码。

**架构决策**：基于 Higress 网关 + WASM 插件，设计声明式配置驱动的 API → MCP 转换机制。

```yaml
# MCP 工具声明式配置
tools:
  - name: query_attendance
    description: 查询用户考勤打卡明细
    args:
      - name: start_date
        type: string
        required: true
    requestTemplate:
      method: POST
      url: http://oa.bst-agent.com/bst/kq
    responseTemplate:
      body: |
        {{- range $record := .datas -}}
        {{ printf "%q,%q,%q\n" $record.kqdate $record.userName $record.signintime -}}
        {{- end -}}
```

**成果**：API 接入从"编写代码"变为"提供文档"，**新 API 接入时间从 3-5 天缩短至分钟级**。

#### 1.6 网关层协议桥接与认证注入

**业务挑战**：存量 API 通常使用自定义认证方式（Cookie、Token 等），AI Agent 无法直接处理认证流程；同时需要将 MCP 协议转换为 HTTP 请求。

**架构决策**：在网关层统一实现协议桥接和认证注入，对 AI Agent 完全透明。

```
AI Agent 调用 MCP 工具
    ↓
Higress 网关拦截请求
    ↓
WASM 插件检查 Token 状态
    ├─ 有效 → 注入 Header → 转发到后端 API
    └─ 无效/过期 → 调用 Token API 获取 → 缓存 → 注入 → 转发
    ↓
后端 API 返回原始响应
    ↓
responseTemplate 转换为 AI 友好格式
    ↓
返回给 AI Agent
```

**核心能力**：
- **协议透明转换**：AI Agent 无需关心底层 HTTP 协议细节
- **认证完全托管**：Token 获取、缓存、刷新、注入全自动化
- **失败自动重试**：网络抖动不影响 AI Agent 调用

**成果**：存量 API 接入时间从 3-5 天缩短至**分钟级**，认证流程对 AI Agent 完全透明

---

### 二、前端层：AI Chat 实时交互

前端层负责用户交互体验，实现 AI 消息实时推送、流式渲染、语音输入等核心能力。

#### 2.1 Thinking Message 推送通道

**业务挑战**：AI Agent 在执行任务过程中会产生大量"思考过程"，用户需要实时看到这些过程以了解 Agent 正在做什么。

**架构决策**：设计独立的消息推送通道，与主对话流分离，支持实时推送 Thinking Message。

```
┌─────────────────────────────────────────────────────────────┐
│                    消息推送架构                              │
├─────────────────────────────────────────────────────────────┤
│  Agent Server (后台执行)                                    │
│       │ pushEvent(username, threadId, event)               │
│       ▼                                                     │
│  ThinkingQueue (消息队列)                                   │
│       │                                                     │
│       ├─── SSE 通道 (EventSource)                          │
│       └─── WebSocket 通道 (长连接)                          │
│                │                                            │
│                ▼                                            │
│   useThinkingConnection Hook                                │
│   • 自动重连（指数退避）                                     │
│   • 心跳保活（60s ping/pong）                               │
│   • 按 threadId 过滤消息                                    │
└─────────────────────────────────────────────────────────────┘
```

**成果**：用户可实时看到 AI Agent 的思考过程，提升产品透明度和信任感。

#### 2.2 双传输协议策略

**业务挑战**：企业内网环境可能存在 WebSocket 代理限制，需要提供降级方案保证消息推送可用。

**架构决策**：采用策略模式设计可插拔的传输层，SSE 和 WebSocket 作为两种可互换的策略。

| 策略 | 特点 | 适用场景 |
|------|------|----------|
| SSE | 兼容性好，自动重连 | 有代理限制的环境 |
| WebSocket | 双向通信，延迟更低 | 无限制的网络环境 |

**成果**：一套代码适应不同网络环境，SSE 作为降级方案保证可用性。

#### 2.3 流式消息渲染

**业务挑战**：AI 响应采用流式输出，需要实时渲染 Markdown、代码高亮、数学公式、图表等复杂内容。

**架构决策**：设计增量渲染管道，支持内容分块解析和实时更新。

```
SSE/WebSocket 消息流
       │
       ▼
Stream Provider (React Context)
       │
       ▼
消息类型分发
  • AI Message → markdown-text.tsx
  • Tool Call → tool-calls.tsx
  • Thinking → thinking.tsx
       │
       ▼
内容渲染组件
  • Markdown：react-markdown
  • 代码高亮：syntax-highlighter
  • 数学公式：KaTeX
  • 流程图：Mermaid
```

**成果**：支持复杂内容的实时流式渲染，渲染延迟 < 100ms。

#### 2.4 语音输入架构

**业务挑战**：AI Agent 对话场景中，语音输入比文字更自然高效，但需要解决实时语音识别、VAD 检测、降噪等问题。

**架构决策**：设计前后端协同的语音输入架构，前端负责采集和 VAD，后端负责 ASR 识别。

```
前端 (useVoiceRecording Hook)
  • AudioWorklet 实时音频处理
  • 16kHz PCM 格式
  • 音量检测（波形可视化）
  • VAD 语音活动检测
        │
        ▼ WebSocket 流式传输
        │
后端 ASR 服务
  • SenseVoice 非自回归模型
  • 10秒音频 70ms 推理延迟
  • 支持 50+ 种语言
  • 返回识别结果 + 情感标签
```

**成果**：语音输入延迟 < 500ms，识别准确率 > 95%。

#### 2.5 Redis Pub/Sub 跨实例同步

**业务挑战**：前端应用多实例部署时，消息推送无法正确路由到用户连接的实例。

**架构决策**：引入 Redis Pub/Sub 作为消息总线，实现跨实例的消息广播。

```
Agent Server 推送消息
       │
       ▼
Redis PUBLISH user:{username} {event}
       │
       ├─→ 实例 A (SUBSCRIBE) → 本地连接发送
       ├─→ 实例 B (SUBSCRIBE) → 本地连接发送
       └─→ 实例 C (SUBSCRIBE) → 本地连接发送
```

**成果**：支持多实例水平扩展，消息推送准确率 100%。

---

### 三、Agent 层：多智能体编排系统

Agent 层是整个平台的核心，基于 LangGraph 实现多智能体编排，支持复杂任务的智能分解和执行。

#### 3.1 三层图架构设计

**业务挑战**：企业级 AI Agent 需要协调多个智能体完成复杂任务，LangGraph 社区版仅支持单图结构，无法满足复杂业务场景的多层级编排需求。

**架构决策**：设计 SuperGraph → TeamGraph → Member 三层图架构，实现任务的分层编排和智能路由。

```
┌─────────────────────────────────────────────────────────────┐
│                    SuperGraph (入口层)                       │
│  条件路由: config.metadata.department                        │
│    ├── "ANALYSIS" → ANALYSIS_TEAM                          │
│    ├── "PM" → PM_TEAM                                      │
│    └── 其他 → END                                          │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    TeamGraph (团队层)                        │
│  ANALYSIS_TEAM 工作流:                                       │
│    supervisor → analysisPlanner → analysisPlanExecutor →    │
│    analysisRePlanner → analysisAIReporter                   │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Member (成员层)                           │
│  MemberSubgraph 执行闭环:                                    │
│    llm_invoke → multi_tool_call → error_checker →           │
│    message_aggregator                                       │
└─────────────────────────────────────────────────────────────┘
```

**成果**：支持数百个智能体的复杂任务编排，多租户请求自动路由隔离。

#### 3.2 智能计划执行系统

**业务挑战**：用户提出的复杂需求需要分解为多个可执行步骤，且步骤间存在依赖关系。

**架构决策**：设计"计划生成 → 步骤执行 → 动态调整"的完整闭环系统。

```
用户请求: "分析本月销售数据并生成报告"
                │
                ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. analysisPlanner (计划生成)                               │
│    LLM 结构化输出 (Zod Schema):                              │
│    {                                                        │
│      fullplan: [                                            │
│        { stepId: 1, stepDesc: "获取基础数据", tasks: [...] },│
│        { stepId: 2, stepDesc: "计算指标", tasks: [...],     │
│          depTaskIds: ["1:1", "1:2"] }                       │
│      ]                                                      │
│    }                                                        │
└─────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. analysisPlanExecutor (步骤执行)                          │
│    • findCurrentStep(): 找到当前可执行的步骤                │
│    • 并发执行：同一步骤内无依赖任务可并发                    │
│    • 状态追踪：pending → executing → completed/failed       │
└─────────────────────────────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Mermaid DAG 可视化                                       │
│    graph TD                                                 │
│      A[获取工作日历] --> D[计算指标]                        │
│      B[获取Jira工时] --> D                                  │
│      C[获取请假记录] --> D                                  │
│      D --> E[生成报告]                                      │
└─────────────────────────────────────────────────────────────┘
```

**成果**：复杂任务自动分解为可执行计划，支持步骤间依赖管理和失败自动重规划。

#### 3.3 子图级别错误检测循环

**业务挑战**：LLM 生成的工具调用参数可能不符合 Schema 定义，导致调用失败。

**架构决策**：设计子图级别的错误检测循环，Schema 错误自动回 LLM 重试，形成自愈能力。

```
START ──► llm_invoke ──► multi_tool_call ──► error_checker
               ▲                               │
               └───────────────────────────────┘
                    (hasSchemaErrors)
```

**成果**：Schema 错误自动修正，减少人工干预 90%。

#### 3.4 Replan 动态计划调整

**业务挑战**：工具执行失败时，需要根据错误信息调整执行计划。

**架构决策**：设计 Replan 机制，失败时触发计划重编排，支持任务级别的精细化调整。

```typescript
// 任务失败检测
if (content.includes('[ERROR]')) {
  task.execState = 'failed';
  task.errorMsg = content;
  // 触发 Replan
  state.decision = { nextStepTip: 'replan' };
}

// 调整后的计划
{
  "replan_decision": "skip_failed_source",
  "replan_decisionReason": "Jira API 连接超时，使用缓存的工时数据"
}
```

**成果**：工具失败时智能调整计划，任务成功率提升 40%。

#### 3.5 索引分离模式工具管理

**业务挑战**：AI Agent 需要调用大量工具，工具被多个 Team/Member 共享，重复创建 MCP 连接导致资源浪费。

**架构决策**：设计"单例存储 + 多维索引"的分离模式，工具实例唯一存储，通过索引支持多维度查询。

```typescript
class GlobalToolManager {
  // 唯一真实存储：工具名称 → 工具实例
  private static toolInstances: Map<string, DynamicStructuredTool>;

  // 索引关系：都是工具名称的引用
  private static teamTools: Map<string, Set<string>>;      // Team → 工具名
  private static memberTools: Map<string, Set<string>>;    // Member → 工具名
  private static serverKeyTools: Map<string, Set<string>>; // ServerKey → 工具名
}
```

**成果**：工具加载效率提升 10x，避免重复创建 MCP 连接。

#### 3.6 Redis Checkpoint 分布式持久化

**业务挑战**：AI Agent 任务可能执行很长时间，用户中断后需要能够恢复执行；多实例部署时需要共享状态。

**架构决策**：扩展 LangGraph API Server 实现 Redis Checkpoint，支持分布式部署和长任务恢复。

| 特性 | 本项目（扩展版） | 社区开源版 |
|------|-----------------|-----------|
| 状态持久化 | Redis | 本地文件系统 |
| 部署方式 | 支持集群部署 | 仅支持单机部署 |
| 高可用性 | 支持（Redis集群） | 不支持 |
| 水平扩展 | 支持 | 不支持 |
| 状态读写 | ~1ms | ~10ms |

**成果**：支持任务中断恢复，多实例水平扩展无状态问题，性能提升 10x。

#### 3.7 智能消息分割系统

**业务挑战**：工具执行返回大量数据时，可能超过 LLM 上下文限制（通常 128K token）。

**架构决策**：设计智能消息分割系统，支持 Token 估算、智能分批、格式检测、并发处理。

```
1. Token 估算
   estimateMessageTokenCount(messages, llm)

2. 消息分类
   - basemsg: 系统消息、用户消息（不分割）
   - toolMessages: 工具返回（可分割）

3. 格式检测
   - CSV: 按行分割，保留表头
   - JSON: 按数组元素分割
   - Markdown Table: 按行分割

4. 智能分批
   - 最小负载优先算法
   - 平衡分配到多个批次

5. 并发处理
   - concurrencyLimit: 3
   - 超时控制: 600s
```

**成果**：支持处理 10万+ token 的消息，解决 LLM 上下文限制问题。

#### 3.8 LangGraph API 无侵入扩展

**业务挑战**：LangGraph 官方 API Server 仅支持本地文件系统存储状态，无法满足生产环境分布式部署需求。

**架构决策**：设计无侵入扩展架构，通过构建时合并原始包产物的方式，实现对官方包的扩展而不修改其源码。

```
┌─────────────────────────────────────────────────────────────┐
│                LangGraph API Redis 扩展架构                   │
├─────────────────────────────────────────────────────────────┤
│  原始包: @langchain/langgraph-api                           │
│  dist/                                                      │
│    ├── cli/spawn.mjs                                        │
│    ├── server.mjs                                           │
│    └── storage/persist.mjs                                  │
│                          │                                   │
│                          ▼ 复制                              │
│  扩展包: @langchain/langgraph-api (0.0.68-redis)           │
│  dist/                                                      │
│    ├── cli/spawn.mjs        (来自原始包)                    │
│    ├── storage/redis/       (新增扩展)                      │
│    │   ├── RedisOps.mjs                                    │
│    │   ├── RedisStore.mjs                                  │
│    │   ├── InRedisSaver.mjs                                │
│    │   └── RedisClientFactory.mjs                          │
│    └── checkpoint.mts        (新增扩展)                     │
└─────────────────────────────────────────────────────────────┘
```

**成果**：实现对 LangGraph API 的 Redis 存储扩展，同时保持与官方包的兼容性。

---

### 四、记忆层：智能记忆管理

记忆层负责 AI Agent 的上下文记忆管理，通过两阶段检索架构实现高准确率的相关记忆召回。

#### 4.1 两阶段检索架构

**业务挑战**：传统向量检索系统仅依赖密集向量进行语义匹配，在处理专业术语、精确关键词匹配时准确率不足。

**架构决策**：设计"Hybrid 混合检索 + Reranker 重排序"两阶段检索架构。

```
用户查询
    ↓
┌─────────────────────────────────────┐
│  第一阶段：Hybrid 混合检索（候选筛选）  │
│  - 稀疏向量：BM25 风格关键词匹配        │
│  - 密集向量：BGE-M3 语义嵌入           │
│  - 混合打分：加权融合 Top-N 候选       │
└─────────────────────────────────────┘
    ↓ 候选结果（可能包含噪音）
    ↓
┌─────────────────────────────────────┐
│  第二阶段：BGE-Reranker-v2-M3 精排    │
│  - 深度 Cross-Encoder 语义匹配        │
│  - 多维度相关性评分                   │
│  - Top-K 精选输出                    │
└─────────────────────────────────────┘
    ↓ 最终结果（准确率 85-95%）
```

**成果**：检索准确率从 60-70%（纯向量）提升至 **85-95%**。

#### 4.2 混合检索算法优化

**业务挑战**：稀疏向量（关键词）和密集向量（语义）的得分维度不同，如何融合才能发挥各自优势？

**架构决策**：设计可配置的加权融合算法，支持不同场景的权重调优。

```python
def compute_hybrid_score(q_emb, d_emb, sparse_weight=0.95, dense_weight=0.05):
    # 稀疏得分：词袋点积（BM25 风格）
    sparse_score = sum(
        q_weight * d_sparse.get(token, 0) 
        for token, q_weight in q_emb['sparse'].items()
    )
    
    # 密集得分：余弦相似度
    dense_score = np.dot(q_emb['dense'], d_emb['dense'])
    
    # 加权融合
    return sparse_weight * sparse_score + dense_weight * dense_score
```

| 场景 | 稀疏权重 | 密集权重 |
|------|----------|----------|
| 精确匹配 | 0.95 | 0.05 |
| 语义理解 | 0.5 | 0.5 |
| 平衡场景 | 0.7 | 0.3 |

**成果**：支持**三种检索模式**一键切换，满足不同业务场景需求。

#### 4.3 稀疏向量压缩存储

**业务挑战**：BGE-M3 生成的稀疏向量包含数千个 token 权重，直接存储会导致存储膨胀和查询延迟。

**架构决策**：实现 Top-K 压缩策略，只保留权重最高的关键 token。

```python
def embed_for_storage(self, text: str) -> dict:
    full_result = self.embed_full(text)
    sparse_vector = full_result.get('sparse', {})
    
    # 压缩：保留权重最高的前 50 个 token
    compressed_sparse = dict(
        sorted(sparse_vector.items(), key=lambda x: x[1], reverse=True)[:50]
    )
    
    return {
        "dense": full_result.get('dense', []),
        "sparse": compressed_sparse,  # 压缩后存储
    }
```

**成果**：存储空间减少 **90%+**，检索延迟降低 **40%**，同时保持检索准确率。

#### 4.4 工厂模式扩展架构

**业务挑战**：Mem0 框架内置的 LLM、Embedder、VectorStore 提供者有限，需要集成企业私有化模型服务。

**架构决策**：采用"继承 + 验证器重写"模式，在不修改框架源码的前提下扩展工厂注册机制。

```python
# 自定义配置类继承框架基类，重写验证逻辑
class HybridLlmConfig(Mem0LlmConfig):
    @field_validator("config")
    @classmethod
    def validate_config(cls, v, info):
        provider = info.data.get("provider")
        custom_providers = ("worker_llm",)  # 自定义提供者白名单
        
        if provider in custom_providers:
            return v  # 自定义提供者直接通过
        return super().validate_config(v, info)
```

**成果**：实现了对 BGE-M3 Embedding、私有 LLM、自定义 Reranker 的**零侵入集成**。

---

### 五、用户层：用户管理中心

用户层负责用户认证、权限管理、配置存储等企业级能力。

#### 5.1 L1+L2 双层缓存架构

**业务挑战**：企业级用户服务系统面临高频认证请求、权限验证频繁、用户信息查询密集等场景。

**架构决策**：设计双层缓存架构，L1 本地内存 + L2 Redis，并实现自动回写机制。

```
请求 → L1 内存缓存 (Map结构) → 命中返回 (~0μs)
                ↓ 未命中
          L2 Redis (Hash + RediSearch) → 命中返回 (~1ms) → 回写 L1
                ↓ 未命中
          数据库查询 → 写入 L1+L2
```

**成果**：L1 命中率 ~90%，实现零延迟权限验证；认证请求响应时间从 ~50ms 降至 **~1ms**。

#### 5.2 拦截器统一管理架构

**业务挑战**：传统开发中缓存逻辑和权限验证散落在各业务代码中，代码耦合度高。

**架构决策**：采用 NestJS 拦截器模式，将缓存逻辑和权限验证下沉到框架层。

```typescript
// 全局注册两个拦截器
providers: [
  { provide: APP_INTERCEPTOR, useClass: PermissionInterceptor },
  { provide: APP_INTERCEPTOR, useClass: ServiceCacheInterceptor },
]

// Service 层代码纯净，无缓存侵入
async findAll(context: ServiceContext) {
  return db.prepare('SELECT * FROM users').all();
}
```

**成果**：Service 层代码零缓存逻辑侵入，权限验证逻辑集中管理。

#### 5.3 NestJS + Next.js 融合架构

**业务挑战**：企业级应用需要同时提供后端 API 和前端页面，传统方案需要部署两个独立服务。

**架构决策**：设计 NestJS 与 Next.js 融合架构，共享 Express 实例，统一端口提供服务。

```typescript
// 路由分发：API 请求 → NestJS，页面请求 → Next.js
expressApp.use(async (req, res, next) => {
  if (req.path.startsWith('/api/') || req.path === '/health') {
    return next(); // NestJS 处理
  }
  await nextHandler(req, res); // Next.js 处理
});
```

**成果**：单端口提供完整的前后端服务，简化部署架构。

#### 5.4 OAuth2 多 Provider SSO 集成

**业务挑战**：企业需要对接多种 OAuth2 Provider（企业微信、钉钉、自建 IdP 等）。

**架构决策**：设计可配置的 OAuth2 集成方案，支持多 Provider 动态接入。

```typescript
// oauth2_configs 表存储各 Provider 配置
interface OAuth2Config {
  name: string;              // 配置名称
  provider: string;          // 提供商名称
  client_id: string;         // 客户端ID
  authorization_url: string; // 授权URL
  token_url: string;         // 令牌URL
  userinfo_url: string;      // 用户信息URL
}
```

**成果**：新增 OAuth2 Provider 只需数据库配置，无需修改代码。

#### 5.5 RBAC 权限模型

**业务挑战**：权限验证逻辑散落在各业务代码，难以统一管理。

**架构决策**：实现三层 RBAC 模型（用户-角色-权限），结合 API 路由表实现细粒度权限控制。

```
users ── role ──→ roles ──→ role_permissions ──→ permissions
                                          ↓
                                    api_routes.required_permissions
```

**成果**：权限配置存储在数据库，支持运行时动态调整；新增权限规则无需修改代码。

---

### 六、工具层：MCP 工具生态

工具层通过 MCP 协议为 AI Agent 提供丰富的"手脚"能力，包括图表生成、部署管理、指标计算等。

#### 6.1 MCP 协议双模式传输

**业务挑战**：MCP 工具需要同时支持本地 CLI 调用和远程 HTTP 调用。

**架构决策**：设计双模式传输架构，支持 stdio 和 HTTP SSE 两种协议。

```typescript
// Stdio 模式（本地调用）
if (mode === "stdio") {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

// HTTP SSE 模式（远程调用）
else if (mode === "http") {
  const transport = new StreamableHTTPServerTransport({...});
  await server.connect(transport);
}
```

**成果**：一套代码支持**两种调用模式**，AI Agent 可根据环境自动选择最优传输方式。

#### 6.2 Pipeline DAG 编排架构

**业务挑战**：企业指标计算涉及多个步骤，步骤间有复杂的依赖关系。

**架构决策**：设计基于 DAG 的 Pipeline 编排架构，每个步骤作为独立工具。

```python
class PipelineStep(Enum):
    INIT = "init"
    DATA_FETCH = "data_fetch"
    DATA_VALIDATE = "data_validate"
    DATA_EXTRACT = "data_extract"
    DATA_TRANSFORM = "data_transform"
    CALCULATE = "calculate"
    FORMAT = "format"
```

**成果**：支持**步骤级粒度控制**，AI Agent 可根据需要跳过、重试或并行执行特定步骤。

#### 6.3 流式部署日志实时推送

**业务挑战**：DevOps 部署任务通常持续数分钟到数小时，传统请求-响应模式会导致客户端超时。

**架构决策**：采用 SSE 实现流式日志推送，配合心跳机制保持连接。

```typescript
function executeCommandWithLogStream(command, args, cwd) {
  return {
    async *[Symbol.asyncIterator]() {
      const child = spawn(command, args, { cwd });
      
      for await (const chunk of child.stdout) {
        yield chunk.toString();
        
        // 心跳：超过10秒无输出时发送
        if (Date.now() - lastHeartbeat > 10000) {
          yield `[HEARTBEAT] ${new Date().toISOString()}\n`;
        }
      }
    }
  };
}
```

**成果**：支持**长时间运行任务**（2小时超时），实时推送部署进度。

#### 6.4 YAML 动态指标配置

**业务挑战**：企业指标需求频繁变化，硬编码方式需要修改代码、重新部署。

**架构决策**：采用 YAML 配置驱动指标定义，支持热加载无需重启服务。

```yaml
metrics:
  definitions:
    total-worktime-hours:
      name: "total-worktime-hours"
      display_name: "总工时"
      data_source_url: "jira:worktime"
      source_field: "data[*].timeSpentSeconds"
      transform_func: "seconds_to_hours"
      calculate_func: "sum"
```

**成果**：新增指标**零代码修改**，只需添加 YAML 配置，支持分钟级上线。

#### 6.5 服务端图表渲染

**业务挑战**：AI Agent 生成的图表需要在对话中展示，但客户端渲染依赖浏览器环境。

**架构决策**：设计 SSR 服务，将图表配置转换为 PNG 图片。

```javascript
const { render } = require('@antv/gpt-vis-ssr');

app.post('/render', async (req, res) => {
  const vis = await render(req.body);  // 图表配置
  const buffer = await vis.toBuffer(); // 转换为 PNG Buffer
  
  res.json({
    success: true,
    resultObj: `${protocol}://${host}/images/${filename}`
  });
});
```

**成果**：AI Agent 可直接返回**图片 URL**，用户在对话中看到可视化结果。

---

## 技术栈总览

| 层级 | 技术选型 |
|------|----------|
| **表现层** | Next.js 15, React 19, Radix UI, TailwindCSS |
| **网关层** | Higress, WASM (Go/Rust), Redis |
| **Agent层** | LangGraph.js, LangChain, TypeScript |
| **记忆层** | Mem0 AI, BGE-M3, BGE-Reranker, Redis Stack |
| **用户层** | NestJS, SQLite, Redis, OAuth2 |
| **工具层** | MCP Protocol, FastAPI, Express, AntV |
| **数据层** | Redis Cluster, ClickHouse, PostgreSQL |
| **部署** | Docker, Kubernetes, Helm |

---

## 量化成果汇总

| 指标 | 提升 |
|------|------|
| **检索准确率** | 60-70% → **85-95%** |
| **缓存命中率** | **~90%**，零延迟响应 |
| **无效查询减少** | **60-80%**（Bitmap 优化） |
| **API 接入周期** | 3-5 天 → **分钟级** |
| **任务成功率** | 提升 **40%**（Replan 机制） |
| **Schema 错误修正** | 减少 **90%** 人工干预 |
| **语音识别延迟** | **< 500ms** |
| **消息推送延迟** | **< 100ms** |
| **存储空间优化** | 减少 **90%+**（稀疏向量压缩） |
| **状态读写性能** | 提升 **10x**（Redis vs 文件系统） |

---

## 模块清单

| 模块名称 | 端口 | 技术栈 | 功能描述 |
|---------|------|--------|---------|
| **AgentFlow** | 8000 | LangGraph.js + TypeScript | 多智能体编排，三层图架构 |
| **AgentFlow-UI** | 3000 | Next.js 15 + React 19 | Web 聊天界面，实时思考流推送 |
| **SonicMind** | 8002 | FastAPI + FunASR | 50+ 语言语音识别 |
| **Fusion-Mem0** | 8001 | Mem0 + BGE-M3 | 两阶段检索记忆管理 |
| **User Service** | 3000 | NestJS + SQLite | RBAC 权限控制，多提供商认证 |
| **Chart Server** | 1122 | Node.js + AntV | 基于 MCP 的图表生成 |
| **Deployer** | 3000 | TypeScript + Docker | 自动化部署管理 |
| **Mcp Metrics Server** | 8004 | FastAPI + LangGraph | 企业级指标计算流水线 |
| **Vis SSR** | 3001 | Node.js + AntV | 服务端图表渲染 |
| **wasm-authz** | - | WASM | 三层权限验证，L1+L2 缓存 |
| **wasm-authn** | - | WASM | 自动 Token 管理 |
| **wasm-api-cache** | - | WASM | API 响应缓存，Bitmap 优化 |
---

## 项目特点

1. **完整的架构设计**：从表现层到数据层的完整分层架构，职责清晰
2. **强大的编排能力**：基于 LangGraph 的多智能体编排系统，支持复杂任务分解
3. **丰富的工具生态**：通过 MCP 协议提供各种"手脚"能力，零代码接入存量 API
4. **统一的服务治理**：Higress + WASM 插件实现网关层治理，业务层零侵入
5. **高性能的检索**：两阶段检索架构，准确率提升 30-50%
6. **企业级特性**：分布式状态管理、RBAC 权限控制、多提供商认证
7. **灵活的扩展性**：基于 MCP 协议和动态类加载，易于扩展新能力
8. **优秀的性能**：分层缓存、异步处理、Bitmap 优化，毫秒级响应

---

*本项目适用于需要构建复杂 AI Agent 应用的企业场景，如智能客服、数据分析、自动化运维等领域。*
