# Health Monitoring Delivery Unit

## Scope

- Entry route: `src/app/health-monitoring/page.tsx`（同时通过 `src/app/health/page.tsx` 重导出）
- Affected users: 监控值班、护理主管、异常升级协同用户
- Rollout stage: 第二十批健康监测主区切换到真实 Vital Observation 数据

## Change Summary

- 健康监测主页从 `@/lib/data/health-data` 静态数据与 `@/lib/mock/admin-ai` 健康 AI 文案，迁移到真实 `Admin BFF /api/admin/vitals` 聚合序列。
- 首屏 KPI（监测中、异常预警、平均心率、平均血氧）、异常跟进队列、近 7 日趋势图、对象卡均由最近 500 条体征观察在前端派生。
- AI 风险解释、趋势解读文案保留在右轨，但解释内容改为按真实异常对象与真实趋势拼装，不再引用 mock 数据。
- 异常判定继续沿用前端 `VITAL_RANGES` 阈值（作为前端策略常量，不属于示例数据），与录入页判定口径保持一致。

## Data Source

- Route type: client component（派生 loading / error state，`reloadToken` 手动刷新）
- Primary source: `fetchAdminVitals({ take: 500 })` in `src/lib/services/admin-vital-services.ts`
- Transport chain: `fetch('/api/admin-vitals/vitals?take=500')` → `src/app/api/admin-vitals/[...segments]/route.ts` → Admin BFF `GET /api/admin/vitals` → Health Service `GET /api/health/vitals`
- Contract: `AdminVitalObservationResponse` in BuildingBlocks `HealthContracts.cs`
- 前端派生口径：
  - 按 `elderId` 取最近一次观察作为当前状态。
  - 近 7 日趋势按 `recordedAtUtc` 的本地日期分桶，计算平均心率 / 收缩压 / 血氧 / 血糖。
  - 异常项依据 `VITAL_RANGES` 阈值拼装 `abnormalItems[]`。

## UI States

- Loading: `records === null && error === null` 派生；显示骨架或加载占位，KPI 以 `--` 占位。
- Empty: 无任何体征观察时显示"暂无体征记录"空态，引导进入 `/elderly/vitals/new` 录入。
- Abnormal empty: 切换异常视图但无异常对象时继续显示显式空态，不退化为静默留白。
- Error: Health Service 不可达显示错误卡 + 重试；右轨 AI 卡降级为提示而不是完整 KPI。
- Mobile: 对象卡 2x2 网格、KPI 两列堆叠、趋势图在窄屏下纵向堆叠。

## Health Signals

- Healthy: KPI、异常队列、趋势图、对象卡来源于同一真实观察序列；新录入体征在刷新后立即反映。
- Failure: KPI 与对象卡不一致、异常跟进队列为空但对象卡显示异常、趋势图为空但存在 7 日内数据。
- Verification proxy: `npm run lint` + `npm run build`；Health Service 与 Admin BFF `dotnet build` 通过。

## Verification

- Minimum gate: `npm run lint`
- Behavior gate: `npm run lint` + `npm run build`
- Backend gate: Health Service / Admin BFF / BuildingBlocks `dotnet build`（Unit 6 已完成，本 Unit 不新增后端改动）
- Docs gate: `npm run docs:build`
- Manual path: 验证全部/异常切换、空态、KPI 与对象卡一致性、跟进队列与真实异常对象一致、AI 运营中心跳转带正确 `elderId/elderName`。

## Rollback

- Revert `src/app/health-monitoring/page.tsx` 与本 delivery note。
- 本 Unit 不改后端契约与代理，回滚面仅限前端页面。
- 若 `/api/admin-vitals` 代理异常，页面会落到错误卡；不会污染其它模块。
