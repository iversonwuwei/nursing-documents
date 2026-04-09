# 后端 AI 与基础设施架构设计

## 交付 framing

- scope: 定义 AI Orchestration Service 的多模型设计、缓存策略、消息扩展和三端 BFF 集成方案。
- affected audience: 后端研发、BFF 研发、AI/ML 工程、DevOps、三端前端开发。
- validation: 需通过 `dotnet build nursing-backend-services.slnx` 和 `npm run docs:build`。
- rollback: 回退本文、相关代码变更和索引修改。

---

## 一、现状与目标

### 现状

| 层面 | 状态 |
|------|------|
| AI Orchestration Service | 空壳，仅有平台脚手架，无业务端点、无 AI SDK |
| Redis | compose 已部署 (redis:7, :6379)，代码无使用 |
| RabbitMQ | 消息拓扑成熟（outbox + retry + dead-letter），但仅覆盖 Care/Visit/Billing → Notification |
| 前端 AI 功能 | Admin ~30 个、Nani 3 个、Family 3 个，全部纯前端 mock |
| BFF AI 代理 | 无任何 AI 端点 |

### 目标

1. 将 AiOrchestration Service 实现为统一的多模型 AI 网关
2. 引入 Redis 缓存层，降低重复推理成本，提升响应速度
3. 扩展消息拓扑，支持异步 AI 推理和结果回写
4. 三端 BFF 各暴露所需的 AI 代理端点

---

## 二、多模型策略

### 模型分类与选择

根据 36 个前端 AI 功能的分析，后端 AI 能力归为 5 类，每类使用最适配的模型：

| 能力类别 | 适用模型 | 模型标识 | 典型场景 |
|----------|----------|----------|----------|
| **结构化摘要** | GPT-4o-mini / Claude Haiku | `summarizer` | Dashboard 概览、运营周报、财务解读、今日摘要 |
| **风险分析与解释** | GPT-4o / Claude Sonnet | `analyzer` | 健康风险解释、报警分析、事故复盘、探视风险判断 |
| **分类与评分** | GPT-4o-mini / 本地分类模型 | `classifier` | 入住评估分级、任务优先级、健康风险等级 |
| **对话问答 (RAG)** | GPT-4o + 向量检索 | `chat` | Admin 问答、Family 今日状态问答 |
| **模板生成** | GPT-4o-mini / Claude Haiku | `generator` | 交接班草稿、升级通知、探视建议 |

### 配置化模型注册

所有模型通过 `appsettings.json` 配置，支持热切换：

```json
{
  "AiModels": {
    "Providers": {
      "openai": {
        "ApiKey": "${AI_OPENAI_API_KEY}",
        "BaseUrl": "https://api.openai.com/v1",
        "DefaultModel": "gpt-4o-mini",
        "TimeoutSeconds": 30,
        "MaxRetries": 2
      },
      "anthropic": {
        "ApiKey": "${AI_ANTHROPIC_API_KEY}",
        "BaseUrl": "https://api.anthropic.com/v1",
        "DefaultModel": "claude-sonnet-4-20250514",
        "TimeoutSeconds": 30,
        "MaxRetries": 2
      },
      "local": {
        "BaseUrl": "http://localhost:11434/v1",
        "DefaultModel": "qwen2.5:7b",
        "TimeoutSeconds": 60,
        "MaxRetries": 1
      }
    },
    "Capabilities": {
      "summarizer": { "Provider": "openai", "Model": "gpt-4o-mini", "Temperature": 0.3, "MaxTokens": 1024 },
      "analyzer":   { "Provider": "openai", "Model": "gpt-4o", "Temperature": 0.2, "MaxTokens": 2048 },
      "classifier": { "Provider": "openai", "Model": "gpt-4o-mini", "Temperature": 0.0, "MaxTokens": 512 },
      "chat":       { "Provider": "openai", "Model": "gpt-4o", "Temperature": 0.5, "MaxTokens": 2048 },
      "generator":  { "Provider": "openai", "Model": "gpt-4o-mini", "Temperature": 0.4, "MaxTokens": 1024 }
    }
  }
}
```

