# Elderly Vitals Delivery Unit

## Scope

- Entry route: src/app/elderly/vitals/page.tsx
- Affected users: 当班护士、护理主管、前台查询
- Rollout stage: vitals 列表从 mock `care-service-workflow` 切换到真实 Health Service 观察值序列

## Change Summary

- 页面改为从 Admin BFF `/api/admin/vitals` 直读 Health Service 生命体征观察序列，不再订阅 `care-service-workflow` mock。
- 默认返回当前租户最近 100 条体征记录（按 `recordedAtUtc` 倒序），覆盖血压、心率、体温、血氧、血糖等字段。
- KPI 以“最新一次真实记录”替代硬编码 demo 值；无记录时展示占位并提示先在 `/elderly/vitals/new` 录入。
- 帮助信息、趋势判读边界继续挂右侧信息轨，主区聚焦真实观察值表格。

## Data Source

- Route type: client component（派生 loading / error state，reloadToken 手动刷新）
- Primary source: `fetchAdminVitals()` in `src/lib/services/admin-vitals-services.ts`
- Transport chain: 前端 `fetch('/api/admin-vitals/vitals')` → Next 代理 `src/app/api/admin-vitals/[...segments]/route.ts` → Admin BFF `GET /api/admin/vitals` → Health Service `GET /api/health/vitals`
- Contract: `AdminVitalObservationResponse` in BuildingBlocks `HealthContracts.cs`

## UI States

- Loading state: `records === null && error === null` 派生；刷新按钮重置为加载态。
- Empty state: 服务返回空数组时提示“暂无体征记录”，引导进入新增页面。
- Error state: Health Service 不可达显示错误卡，保留重试；KPI 回退到占位 `--`。
- Mobile impact: 表格横向可滚动；KPI 以两列堆叠。

## Health Signals

- Healthy: 接口返回真实体征；KPI 显示“最新一次” 值；新增记录刷新后立刻可见。
- Failure: 错误态、KPI 不一致、筛选命中但表格为空。
- Verification proxy: `npm run lint` + `npm run build`；Health Service 与 Admin BFF `dotnet build`；EF 迁移 `dotnet ef database update` 成功。

## Verification

- Minimum gate: `npm run lint`
- Behavior gate: `npm run lint` + `npm run build`
- Backend gate: `dotnet build` + `dotnet ef migrations add` + `dotnet ef database update` 成功
- Manual path: `/elderly/vitals` 展示真实记录；先新增一条后刷新可见；断开 Health Service 触发错误态。

## Rollback

- 还原文件：
  - `src/app/elderly/vitals/page.tsx`
  - `src/lib/services/admin-vitals-services.ts`（删除）
  - `src/app/api/admin-vitals/[...segments]/route.ts`（删除）
  - Admin BFF `/api/admin/vitals` endpoints（删除）
  - Health Service `/api/health/vitals` endpoints（删除）
  - HealthContracts.cs 中新增 `VitalObservationCreateRequest` / `VitalObservationResponse` / `AdminVitalObservationResponse`（删除）
  - HealthEntities.cs 中 `VitalObservationEntity`（删除）
  - HealthDbContext VitalObservations DbSet（删除）
  - EF migration `AddVitalObservations` 回退 `dotnet ef database update InitialCreate`
