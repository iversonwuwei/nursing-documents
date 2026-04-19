# Backend 数据库初始化与测试数据入库

## Scope

- scope: 为 nursing-backend-services 补齐本地 PostgreSQL 建表、跨服务初始化数据与测试数据入库能力，并完成 admin 或 family 或 nani 依赖链路的前后端 API 联调验证。
- affected audience: 后端研发、前端研发、联调测试、运维与本地开发人员。
- changed behavior: 本地 backend 不再依赖零散 `EnsureCreated` 或内存 seed；迁移工具负责补建缺失数据库并建表，统一 seed 工具负责把可复现的联调样本写入真实数据库。
- dependent systems: PostgreSQL 本地实例、DatabaseMigrator、DatabaseSeeder、各领域服务 DbContext、Admin BFF 与三端前端 live API 读取路径。
- verification: `npm run docs:build`；`dotnet build nursing-backend-services.slnx && dotnet test nursing-backend-services.slnx`；运行 migrator 与 seed 后，关键 API 和前端 live 页面能读取真实库数据。
- rollback: 回退本文与设计文档、移除本轮新增 seeder 或持久化改动，清空或重建本地开发数据库，恢复到当前 demo 或内存模式。

## Problem Statement

当前本地 backend 虽然已经具备多个 EF Core DbContext 和 migration 文件，但真实联调链路仍有三个明显缺口：

- 本地数据库初始化没有统一执行入口，开发环境经常出现 database 存在但表未创建或服务未迁移到最新 schema。
- elder、health、visit、billing、notification 缺少统一的初始化或测试数据入库工具，前端 live API 经常只能读到空表。
- operations 的 alerts 仍是内存 `ConcurrentDictionary` seed，admin 报警页虽然能读到 live API，但不是数据库事实，无法满足“正经数据表 + 入库联调”的要求。
- organization、rooms、staffing 虽然已切到真实服务，但本地 migrator 和 seeder 尚未覆盖，导致这些服务在联调环境里经常出现 database 不存在、schema 未迁移或页面读取为空的情况。

## Required Outcomes

1. 所有参与本地联调的核心服务都有真实数据库表，而不是仅靠内存集合或隐式建表。
2. 本地开发可通过统一命令完成 migration 与 deterministic seed，而不是手工逐服务制造数据。
3. admin alerts、financial、notifications 至少能读取真实数据库样本。
4. family 或 nani 关键聚合链路至少有一条能够读取到真实 elder、health、care、notification 或 visit 样本。
5. seed 重复执行应保持幂等，避免每次联调都生成脏重复数据。
6. admin `/organizations`、`/rooms`、`/staff` 至少能读取一组稳定真实样本，且 organization detail 能聚合 rooms 与 staff。

## In Scope

- 为 Operations Service 增加真实数据库持久化，至少覆盖 alert case 读写。
- 为 Elder、Health、Care、Visit、Billing、Notification、Operations 补齐本地测试数据入库能力。
- 为 Organization、Rooms、Staffing 补齐本地 migration 与测试数据入库能力。
- 为本地 PostgreSQL 增加缺失的 service database 初始化配置。
- 为 backend 根目录提供可执行的 migration + seed 路径，并记录验证方式。
- 完成 admin live API 页面与必要的 backend API 级验证。

## Out Of Scope

- 不在本轮引入生产级初始化平台或远程环境 seed 流程。
- 不在本轮重构 Identity、Tenant、Config、AI Orchestration 的业务数据模型。
- 不把所有 demo 页面一次性都切到真实后端；本轮只收口真实入库与可验证的联调闭环。

## Acceptance

- 运行 migrator 后，本地各服务数据库存在对应业务表与 `__EFMigrationsHistory`。
- 运行 seed 后，elder、health、care、visit、billing、notification、operations 至少各有一组可读业务样本。
- 运行 seed 后，organization、rooms、staffing 也各有一组可读业务样本，并能与 elder room number 聚合命中。
- room occupant 聚合查询 Elder Service 时，入住状态过滤必须沿用 Elder API 已存在的 `Active` 契约值，不能改用前端展示文案 `已入住`。
- 本地用 Identity Service `dev-login` 验证 admin `/organizations`、`/rooms`、`/staff` 时，token 中的 `tenantId` 必须使用 seed 固定租户 `tenant-demo`，不能误填机构 id 如 `ORG-PD-01`。
- admin `/alerts`、`/financial`、`/notifications` 打开后显示 `Live API`，且内容来自真实数据库而不是内存对象。
- admin `/organizations`、`/rooms`、`/staff` 打开后显示 `Live API`，且 organization detail 能看到真实 rooms 与 staff 聚合摘要。
- 至少一条 family 或 nani 真实聚合接口返回非空样本，证明跨服务链路已打通。
- seed 再次执行不会制造不可控重复数据；若数据已存在，则更新或跳过。

## Delivery Unit

### Phase 1.3 Database Bootstrap And Seed Closure

- entry points: `src/Tools/NursingBackend.DatabaseMigrator`、新增 seeder 工具、Operations Service alert API、admin 三个 live 页面。
- entry points: `src/Tools/NursingBackend.DatabaseMigrator`、`src/Tools/NursingBackend.DatabaseSeeder`、Organization/Rooms/Staffing 服务、Admin BFF 的 organizations/rooms/staff 页面与 API。
- rollout stage: 本地开发与联调环境优先，不改变生产发布策略。
- validation gate: `npm run docs:build`；`dotnet build nursing-backend-services.slnx && dotnet test nursing-backend-services.slnx`；运行 migrator 与 seed；关键 API 与页面联调通过。
- rollback path: 回退 seeder 与 operations 持久化改动，删除新增 migration，重置本地服务数据库，页面恢复到当前 demo 或内存读模型。

### Phase 1.3 Acceptance

- operations alerts 从数据库读取与更新，不再只存在于进程内内存集合。
- 新 seed 工具能一次写入 elders、health archives、care workflow、visit appointments、billing invoices、notification queue 与 alert cases。
- migrator 能在复用已有本地 Postgres volume 时自动补建 `nursing_organizations`、`nursing_rooms`、`nursing_staffing` 等缺失 database，而不要求开发者先清空 volume 重建。
- seed 工具能一次写入 organizations、rooms、staffing 样本，并让 room occupant 聚合直接命中 elder 已入住房号。
- BFF 在读取 elder 台账做 room occupant 聚合时必须使用 Elder 列表接口已定义的 `status=Active` 过滤值，确保房间与机构占床统计能命中真实在住对象。
- 关键 seed 数据带固定 tenantId、elderId 与关联键，能支撑 BFF 聚合和前端页面复现。
- 运行日志能明确输出各服务迁移完成与 seed 写入数量，便于判断本地环境是否健康。