### 模型路由逻辑

```text
请求 → AiOrchestration
  → 1. 解析 capability (summarizer/analyzer/classifier/chat/generator)
  → 2. 查找 Capabilities[capability] → provider + model + params
  → 3. 通过统一 ICompletionClient 调用 Providers[provider]
  → 4. 返回结构化结果 + 审计元数据
```

### 降级策略

- Provider 不可用 → 自动降级到 `local` provider（需 Ollama 或兼容 API）
- Provider + local 全不可用 → 返回 `{ available: false, fallbackHint: "..." }` + HTTP 503

---

## 三、缓存架构 (Redis)

### 缓存分层

| 缓存层 | Key 模式 | TTL | 用途 |
|--------|----------|-----|------|
| **AI 推理结果** | `ai:{tenantId}:{capability}:{inputHash}` | 5~30 分钟 | 相同输入的重复推理缓存 |
| **Dashboard 聚合** | `dash:{tenantId}:{section}` | 2 分钟 | Admin Dashboard KPI 和摘要 |
| **BFF 读模型** | `bff:{bffType}:{tenantId}:{dataKey}` | 1~5 分钟 | 高频读场景的响应缓存 |
| **会话上下文** | `session:{tenantId}:{userId}:{topic}` | 30 分钟 | AI 对话多轮上下文 |
| **配置快照** | `config:{tenantId}:snapshot` | 10 分钟 | 配置服务快照 |

### 配置

```json
{
  "Redis": {
    "ConnectionString": "${REDIS_CONNECTION_STRING:localhost:6379}",
    "InstanceName": "nursing:",
    "DefaultDatabase": 0,
    "ConnectTimeout": 5000,
    "SyncTimeout": 3000,
    "AbortOnConnectFail": false
  },
  "CacheTtl": {
    "AiInferenceMinutes": 15,
    "DashboardSeconds": 120,
    "BffReadModelSeconds": 180,
    "SessionMinutes": 30,
    "ConfigSnapshotMinutes": 10
  }
}
```

### 缓存策略

- **Cache-Aside**: BFF 和 AI 推理场景，先查 Redis，miss 时查下游并回填
- **Write-Through Invalidation**: Config 写入时删除对应缓存 key
- **事件驱动失效**: 领域事件产生时通过 RabbitMQ 消费者清除相关缓存

---

## 四、消息扩展

### 现有拓扑 (保持不变)

```text
nursing.domain.events (topic exchange)
  ├── care.CarePlanGenerated   → nursing.notification.events
  ├── visit.VisitRequested     → nursing.notification.events
  └── billing.InvoiceIssued    → nursing.notification.events
```

### 新增拓扑

```text
nursing.domain.events (topic exchange) — 新增 routing key
  ├── elder.ElderAdmitted        → nursing.notification.events
  ├── health.VitalRecorded       → nursing.ai.events (新队列)
  ├── health.HealthRiskDetected  → nursing.ai.events
  ├── operations.AlertRaised     → nursing.notification.events
  └── staffing.ShiftAssigned     → nursing.notification.events

nursing.ai.events (新队列)
  → AI Worker 消费: 自动触发健康风险分析 + 结果缓存
  → 失败: nursing.ai.events.retry (TTL=30s) → 回到主 exchange
  → 三次失败: nursing.ai.events.dead
```

### 新增 EventWorker 消费者

```text
AiEventConsumerWorker:
  health.VitalRecorded → 调用 AI Orchestration /api/ai/health-risk → 结果写入 Redis 缓存
  health.HealthRiskDetected → 调用 AI Orchestration /api/ai/alert-suggestion → 结果写入缓存
```

### 配置

```json
{
  "RabbitMQ": {
    "HostName": "${RABBITMQ_HOSTNAME:localhost}",
    "Port": 5672,
    "UserName": "${RABBITMQ_USERNAME:guest}",
    "Password": "${RABBITMQ_PASSWORD:guest}",
    "VirtualHost": "/",
    "PrefetchCount": 10,
    "RetryMaxCount": 3,
    "RetryDelayMs": 30000
  }
}
```

---

