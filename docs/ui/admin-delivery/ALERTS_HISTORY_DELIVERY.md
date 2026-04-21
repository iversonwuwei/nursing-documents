# Alerts History Delivery Unit

## Scope

- Entry route: src/app/alerts/history/page.tsx
- Affected users: 运营复盘、报警审计与质控用户
- Rollout stage: 第二批高频路由治理说明

## User Impact

- 报警历史页不再基于 StandardModulePage 的静态 `alertRecords` 演示数据，而是读取真实告警记录，默认展示已处置与处置中的历史告警，支持按状态切换回看。
- 运营复盘、质控审计可以直接看到 Operations Service 下沉的真实告警记录（类型、级别、责任人、处置时间、处置说明）。
- 当前交付不引入新的写路径，处置动作仍在 `/alerts` 主页发起。

## Data Source

- Route type: live alerts history page
- Primary source: `fetchAdminAlertHistory(filters)` in `src/lib/services/admin-module-services.ts` -> `/api/content/alerts?status=...` -> Admin BFF `/api/admin/alerts` -> Operations Service `/api/operations/alerts`
- Status filter: 默认 `resolved`，允许切换到 `processing` 和 `all`（all 不带 status 参数，拉全量历史）
- Removed source: `src/lib/data/standard-pages.tsx` 里的 `alertsHistoryPage` 静态配置和 `alertRecords` 样例，不再被该路由消费

## UI States

- Loading state: 切换过滤或首次加载时显式展示“加载中”占位，不保留上一次的列表。
- Empty state: 当前过滤条件下无历史告警时显式提示，并给出返回主告警中心的入口。
- Error state: BFF 或 Operations Service 不可达时展示明确的错误文案与重试按钮，不回退到 mock `alertRecords`。
- Mobile impact: 历史表格采用与 `/alerts` 一致的 DataCard 纵向堆叠，窄屏下可读。

## Health Signals

- Healthy signal: `/alerts/history` 默认呈现真实已处置告警，KPI 数字与 `/alerts` 主页的 summary 对账一致；切换状态过滤时列表随后端结果变化。
- Failure signal: 出现 `alertRecords` 静态文案、或与 `/alerts` 主页口径冲突。
- Verification proxy: `npm run lint` + `npm run build` + `docs:build`。

## Verification

- Minimum gate: `npm run lint`
- Required gate for this batch (behavior change): `npm run lint` + `npm run build`
- Manual path: 登录 admin -> `/alerts/history` 默认看 resolved 历史；切到 processing 应看到正在处理中的记录；停 BFF 应看到错误态与重试。

## Rollback

- Revert this delivery note 和 `src/app/alerts/history/page.tsx` 前端 commit 即可回到旧的 StandardModulePage 静态视图。
- 由于没有新增后端/BFF 端点（复用已有 `/api/admin/alerts` 和 status 过滤），不存在服务侧需同步回滚的改动。
