# Health Metric Delivery Unit

## Scope

- Entry route: src/app/health/[metric]/page.tsx
- Affected users: 健康专题查看、护理分析、监控值班用户
- Rollout stage: 第二十批健康专题页切换到真实 Vital Observation 聚合数据

## Change Summary

- `/health/[metric]` 不再渲染 `standard-pages.tsx` 中的静态专题配置，而是切到真实 `fetchHealthMonitoringData()` 结果。
- 支持 `bp`、`hr`、`sleep` 三类专题：
- `bp` 基于真实最近一次血压与近 7 日高压趋势排序重点对象。
- `hr` 基于真实最近一次心率与近 7 日平均心率趋势排序重点对象。
- `sleep` 当前无独立睡眠服务，显式使用夜间体征样本作为代理视角，并在页面中声明边界。
- 非法 metric 仍然通过 `notFound()` 显式失败，不做静默回退。

## Data Source

- Route type: server wrapper + client metric page
- Primary source: `fetchHealthMonitoringData()` in `src/lib/services/admin-health-services.ts`
- Transport chain: `fetch('/api/admin-vitals/vitals?take=500')` → `src/app/api/admin-vitals/[...segments]/route.ts` → Admin BFF `GET /api/admin/vitals` → Health Service `GET /api/health/vitals`
- Derived behavior:
- `bp`/`hr` 直接从最近一次 observation 与 7 日聚合点派生。
- `sleep` 从 22:00-06:00 observation 子集派生夜间代理指标，不宣称独立睡眠医疗数据。

## UI States

- Loading state: `data === null && error === null` 时显示专题级占位。
- Empty state: 实时 observation 为空时显示专题空态，而不是静态样例表。
- Error state: Health Service 不可达显示错误卡；非法 metric 继续 `notFound()`。
- Mobile impact: KPI、趋势图、重点对象表在窄屏下堆叠；表格保持横向滚动。

## Health Signals

- Healthy signal: 三个合法 metric 页面都来自同一真实 observation 数据源，重点对象排序与总览口径一致。
- Failure signal: metric 页面仍显示静态样例、排序与总览不一致、sleep 页面未声明代理边界。
- Verification proxy: docs build、lint、build 通过；手测 `bp`、`hr`、`sleep` 与非法 metric。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Docs gate: npm run docs:build
- Manual path: 验证 `/health/bp`、`/health/hr`、`/health/sleep` 可正常渲染实时专题，非法 metric 返回 notFound

## Rollback

- Revert `src/app/health/[metric]/page.tsx`、`src/app/health/[metric]/health-metric-client.tsx` 与本 delivery note。
- 若真实 observation 链路不可用，页面会走错误态；回滚后可恢复原静态专题配置。