## 五、AI Orchestration Service 端点设计

### 端点概览

所有端点统一返回 `AiResult<T>` 信封：

```json
{
  "available": true,
  "capability": "analyzer",
  "provider": "openai",
  "model": "gpt-4o",
  "result": { /* 结构化结果 */ },
  "cached": false,
  "latencyMs": 342,
  "traceId": "...",
  "auditId": "..."
}
```

### Admin 场景端点

| 端点 | 方法 | 能力 | 说明 |
|------|------|------|------|
| `/api/ai/dashboard-insights` | POST | summarizer | Dashboard 运营摘要 |
| `/api/ai/health-risk` | POST | analyzer | 健康风险解释 |
| `/api/ai/alert-suggestion` | POST | analyzer | 报警处理建议 |
| `/api/ai/task-priority` | POST | classifier | 任务优先级排序 |
| `/api/ai/admission-assessment` | POST | classifier | 入住评估分级 |
| `/api/ai/ops-report` | POST | summarizer | 运营周/月报生成 |
| `/api/ai/financial-insights` | POST | summarizer | 财务解读 |
| `/api/ai/device-insights` | POST | analyzer | 设备巡检建议 |
| `/api/ai/incident-analysis` | POST | analyzer | 事故分析复盘 |
| `/api/ai/resource-insights` | POST | summarizer | 资源摘要 (房间/物资/排班等) |
| `/api/ai/chat` | POST | chat | 管理员自然语言问答 |
| `/api/ai/elder-detail-action` | POST | analyzer | 老人详情跟进建议 |

### Nani 场景端点

| 端点 | 方法 | 能力 | 说明 |
|------|------|------|------|
| `/api/ai/shift-summary` | POST | summarizer | 班次摘要 |
| `/api/ai/care-copilot` | POST | analyzer | 任务执行辅助建议 |
| `/api/ai/handover-draft` | POST | generator | 交接班草稿生成 |
| `/api/ai/escalation-draft` | POST | generator | 升级通知草稿生成 |

### Family 场景端点

| 端点 | 方法 | 能力 | 说明 |
|------|------|------|------|
| `/api/ai/today-summary` | POST | summarizer | 今日护理状态摘要 |
| `/api/ai/health-explain` | POST | analyzer | 健康指标家属友好解释 |
| `/api/ai/visit-assistant` | POST | generator | 探视建议与时段推荐 |
| `/api/ai/visit-risk` | POST | classifier | 探视时段风险判断 |
| `/api/ai/family-chat` | POST | chat | 家属自然语言护理问答 |

### 治理端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/ai/rules` | GET | 获取所有 AI 规则开关 |
| `/api/ai/rules/{ruleId}/toggle` | PATCH | 启用/禁用规则 |
| `/api/ai/models/status` | GET | 获取所有模型注册状态 |
| `/api/ai/audit-logs` | GET | AI 推理审计日志查询 |
| `/api/ai/audit-logs/{auditId}` | GET | 单条审计详情 |

---

## 六、审计与日志

### AI 审计记录

每次 AI 调用产生一条审计记录，持久化到 PostgreSQL：

```text
AiAuditLogEntity:
  AuditId         string    PK
  TenantId        string
  UserId          string
  Capability      string    (summarizer/analyzer/classifier/chat/generator)
  Provider        string    (openai/anthropic/local)
  Model           string
  Endpoint        string
  InputHash       string
  InputSizeBytes  int
  OutputSizeBytes int
  Cached          bool
  LatencyMs       int
  Success         bool
  ErrorMessage    string?
  CreatedAtUtc    DateTimeOffset
```

### AI 规则实体

```text
AiRuleEntity:
  RuleId          string    PK
  TenantId        string
  RuleCode        string    (如 admission_assessment, health_risk, task_priority)
  RuleName        string
  Description     string
  Capability      string
  IsEnabled       bool
  Priority        int
  CreatedAtUtc    DateTimeOffset
  UpdatedAtUtc    DateTimeOffset
```

---

## 七、三端 BFF 代理设计

### Admin BFF 新增端点

