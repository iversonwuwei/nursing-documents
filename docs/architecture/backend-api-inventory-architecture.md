# 基于现有 API 清单的 Backend 架构设计

## 交付 framing

- scope: 基于已审计的真实 API 清单，为养老护理项目设计 phase 1 backend 架构，覆盖内容管理、护理工作流、AI 运营中心，以及 staff/family AI 预览所依赖的后端边界。
- affected audience: 平台架构师、后端研发、BFF 研发、Admin/Family/Nani 前端研发、测试、运维。
- validation: 文档需与现有 backend 代码边界保持一致，并通过 `npm run docs:build`。
- rollback: 回退本文、OpenAPI 文件、API 说明页和索引修改。

---

## 一、背景与范围

本设计不是从零假设一个新后端，而是以当前 workspace 中已经出现的真实 API 面为约束来做实施级架构设计。根据此前对 admin 前端的 API 审计，当前真正连到后端的链路主要有三类：

| API 面 | 当前真实路径 | 当前调用方 | 后端目标 |
| --- | --- | --- | --- |
| 内容管理 | `/api/admin/static-texts`、`/api/admin/option-groups`、`/api/admin/audit-logs` | admin 设置中心 | 承载静态文案、静态选项和配置审计 |
| 护理工作流 | `/api/admin/nursing/*` | admin 护理套餐、计划、任务、排班页 | 承载套餐、计划、任务执行和可观测性 |
| AI 能力 | `/api/admin/ai/*`、`/api/nani/ai/*`、`/api/family/ai/*` | admin AI 中心、staff/family 预览页 | 承载 AI 推理、治理、审计、跨端摘要与解释 |

因此，phase 1 backend 架构的设计目标不是“一次覆盖全部业务模块”，而是先为这三条已落地 API 面提供可扩展、可治理、可观测的后端基础。

### 本文纳入范围

1. 技术栈确定
2. backend 目录结构
3. 模块与服务划分
4. 核心数据库表结构
5. phase 1 Swagger/OpenAPI 契约输出

### 本文明确不纳入范围

1. 仍处于 mock 的页面级读接口
2. 尚未存在后端能力的 clock-in review 专用端点
3. 搜索、报表、推荐等后续分析型只读投影服务的最终拆分

---

## 二、技术栈结论

## 2.1 主栈选择

选择 **.NET 10 + ASP.NET Core Minimal API** 作为主后端栈，继续沿用现有 `nursing-backend-services` 的技术基线。

| 维度 | 选型 | 原因 |
| --- | --- | --- |
| 网关 / BFF / 领域服务 | .NET 10 + ASP.NET Core Minimal API | 当前仓库已采用；与现有 BFF、服务、Contracts、Platform Defaults 完全一致；适合 typed contracts 与高密度 BFF 聚合路由 |
| 数据访问 | EF Core + Npgsql | 现有服务已接入；适合多服务独立 schema 和事务边界 |
| 主关系库 | PostgreSQL 16 | 适合事务型业务、JSONB 扩展、多租户索引策略 |
| 时序指标 | TimescaleDB 扩展 | 健康监测和设备监测天然适合时序分区与压缩 |
| 缓存 | Redis | 已在 AI Orchestration 中具备缓存设计；适合 AI 结果缓存与配置快照 |
| 消息骨干 | RabbitMQ + Outbox | 现有 BuildingBlocks 已具备拓扑、重试和死信能力 |
| 鉴权 | JWT Bearer + Identity Service | 与当前 dev-login / tenant claims / BFF 透传模式一致 |
| 可观测性 | OpenTelemetry + Prometheus + 结构化日志 | 现有平台默认能力，可覆盖 trace、指标、审计与重试链路 |
| API 契约 | OpenAPI 3.1 YAML | 便于先沉淀审阅面，再逐步转向代码自动导出 |

## 2.2 为什么不选 Node.js 或 Python 作为主栈

- Node.js 不作为主栈：当前 backend solution、共享合同、平台默认件、认证和 OTEL 基础都在 .NET 中，切换到 Node.js 会引入双运行时运维成本和额外契约漂移风险。
- Python 不作为主栈：Python 适合模型离线处理或数据科学任务，但不适合作为当前高治理、多租户、强审计 BFF/领域服务主栈。
- 结论：**.NET 负责在线业务后端；Python 只保留为未来可选的离线模型评估、Embedding 构建或数据处理 worker。**

