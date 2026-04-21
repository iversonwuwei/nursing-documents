# Elderly Visits Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/page.tsx
- Affected users: 前台接待、家属沟通、探视审核用户
- Rollout stage: visits 列表从 mock `care-service-workflow` 切换到真实 Visit Service 只读通道

## Change Summary

- 页面改为从 Admin BFF `/api/admin/visits` 直读 Visit Service 预约记录，不再订阅 `care-service-workflow` mock。
- 默认展示当前租户下最近 100 条预约（按 `PlannedAtUtc` 倒序），状态取真实值（`Requested` / `Approved` / `Rejected` / `Completed` / `Cancelled`）。
- 本期审批写入仍挂起（Visit Service 尚未暴露 approve/reject API），页面为只读；主区保留审核说明和入口占位，明确标记为“审批能力接入中”。
- AI 探视建议与页面说明继续挂在右侧信息轨，保持主区聚焦真实预约列表。

## Data Source

- Route type: client component（派生 loading state，reloadToken 手动刷新）
- Primary source: `fetchAdminVisits()` in `src/lib/services/admin-visit-services.ts`
- Transport chain: 前端 `fetch('/api/admin-visits/visits')` → Next.js 代理 `src/app/api/admin-visits/[...segments]/route.ts` → Admin BFF `GET /api/admin/visits` → Visit Service `GET /api/visits/appointments`
- Contract: `AdminVisitAppointmentResponse` in BuildingBlocks `VisitContracts.cs`，字段含 `visitId`, `elderId`, `tenantId`, `visitorName`, `relation`, `phone?`, `plannedAtUtc`, `visitType`, `status`, `notes?`.

## UI States

- Loading state: 初次加载与 reload 过程中显示加载态；采用 `records === null && error === null` 派生，不在 effect 内部 setState。
- Empty state: 真实服务返回空数组时显式提示“暂无预约记录”，提供刷新按钮；不回落 mock。
- Error state: fetch 失败时在主区渲染错误卡，提示“Visit Service 暂不可用”，保留重试按钮；统计 KPI 显示 0。
- Mobile impact: 列表卡保持横向可滚动；审批说明占位在窄屏下以全宽堆叠。
- Help state: 信息轨继续承载 AI 探视建议、审批边界说明和帮助入口，主区不被压缩。

## Health Signals

- Healthy signal: 接口返回真实预约数据，KPI（总数/今日/待审核/已完成）与表格一致；切换筛选保留真实口径。
- Failure signal: Visit Service 不可达、BFF 代理 502、字段缺失导致统计为 0 但列表非空、错误态未显示。
- Verification proxy: `npm run lint` + `npm run build`；后端 `dotnet build` 绿；手工访问 `/elderly/visits` 观察真实接口命中。

## Verification

- Minimum gate: `npm run lint`
- Behavior gate: `npm run lint` + `npm run build`
- Backend gate: `dotnet build` for BuildingBlocks → Visit Service → Admin BFF
- Manual path: `/elderly/visits` 可见真实预约；断开 Visit Service 观察错误态；点击刷新恢复。

## Rollback

- 还原文件：
  - `src/app/elderly/visits/page.tsx`（回到 mock 订阅版本）
  - `src/lib/services/admin-visit-services.ts`（删除）
  - `src/app/api/admin-visits/[...segments]/route.ts`（删除）
  - Admin BFF `Program.cs` 中新增的 `/api/admin/visits` endpoint（删除）
  - Visit Service `Program.cs` 中新增的跨长者 `GET /api/visits/appointments` endpoint（删除）
  - BuildingBlocks `VisitContracts.cs` 中新增的 `AdminVisitAppointmentResponse` 契约（删除）
- 回滚后前端退回 mock `care-service-workflow` 订阅；写入路径仍由现有 `POST /api/visits/appointments` 承担。

# Elderly Visits Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/page.tsx
- Affected users: 前台接待、家属沟通、探视审核用户
- Rollout stage: 探视审核页主区收口与帮助后置批次

## User Impact

- 探视记录页不再只承接 admin 端手工新建预约，也需要承接 family 端家属预约回流后的待审核队列。
- 审核用户需要在同一页看到预约来源、是否命中 AI 风险、是否已经在家属端自动通过，以及哪些预约仍需人工放行或驳回。
- 主区优先保留待审核对象、审核动作和探视列表，AI 探视建议与页面说明后置到信息轨。
- 保持探视记录、AI 探视建议和审核动作在同一页面联动展示，但把“待审核入口”升级为真正的审核工作台而不是只有一个通过按钮。

## Data Source

- Route type: client visits page with local search state
- Primary sources: care service workflow visit store, family AI visit suggestions, family visit workflow backflow mock
- Downstream dependency: local approve or reject actions, family-origin visit context banner, selected record review state

## UI States

- Loading state: 当前搜索、筛选和审核为本地即时响应；后续接真实预约服务时需补 family 回流延迟与审核提交反馈。
- Empty state: 搜索后无匹配探视记录时显式提示无结果；若当前没有待审核预约，也要明确显示“待审核队列为空”。
- Error state: 待审核统计、family 回流来源、AI 风险摘要和列表状态不一致时需显式暴露。
- Mobile impact: 待审核摘要卡、family 来源 banner、审核动作和探视表格在窄屏下需要维持清晰顺序和可点击性。
- Help state: 通过后置信息轨查看审核边界、AI 建议和帮助入口，不再把长说明压缩主区。

## Health Signals

- Healthy signal: family 端回流的待审核预约会进入同一 visits 审核队列，统计卡、来源标签、AI 风险摘要和审核结果保持一致；审核通过或驳回后，当前记录状态和摘要区同步刷新。
- Failure signal: family 端预约未进入待审核队列、来源信息丢失、AI 风险与审核动作脱节、待审核数量与列表不一致，或审核后仍停留在旧状态。
- Verification proxy: lint 通过；行为改动时加 build；手工回归覆盖 family-origin 预约进入待审核、审核通过、驳回三条路径。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 family-origin 待审核预约进入 visits 队列、选中后显示 family 来源和 AI 风险、执行通过或驳回后状态回流；验证搜索过滤、AI 探视助手和探视记录表仍可共同表达当前探视状态

## Rollback

- Revert this delivery note and any future elderly visits route changes together.
- If regressions appear, fallback is the current local visits dashboard behavior with only手工新建 -> 待审核 -> 通过预约，不展示 family 回流上下文或驳回动作。