```text
POST /api/admin/ai/dashboard-insights    → AI /api/ai/dashboard-insights
POST /api/admin/ai/health-risk           → AI /api/ai/health-risk
POST /api/admin/ai/alert-suggestion      → AI /api/ai/alert-suggestion
POST /api/admin/ai/task-priority         → AI /api/ai/task-priority
POST /api/admin/ai/admission-assessment  → AI /api/ai/admission-assessment
POST /api/admin/ai/ops-report            → AI /api/ai/ops-report
POST /api/admin/ai/financial-insights    → AI /api/ai/financial-insights
POST /api/admin/ai/device-insights       → AI /api/ai/device-insights
POST /api/admin/ai/incident-analysis     → AI /api/ai/incident-analysis
POST /api/admin/ai/resource-insights     → AI /api/ai/resource-insights
POST /api/admin/ai/chat                  → AI /api/ai/chat
POST /api/admin/ai/elder-detail-action   → AI /api/ai/elder-detail-action
GET  /api/admin/ai/rules                 → AI /api/ai/rules
PATCH /api/admin/ai/rules/{ruleId}/toggle → AI /api/ai/rules/{ruleId}/toggle
GET  /api/admin/ai/models/status         → AI /api/ai/models/status
GET  /api/admin/ai/audit-logs            → AI /api/ai/audit-logs
GET  /api/admin/ai/audit-logs/{auditId}  → AI /api/ai/audit-logs/{auditId}
```

### Nani BFF 新增端点

```text
POST /api/nani/ai/shift-summary          → AI /api/ai/shift-summary
POST /api/nani/ai/care-copilot           → AI /api/ai/care-copilot
POST /api/nani/ai/handover-draft         → AI /api/ai/handover-draft
POST /api/nani/ai/escalation-draft       → AI /api/ai/escalation-draft
```

### Family BFF 新增端点

```text
POST /api/family/ai/today-summary        → AI /api/ai/today-summary
POST /api/family/ai/health-explain        → AI /api/ai/health-explain
POST /api/family/ai/visit-assistant       → AI /api/ai/visit-assistant
POST /api/family/ai/visit-risk            → AI /api/ai/visit-risk
POST /api/family/ai/family-chat           → AI /api/ai/family-chat
```

---

## 八、数据库设计

### 现有数据库

所有服务共享 PostgreSQL 单实例 `nursing_platform`，按 `TenantId` 逻辑隔离。

### AI Orchestration 新增表

| 表名 | 用途 |
|------|------|
| `ai_audit_logs` | AI 推理审计日志 |
| `ai_rules` | AI 规则开关配置 |
| `ai_conversation_messages` | AI 对话消息（chat 能力用） |

### 数据库配置

```json
{
  "ConnectionStrings": {
    "Postgres": "${DATABASE_CONNECTION_STRING:Host=localhost;Port=5432;Database=nursing_platform;Username=nursing;Password=nursing}"
  }
}
```

---

## 九、安全边界

- AI 端点统一要求 JWT 认证 (`RequireAuthorization`)
- AI 请求必须携带 TenantId（平台上下文中间件自动解析）
- API Key 通过环境变量注入，不存储在代码或配置文件中
- AI 审计日志不记录完整 prompt/response 内容，仅记录 hash + 大小

---

## 十、可观测性

### OTel 指标 (Meter: NursingBackend.AiOrchestration)

| 指标 | 类型 | 说明 |
|------|------|------|
| `ai.requests.total` | Counter | AI 请求总数 (tag: capability, provider) |
| `ai.requests.cached` | Counter | 缓存命中数 |
| `ai.requests.failed` | Counter | 失败数 (tag: capability, error_type) |
| `ai.latency.ms` | Histogram | 推理延迟分布 |
| `ai.tokens.input` | Counter | 输入 token 数 |
| `ai.tokens.output` | Counter | 输出 token 数 |

### 健康检查

- `/health` — 平台标准健康检查
- AI 就绪状态纳入启动检查：至少一个 provider 可达

---

## 十一、配置总览

所有可配置项汇总：