---

## 三、目录结构设计

建议在现有 solution 结构上保持“入口层 + 领域层 + 支撑层 + 运行层”四层组织，不引入新的平行目录体系。

```text
nursing-backend-services/
├── src/
│   ├── Gateway/
│   │   └── NursingBackend.ApiGateway/
│   ├── Bff/
│   │   ├── NursingBackend.Bff.Admin/
│   │   ├── NursingBackend.Bff.Family/
│   │   └── NursingBackend.Bff.Nani/
│   ├── Services/
│   │   ├── NursingBackend.Services.Identity/
│   │   ├── NursingBackend.Services.Tenant/
│   │   ├── NursingBackend.Services.Config/
│   │   ├── NursingBackend.Services.Elder/
│   │   ├── NursingBackend.Services.Health/
│   │   ├── NursingBackend.Services.Care/
│   │   ├── NursingBackend.Services.Visit/
│   │   ├── NursingBackend.Services.Staffing/
│   │   ├── NursingBackend.Services.Operations/
│   │   ├── NursingBackend.Services.Billing/
│   │   ├── NursingBackend.Services.Notification/
│   │   └── NursingBackend.Services.AiOrchestration/
│   ├── BuildingBlocks/
│   │   └── NursingBackend.BuildingBlocks/
│   ├── Workers/
│   │   └── NursingBackend.EventWorker/
│   └── Tools/
│       ├── NursingBackend.DatabaseMigrator/
│       └── NursingBackend.DeadLetterReplay/
├── tests/
│   └── NursingBackend.ArchitectureTests/
└── deploy/
    ├── compose.infrastructure.yml
    └── k8s/
```

### 服务内推荐目录

每个领域服务内部建议统一采用如下结构，避免 Program.cs 持续膨胀成“全部逻辑入口文件”：

```text
NursingBackend.Services.Xxx/
├── Api/
│   ├── Endpoints/
│   ├── Requests/
│   └── Responses/
├── Application/
│   ├── Commands/
│   ├── Queries/
│   ├── Handlers/
│   └── Policies/
├── Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Events/
│   └── Services/
├── Infrastructure/
│   ├── Persistence/
│   ├── Messaging/
│   ├── ReadModels/
│   └── ExternalClients/
└── Program.cs
```

### 目录层面的关键决策

1. `Config Service` 独立存在，不挂到 Tenant Service 内部子目录。
原因：内容管理链路已有独立 API 面、审计需求和 App 配置快照需求，后续很容易扩展为跨端内容治理服务。

2. `AiOrchestration Service` 保持独立，不并入任一 BFF。
原因：AI 是跨 admin、nani、family 三端共享能力，且治理、规则、审计、缓存有统一诉求。

3. `Workers` 继续独立于业务服务。
原因：AI 预热、事件消费、通知补偿、缓存失效都更适合作为后台消费进程而不是 HTTP 服务进程的一部分。

---

## 四、模块 / 服务划分

## 4.1 入口层划分

| 层 | 职责 | 不负责 |
| --- | --- | --- |
| API Gateway | 统一入口、认证透传、租户解析、基础限流、路由到 BFF/服务 | 页面 DTO 组装、复杂业务决策 |
| Admin BFF | admin 页面级聚合、内容管理代理、护理工作流聚合、AI 运营中心代理 | 核心写模型持久化 |
| Family BFF | 家属视图裁剪、家属端 AI 解释与摘要代理 | 内部运营字段暴露 |
| Nani BFF | 护工端班次、任务、交接班与 AI 摘要代理 | 业务事实源 |

## 4.2 领域服务划分

