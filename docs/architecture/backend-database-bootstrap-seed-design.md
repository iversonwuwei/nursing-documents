# Backend 数据库初始化与种子设计

## Scope

- scope: 设计本地 backend 的数据库初始化、跨服务测试数据入库和 API 联调验证方案。
- boundaries: 仅覆盖本地 PostgreSQL、多服务 DbContext、Operations alert 持久化与 deterministic seed；不扩展到生产发布编排。
- dependencies: DatabaseMigrator、PostgreSQL service databases、各领域服务实体与 BFF 读路径。
- rollback: 回退本文与对应实现，重建本地数据库，恢复到当前 demo 或内存模式。

## Entry Points

- migration tool: `src/Tools/NursingBackend.DatabaseMigrator`
- seed tool: 新增 `src/Tools/NursingBackend.DatabaseSeeder`
- persisted alert entry: `src/Services/NursingBackend.Services.Operations/Program.cs`
- verification entries:
  - admin: `/alerts`、`/financial`、`/notifications`
  - backend API: operations、billing、notification 以及至少一条 family 或 nani 聚合接口

## Affected Users

- backend 开发: 需要一键得到可用 schema 和测试数据。
- frontend 与联调测试: 需要页面和 BFF 能读到稳定的真实样本，不再依赖空表或前端 demo。
- 运维与平台: 需要清楚 migration、seed、验证、回滚各自的边界。

## Design Principles

- schema 创建和数据入库分离，先迁移后 seed，避免 seed 隐式建表。
- 每个持久化领域服务默认拥有独立 database，继续沿用同实例多 database 的本地模型。
- seed 必须 deterministic 且幂等，固定业务主键或自然键，重复执行只补齐缺失数据。
- API 合同尽量不变，优先修正存储实现与数据准备方式。
- 对关键联调链路，seed 数据应覆盖读模型、写模型和必要关联键，而不是只造孤立单表行。

## Persistence Design

### Service Databases

- elder: `nursing_elder`
- health: `nursing_health`
- care: `nursing_care`
- visit: `nursing_visit`
- billing: `nursing_billing`
- notification: `nursing_notification`
- operations: 新增 `nursing_operations`

### Operations Alert Persistence

- 当前问题: alerts 存在于进程内 `ConcurrentDictionary`，服务重启即丢失，也没有 migration 或数据库表。
- target design:
  - 为 Operations Service 增加 `OperationsDbContext`
  - 新增 `AlertCaseEntity`
  - alert summary、queue、action API 全部改为基于数据库查询与更新
  - 保留当前 API 响应 shape，避免 Admin BFF 和前端再改一轮契约
- failure handling:
  - action 写入失败时返回错误，不再默默只改内存对象
  - 数据不存在时继续返回 404，保持现有调用方兼容

## Seed Strategy

### Tool Shape

- seeder 使用独立 console tool，直接引用各服务 DbContext 与实体。
- 工具启动时按服务顺序写入数据：elder -> health -> care -> visit -> billing -> notification -> operations。
- 每一步按固定主键检查是否已存在，存在则跳过或更新，不存在则插入。

### Seed Dataset Matrix

| 服务 | 关键数据 | 目标用途 |
| --- | --- | --- |
| Elder | 3 到 5 个老人档案与 admission | 支撑 admin 列表、family summary、care 与 billing 关联 |
| Health | 每位老人的健康档案摘要 | 支撑 family 或 admin 健康聚合 |
| Care | 护理计划、任务、排班、审计 | 支撑 admin workflow、nani task feed |
| Visit | 预约申请与状态样本 | 支撑 family 或 admin 探视聚合 |
| Billing | 已出账、逾期、待补偿账单 | 支撑 admin financial live summary 与 invoice queue |
| Notification | queued、delivered、failed 样本 | 支撑 admin notifications 与 family or nani 消息读取 |
| Operations | pending、processing、resolved alerts | 支撑 admin alerts live summary 与 action 回写 |

### Seed Identity Rules

- 使用固定 tenantId，例如 `tenant-demo`。
- elderId、visitId、invoiceId、notificationId、alertId 使用稳定值，而不是时间戳。
- 跨服务关联统一复用固定 elderId 与 sourceEntityId，确保 BFF 聚合能够命中。

## Verification Design

- docs: `npm run docs:build`
- backend static gate: `dotnet build nursing-backend-services.slnx && dotnet test nursing-backend-services.slnx`
- backend runtime gate:
  1. 运行 DatabaseMigrator
  2. 运行 DatabaseSeeder
  3. 查询或日志确认各服务写入数量
- integration gate:
  1. admin `/alerts`、`/financial`、`/notifications` 显示 `Live API`
  2. backend 关键 API 返回非空样本
  3. 至少一条 family 或 nani 聚合接口返回真实数据

## Observability

- migrator 输出每个 DbContext 的 migration 完成日志。
- seeder 输出每个服务新增或跳过的记录数。
- 对 operations alert action、billing invoice、notification queue 这三条联调关键链路，健康信号以可观察返回结果为准，而不是仅看服务存活。

## Rollback

- 回退 seeder 工具、Operations DbContext 与 migration。
- 从本地 PostgreSQL 删除或重建新增 database，重新运行当前基础设施脚本。
- admin 页面若因 backend 真实读模型回归失败，可临时退回 demo 模式，但不保留半持久化状态。