| 配置路径 | 环境变量 | 默认值 | 说明 |
|----------|----------|--------|------|
| `AiModels:Providers:openai:ApiKey` | `AI_OPENAI_API_KEY` | — | OpenAI API Key |
| `AiModels:Providers:openai:BaseUrl` | `AI_OPENAI_BASE_URL` | `https://api.openai.com/v1` | OpenAI 端点 |
| `AiModels:Providers:openai:DefaultModel` | `AI_OPENAI_DEFAULT_MODEL` | `gpt-4o-mini` | 默认模型 |
| `AiModels:Providers:openai:TimeoutSeconds` | — | `30` | 超时 |
| `AiModels:Providers:openai:MaxRetries` | — | `2` | 重试次数 |
| `AiModels:Providers:anthropic:ApiKey` | `AI_ANTHROPIC_API_KEY` | — | Anthropic API Key |
| `AiModels:Providers:anthropic:BaseUrl` | `AI_ANTHROPIC_BASE_URL` | `https://api.anthropic.com/v1` | Anthropic 端点 |
| `AiModels:Providers:anthropic:DefaultModel` | `AI_ANTHROPIC_DEFAULT_MODEL` | `claude-sonnet-4-20250514` | 默认模型 |
| `AiModels:Providers:local:BaseUrl` | `AI_LOCAL_BASE_URL` | `http://localhost:11434/v1` | 本地模型端点 |
| `AiModels:Providers:local:DefaultModel` | `AI_LOCAL_DEFAULT_MODEL` | `qwen2.5:7b` | 本地默认模型 |
| `AiModels:Capabilities:*:Provider` | — | 见第二节 | 能力 → 提供商映射 |
| `AiModels:Capabilities:*:Model` | — | 见第二节 | 能力 → 模型映射 |
| `AiModels:Capabilities:*:Temperature` | — | — | 推理温度 |
| `AiModels:Capabilities:*:MaxTokens` | — | — | 最大输出 token |
| `Redis:ConnectionString` | `REDIS_CONNECTION_STRING` | `localhost:6379` | Redis 连接串 |
| `Redis:InstanceName` | — | `nursing:` | Key 前缀 |
| `CacheTtl:AiInferenceMinutes` | — | `15` | AI 推理缓存 TTL |
| `CacheTtl:DashboardSeconds` | — | `120` | Dashboard 缓存 TTL |
| `CacheTtl:BffReadModelSeconds` | — | `180` | BFF 读模型缓存 TTL |
| `CacheTtl:SessionMinutes` | — | `30` | 会话缓存 TTL |
| `RabbitMQ:HostName` | `RABBITMQ_HOSTNAME` | `localhost` | RabbitMQ 主机 |
| `RabbitMQ:Port` | `RABBITMQ_PORT` | `5672` | RabbitMQ 端口 |
| `RabbitMQ:UserName` | `RABBITMQ_USERNAME` | `guest` | RabbitMQ 用户 |
| `RabbitMQ:Password` | `RABBITMQ_PASSWORD` | `guest` | RabbitMQ 密码 |
| `ConnectionStrings:Postgres` | `DATABASE_CONNECTION_STRING` | `Host=localhost;...` | PostgreSQL 连接串 |
| `ServiceEndpoints:AiOrchestration` | — | `http://localhost:5267` | AI 服务地址 |

---

## 十二、实施计划

### Phase 1: AI Orchestration Service 核心 (本次交付)

1. 引入 `Microsoft.Extensions.AI` 抽象 + `Microsoft.Extensions.AI.OpenAI` 实现
2. 实现多模型路由 (`AiModelRouter`)
3. 实现 Redis 缓存层 (`AiResultCache`)
4. 实现审计日志持久化 (`AiAuditLog` EF 表)
5. 实现核心端点（优先覆盖三端最高频场景）
6. 三端 BFF 添加 AI 代理端点

### Phase 2: 异步 AI + 事件扩展

1. EventWorker 新增 AI 事件消费者
2. 健康风险自动检测链路
3. 对话消息持久化与多轮上下文

### Phase 3: RAG 与知识检索

1. 向量存储集成（pgvector 或外部向量数据库）
2. 护理知识库索引
3. Admin/Family 问答增强
