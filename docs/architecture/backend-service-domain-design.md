# 后端服务设计与边界说明

## 交付 framing

- scope: 定义 nursing-backend-services 中网关、BFF 与领域服务的职责边界、主数据归属、主要事件与调用关系。
- affected audience: 后端研发、BFF 研发、测试、数据建模与平台团队。
- validation: 服务划分需与三端输入分析和总体架构文档一致，并通过 `npm run docs:build`。
- rollback: 回退本文和相关索引，恢复到未细分服务边界的状态。

## 设计原则

- 以领域边界而不是页面目录拆服务。
- 以写模型归属定义服务所有权，以读模型聚合交给 BFF。
- 先保证服务边界清晰，再决定是否进一步细分子服务。
- 每个拥有持久化写模型的领域服务默认使用独立数据库；服务之间通过 API、事件或投影交互，而不是共用业务表。

## Edge 服务

### API Gateway

- 职责: 统一入口、认证透传、租户解析、限流、基础审计、向 BFF/服务路由。
- 不负责: 页面 DTO 拼装、复杂业务逻辑、工作流决策。

### Admin BFF

- 职责: dashboard、analytics、运营聚合、跨模块审批页、AI 运营中心读模型。
- 主要依赖: Elder、Care、Health、Operations、Billing、Notification、AI。

### Family BFF

- 职责: 家属首页摘要、健康趋势摘要、护理记录摘要、探视中心、账单中心、消息中心、家属 AI 解释层。
- 主要依赖: Elder、Health、Care、Visit、Billing、Notification、AI。

### Nani BFF

- 职责: 班次首页、任务队列、报警响应、健康录入上下文、交接班、移动消息中心。
- 主要依赖: Identity、Care、Health、Staffing、Visit、Operations、Notification、AI。

## 领域服务地图

| 服务 | 拥有的数据 | 核心命令 | 主要事件 | 主要消费者 |
| --- | --- | --- | --- | --- |
| Identity Service | 用户、角色、凭证、设备会话 | 登录、登出、刷新、授权 | UserSignedIn, RoleChanged | Gateway, BFF, 所有服务 |
| Tenant Service | 租户、套餐、特性开关、院区 | 开通租户、配置套餐、启停模块 | TenantProvisioned, FeatureFlagChanged | Gateway, BFF, 所有服务 |
| Elder Service | 老人档案、家属关系、入住退住 | 创建入住、更新档案、绑定家属 | ElderAdmitted, ElderUpdated, FamilyBound | Admin BFF, Family BFF, Care, Visit, Billing |
| Care Service | 护理计划、护理任务、执行记录、交接班 | 生成计划、下发任务、完成任务、交接班 | CarePlanGenerated, CareTaskAssigned, CareTaskCompleted, HandoverSubmitted | Admin BFF, Nani BFF, AI, Notification |
| Health Service | 生命体征、时序指标、健康草稿、异常输入 | 录入指标、确认建档、标记异常 | VitalRecorded, HealthArchiveCreated, HealthRiskDetected | Admin BFF, Family BFF, Nani BFF, AI |
| Visit Service | 探视预约、审批、签到、视频协同元数据 | 提交预约、审批预约、签到探视 | VisitRequested, VisitApproved, VisitCheckedIn | Family BFF, Admin BFF, Notification |
| Staffing Service | 员工档案、班次、排班、责任归属 | 创建员工、确认入职、排班、换班 | StaffOnboarded, ShiftAssigned, ShiftChanged | Admin BFF, Nani BFF, Care |
| Operations Service | 机构、房间、设备、物资、报警、事件 | 设备验收、物资入库、报警分派、房间维护 | EquipmentAccepted, SupplyStocked, AlertRaised, AlertDispatched, IncidentClosed | Admin BFF, Nani BFF, Notification, AI |
| Billing Service | 套餐、账单、支付状态、欠费提醒 | 出账、调整账单、确认支付 | BillIssued, PaymentRecorded, BillOverdue | Admin BFF, Family BFF, Notification |
| Notification Service | 站内信、短信、push、模板、发送记录 | 发送通知、标记已读、模板发布 | NotificationQueued, NotificationSent, NotificationRead | Admin BFF, Family BFF, Nani BFF |
| AI Orchestration Service | Prompt 组装、模型调用记录、建议快照、审计关联 | 触发 AI 评估、生成摘要、请求解释 | AiAssessmentCompleted, AiSummaryGenerated | Admin BFF, Family BFF, Nani BFF |

## 重点边界说明

### Elder 与 Health

- Elder 管主档和关系。
- Health 管指标和健康事实。
- 家属首页展示的数据可同时依赖两者，但写模型不能混在一个服务内。

### Care 与 Staffing

- Staffing 决定“谁在什么班”。
- Care 决定“什么任务给谁做”。
- 任务归属由 Care 生成，但必须引用 Staffing 的有效班次和责任人。

### Operations 与 Visit

- Visit 单独成服务，因为它连接 family 与 admin，并包含审批、签到、视频协同。
- 设备、物资、机构、房间和报警在 phase 1 可归入 Operations，待规模扩大再拆分 Facility / Device / Supply / Alert 子服务。

### AI Orchestration

- AI Orchestration 只产出建议、摘要、评分和解释。
- 最终业务状态仍由业务服务确认和写入。
- AI 输出必须带 trace 和审计上下文，不能成为最终事实源。

## 服务拆分优先级

### phase 1 必做

- API Gateway
- Admin BFF
- Family BFF
- Nani BFF
- Identity Service
- Tenant Service
- Elder Service
- Care Service
- Health Service
- Visit Service
- Staffing Service
- Operations Service
- Billing Service
- Notification Service
- AI Orchestration Service

### phase 2 可继续拆分

- Facility Service
- Device Service
- Supply Service
- Alert Service
- Analytics Projection Service
- Audit Service

## 设计结论

当前服务粒度既能支持三端协同，也能为后续 SaaS 多租户、IoT 接入、AI 扩展和报表读模型提供演进空间；它避免了按页面拆服务，也避免了过早把所有运营对象压成单体后端。

## 数据库边界补充

- Identity 与 Tenant 当前可保持轻量服务形态；Tenant 是否持久化可按后续需求决定。
- Elder、Health、Care、Visit、Billing、Notification、Config、AI Orchestration 一旦持久化，默认各自拥有独立 PostgreSQL 数据库。
- 本地开发可以继续共用同一个 PostgreSQL 实例，但必须为不同服务创建不同 database，而不是把全部表放进同一个 `nursing_platform`。
- migration、seed、event worker 和 design-time DbContext factory 都必须按服务数据库分别配置，避免一个服务的建表流程依赖另一个服务的数据库状态。