| 服务 | 归属 API 面 | 核心职责 | 主要表 / 状态 |
| --- | --- | --- | --- |
| Identity Service | 全局 | 用户、角色、token、设备会话 | `app_user`、`user_role`、`device_session` |
| Tenant Service | 全局 | 租户、套餐、特性开关、院区配置 | `tenant`、`tenant_feature_flag`、`tenant_branch` |
| Config Service | 内容管理 | 静态文本、选项组、配置快照、内容审计 | `static_text`、`option_group`、`option_item`、`content_audit_log`、`app_config_snapshot` |
| Elder Service | admin / family | 老人主档、入住、家属绑定 | `elder`、`admission_case`、`elder_family_binding` |
| Health Service | admin / family / nani | 生命体征、趋势、异常识别输入 | `vital_record`、`health_alert`、`health_trend_bucket` |
| Care Service | 护理工作流 | 套餐、计划、任务、交接班、任务审计 | `service_package`、`service_plan`、`service_plan_task`、`care_handover` |
| Staffing Service | admin / nani | 员工、班次、排班、责任归属 | `staff_member`、`shift_assignment`、`staff_schedule` |
| Visit Service | family / admin | 探视预约、审批、签到、视频协同元数据 | `visit_appointment`、`visit_checkin` |
| Operations Service | admin / nani | 房间、床位、设备、物资、报警、事件 | `room`、`bed`、`equipment`、`supply_stock`、`operations_alert`、`incident` |
| Billing Service | admin / family | 账单、支付状态、欠费观察 | `invoice`、`invoice_line`、`payment_record` |
| Notification Service | 全局 | 站内信、短信、push、发送结果 | `notification_message`、`notification_delivery_attempt` |
| AI Orchestration Service | AI | 推理路由、规则治理、审计、会话、缓存 | `ai_rules`、`ai_audit_logs`、`ai_conversation_messages` + Redis |

## 4.3 与当前 API 清单的映射

### 内容管理 API

- `/api/admin/static-texts*` → Admin BFF → Config Service
- `/api/admin/option-groups*` → Admin BFF → Config Service
- `/api/admin/audit-logs*` → Admin BFF → Config Service

### 护理工作流 API

- `/api/admin/nursing/workflow-board` → Admin BFF 聚合 Care + Staffing + Elder
- `/api/admin/nursing/observability` → Admin BFF → Care Service 可观测读模型
- `/api/admin/nursing/audits` → Admin BFF → Care Service 审计查询
- `/api/admin/nursing/packages*`、`/plans*`、`/tasks*` → Admin BFF → Care Service

### AI API

- `/api/admin/ai/*` → Admin BFF → AiOrchestration Service
- `/api/nani/ai/*` → Nani BFF → AiOrchestration Service
- `/api/family/ai/*` → Family BFF → AiOrchestration Service

---

## 五、关键数据流设计

## 5.1 内容管理链路

```text
Admin Page
  -> Admin BFF
  -> Config Service
  -> PostgreSQL (static_text / option_group / option_item)
  -> content_audit_log
  -> Redis app_config_snapshot invalidation
```

健康信号：

1. 配置读写成功率
2. 审计日志写入成功率
3. 快照生成延迟

## 5.2 护理工作流链路

```text
Admin Page
  -> Admin BFF
  -> Care Service
  -> PostgreSQL (service_package / service_plan / service_plan_task)
  -> Outbox
  -> RabbitMQ
  -> Notification / AI / Observability consumers
```

健康信号：

1. `PendingReviewPlans`
2. `UnassignedPlans`
3. `TaskCompletionTotal`
4. 审计记录可查询率

## 5.3 AI 运营与预览链路

```text
Admin / Nani / Family BFF
  -> AiOrchestration Service
  -> AiModelRouter
  -> Provider Adapter (OpenAI-compatible / local)
  -> Redis result cache
  -> PostgreSQL (ai_audit_logs / ai_rules / ai_conversation_messages)
```

健康信号：

1. AI 可用率与 provider reachability
2. 单 capability latency
3. cache hit ratio
4. audit log success ratio
5. rule toggle 生效率

---

## 六、数据库表结构设计

下表为 phase 1 推荐的核心表，不替代更细粒度 DDL，但足以支撑当前已审计 API 面。

## 6.1 租户与身份

| 表名 | 主键 | 关键字段 | 说明 |
| --- | --- | --- | --- |
| `tenant` | `tenant_id` | `tenant_name`, `status`, `plan_code` | 多租户根实体 |
| `tenant_feature_flag` | `flag_id` | `tenant_id`, `flag_code`, `is_enabled` | 分租户能力开关 |
| `app_user` | `user_id` | `tenant_id`, `username`, `display_name`, `status` | 用户主档 |
| `user_role` | `user_role_id` | `user_id`, `role_code` | RBAC 关联 |
| `device_session` | `session_id` | `user_id`, `device_id`, `refresh_token_hash`, `expires_at_utc` | 移动端设备会话 |

