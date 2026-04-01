# SaaS 微服务后端总体架构

## 交付 framing

- scope: 定义 nursing-backend-services 的总体技术架构、服务分层、多租户策略和运行边界。
- affected audience: 后端研发、平台工程、DevOps、架构评审、SaaS 交付团队。
- validation: 本文需与三端输入分析、平台总体架构和实施蓝图一致，并通过 `npm run docs:build`。
- rollback: 回退本文与文档索引变更，恢复到原有平台架构文档集合。

## 目标

为 admin、family、nani 三端建立一个可持续扩展到 SaaS 的 backend 基线，满足以下四类需求：

- 多端协同
- 领域解耦
- 工作流闭环
- 多租户运营

## 推荐分层

```text
Client Layer
├── Admin Web
├── Family App
└── Nani App

Edge Layer
├── API Gateway
├── Admin BFF
├── Family BFF
└── Nani BFF

Domain Layer
├── Identity Service
├── Tenant Service
├── Elder Service
├── Care Service
├── Health Service
├── Visit Service
├── Staffing Service
├── Operations Service
├── Billing Service
├── Notification Service
└── AI Orchestration Service

Platform Layer
├── PostgreSQL
├── Redis
├── Message Bus
├── Object Storage
├── Search / Analytics Store
└── Observability Stack
```

## 核心原则

### 前端只进 BFF，不直连领域服务

- Admin 需要页面聚合、搜索、运营态组合和权限裁剪。
- Family 需要隐私裁剪和关系绑定后的摘要对象。
- Nani 需要任务化、班次化、责任人化的移动读模型。

### 领域服务拥有自己的写模型

- Elder Service 负责老人主档、入住退住、家属关系。
- Care Service 负责护理计划、任务、执行、交接班。
- Health Service 负责生命体征、异常判定输入、时序指标。
- Visit Service 负责探视预约、审批、签到与视频协同。
- Operations Service 负责设备、物资、报警、机构设施等运营对象。

### 关键流程事件化

推荐所有高价值状态变化发出领域事件，例如：

- AdmissionApproved
- CarePlanGenerated
- VitalRecorded
- VisitRequested
- VisitApproved
- AlertRaised
- AlertClosed
- BillIssued
- NotificationDispatched

## SaaS 支持策略

### 租户模型

推荐三层租户模型：

- platform tenant: 平台运营方。
- organization tenant: 单个养老机构租户。
- site or branch: 租户下的院区、楼层、运营单元。

### 数据隔离策略

按租户规模分级支持：

| 场景 | 推荐隔离方式 | 说明 |
| --- | --- | --- |
| 小中型 SaaS 租户 | shared database + tenant_id + RLS | 成本低，适合标准化托管 |
| 大型连锁客户 | database per tenant 或 schema per tenant | 便于性能隔离与合规扩展 |
| 高敏机构 | 独立部署单元 | 满足强隔离与私有化要求 |

推荐在第一阶段统一实现 tenant context、tenant-aware repository、tenant-aware cache key 和 tenant-aware audit，再按套餐扩展到 schema 或 database 级隔离。

### 配置与特性开关

Tenant Service 应提供：

- 套餐与配额
- 模块启停
- AI 能力开关
- 数据保留策略
- 通知渠道开关
- 第三方集成配置

### 可观测性

所有日志、trace、指标必须至少包含：

- tenant_id
- organization_id
- user_id
- correlation_id
- workflow_instance_id

## 运行架构建议

### 第一阶段

- 部署方式: 容器化 + Kubernetes 或轻量容器编排。
- 协议: REST 为主，服务间同步可用 HTTP/gRPC，异步采用 message bus。
- 存储: PostgreSQL + Redis。
- 消息总线: RabbitMQ 起步，若事件量和分析链路变大再升级 Kafka。

### 第二阶段

- 引入 workflow/saga 编排。
- 引入 read model projection 与独立 analytics store。
- 引入 object storage 管理图片、护理留痕附件、报表导出文件。

## 失败模式与治理

### 典型失败模式

- BFF 查询扇出超时，导致前端聚合页渲染失败。
- workflow 跨服务推进中断，导致对象停留在中间态。
- tenant context 丢失，导致越租户访问风险。
- 移动端重复提交，导致护理打卡、健康录入或报警处理重复写入。

### 治理策略

- 所有命令接口支持 idempotency key。
- 所有跨服务状态推进采用 outbox/inbox 或 saga。
- 所有租户访问在 gateway、BFF、service 三层重复校验。
- 所有重要状态变更写入 audit log 和 domain event。

## 架构结论

1. backend 采用 edge layer + domain layer + platform layer 的标准微服务分层。
2. BFF 是必须项，不是可选优化。
3. SaaS 能力必须以内建租户上下文和分级隔离策略进入第一版工程。
4. 工作流、审计和事件总线是养老业务 backend 的核心，不是附属能力。