## 6.2 内容管理

| 表名 | 主键 | 关键字段 | 说明 |
| --- | --- | --- | --- |
| `static_text` | `id` | `tenant_id`, `namespace`, `text_key`, `locale`, `text_value`, `version` | 多语言静态文案 |
| `option_group` | `id` | `tenant_id`, `group_code`, `group_name`, `status`, `is_system` | 选项分组 |
| `option_item` | `id` | `group_id`, `option_code`, `label_zh`, `label_en`, `sort_order`, `is_active` | 选项项 |
| `content_audit_log` | `id` | `tenant_id`, `resource_type`, `resource_id`, `action`, `operator_id`, `before_snapshot`, `after_snapshot` | 配置和内容治理审计 |
| `app_config_snapshot` | `id` | `tenant_id`, `namespace`, `locale`, `snapshot_version`, `content` | 客户端批量配置快照 |

## 6.3 护理工作流

| 表名 | 主键 | 关键字段 | 说明 |
| --- | --- | --- | --- |
| `service_package` | `package_id` | `tenant_id`, `name`, `care_level`, `target_group`, `monthly_price`, `status` | 护理套餐 |
| `service_plan` | `plan_id` | `tenant_id`, `package_id`, `elder_id`, `focus`, `shift`, `owner_role`, `owner_name`, `status` | 护理计划 |
| `service_plan_task` | `task_id` | `plan_id`, `title`, `scheduled_time`, `priority`, `status`, `handled_by`, `handled_at_utc` | 护理任务 |
| `service_plan_task_note` | `note_id` | `task_id`, `status`, `action_note`, `handled_by`, `handled_at_utc` | 任务备注与执行说明 |
| `care_handover` | `handover_id` | `tenant_id`, `from_shift`, `to_shift`, `draft`, `submitted_by` | 交接班草稿与确认 |
| `care_workflow_audit` | `audit_id` | `aggregate_type`, `aggregate_id`, `action_type`, `operator_user_id`, `correlation_id`, `detail_json` | 工作流审计 |

## 6.4 业务主数据与协同支撑

| 表名 | 主键 | 关键字段 | 说明 |
| --- | --- | --- | --- |
| `elder` | `elder_id` | `tenant_id`, `name`, `gender`, `birthday`, `care_level`, `status` | 老人主档 |
| `admission_case` | `admission_id` | `elder_id`, `requested_care_level`, `recommended_care_level`, `status` | 入住认定与接收入院 |
| `elder_family_binding` | `binding_id` | `elder_id`, `family_user_id`, `relationship`, `scope` | 家属绑定 |
| `vital_record` | `record_id` | `elder_id`, `metric_type`, `metric_value`, `recorded_at_utc`, `source` | 原始健康指标 |
| `health_alert` | `alert_id` | `elder_id`, `risk_level`, `alert_type`, `status`, `triggered_at_utc` | 健康异常 |
| `staff_member` | `staff_id` | `tenant_id`, `name`, `role_code`, `employment_source`, `status` | 员工主档 |
| `shift_assignment` | `assignment_id` | `staff_id`, `shift`, `date`, `status` | 班次归属 |
| `visit_appointment` | `visit_id` | `elder_id`, `family_user_id`, `time_slot`, `status`, `approved_by` | 探视预约 |
| `operations_alert` | `alert_id` | `tenant_id`, `elder_id`, `alert_type`, `severity`, `status`, `assigned_to` | 运营报警 |
| `incident` | `incident_id` | `tenant_id`, `incident_type`, `severity`, `status`, `closed_at_utc` | 事件闭环 |
| `invoice` | `invoice_id` | `tenant_id`, `elder_id`, `billing_period`, `total_amount`, `status` | 账单 |
| `notification_message` | `message_id` | `tenant_id`, `audience`, `audience_key`, `channel`, `status`, `payload_json` | 通知消息 |

## 6.5 AI 治理与审计

当前仓库中 `AiDbContext` 已落地以下表，建议继续作为 AI 服务自有数据库：

| 表名 | 主键 | 关键字段 | 说明 |
| --- | --- | --- | --- |
| `ai_audit_logs` | `audit_id` | `tenant_id`, `user_id`, `capability`, `provider`, `model`, `endpoint`, `input_hash`, `success`, `latency_ms` | AI 审计与调用结果 |
| `ai_rules` | `rule_id` | `tenant_id`, `rule_code`, `rule_name`, `capability`, `is_enabled`, `priority` | AI 规则开关与优先级 |
| `ai_conversation_messages` | `message_id` | `tenant_id`, `conversation_id`, `user_id`, `role`, `content`, `created_at_utc` | 多轮对话上下文 |

同时使用 Redis 作为非持久层：

- `ai:{tenantId}:{capability}:{inputHash}`: AI 结果缓存
- `config:{tenantId}:{namespace}:{locale}`: 配置快照缓存
- `session:{tenantId}:{userId}:{conversationId}`: 会话临时上下文

---

## 七、接口定义与 OpenAPI 输出策略

本次交付输出一份 **phase 1 统一 BFF OpenAPI 文件**，覆盖已审计并真实接入后端的 API 面：

- 内容管理
- 护理工作流
- Admin AI
- Nani AI
- Family AI

OpenAPI 文件位置：

- `/openapi/backend-bff-phase1.yaml`

站点内说明页：

- `/api/backend-bff-openapi`

### 设计约束

1. 先描述 BFF 暴露给前端的契约，不直接暴露内部服务地址。
2. AI 接口统一以 `AiResult<T>` 信封返回，保留 `available`、`provider`、`model`、`cached`、`latencyMs`、`traceId`、`auditId`。
3. 非 AI 写接口优先使用 typed request/response contract，避免匿名对象成为长期事实标准。
4. phase 1 不引入路径级 `v1` 前缀，采用向后兼容字段新增策略；若后续出现 breaking change，再进入 versioned contract 管理。

---

## 八、安全、可观测性与失败处理

## 8.1 安全与权限

- 所有 BFF 路径默认 `RequireAuthorization()`。
- Admin 侧内容管理、AI 治理接口必须绑定运营后台角色。
- Family BFF 只能返回绑定关系内的老人数据。
- Nani BFF 写入操作必须附带责任人、班次、幂等上下文。

## 8.2 失败处理

- 下游服务失败时，BFF 返回 `502 Bad Gateway`，并保留问题详情。
- 并发冲突使用 `409 Conflict`。
- 参数或状态非法使用 `400/422`。
- AI provider 不可用但业务可回退时返回 `available=false`；无法回退时返回 `503`。

## 8.3 可观测性

- 每条跨服务调用携带 `traceId` / `correlationId`。
- AI 返回值必须保留 `auditId`，便于从页面跳转到日志审计。
- 护理工作流必须保留 audit trail，支持按聚合对象和操作人检索。
- 内容管理必须同时记录 before / after snapshot。

---

## 九、实施顺序建议

1. 先稳固现有三条真实 API 面：Config、Care workflow、AI。
2. 再把 mock 仍然较重的 elders / billing / notifications 等读接口逐步接入对应领域服务。
3. 当跨服务只读查询明显增多后，再补 `Analytics Projection Service` 和专门的 `Audit Service`。

---

## 十、验证与回滚

### 验证

文档交付门禁：

1. `npm run docs:build`

代码落地时建议门禁：

1. `dotnet build nursing-backend-services.slnx`
2. `dotnet test nursing-backend-services.slnx`
3. admin 前端联调路径的 `npm run lint` 与 `npm run build`

### 回滚

1. 回退本文、OpenAPI 文件和索引入口。
2. 若后续代码按本文实施并需临时回滚 AI 预览行为，前端仍可通过 `NEXT_PUBLIC_ADMIN_AI_MODE=demo` 立即切回 demo。
3. 若内容管理或护理工作流新接口出现回归，优先回滚 BFF 路径映射，不直接破坏已有领域表结构。

---

## 设计结论

phase 1 的最佳 backend 方案不是“一个大而全单体”，也不是“立刻拆到极细微服务”，而是：

- 用 .NET 10 维持统一工程栈
- 用 Gateway + 三个 BFF 承接前端差异
- 用 Config / Care / AI 三个当前最活跃的服务面先做深做稳
- 用 PostgreSQL + Redis + RabbitMQ + OpenTelemetry 构成运行底座
- 用统一 OpenAPI 先锁定真实契约，再继续扩展其余业务域

这套架构既对齐当前代码现实，也保留了后续向 elders、billing、notifications、analytics 等更大 API 面扩展的空间。