# Data Dashboard Delivery Unit

## Scope

- Entry route: src/app/data-dashboard/page.tsx
- Affected users: 运营分析、院务管理、绩效查看用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- 数据分析看板当前承接入住率趋势、服务执行率、收入结构、分院入住和员工效率排行。
- 当前交付单元先固定分析看板职责和验证门禁，不修改图表和统计口径。
- 保持数据分析看板作为 `/analytics` 兼容入口的源视图不变。

## Data Source

- Route type: client analytics dashboard page
- Primary sources: organizations, occupancyData, serviceExecution, incomeItems, staffRanking
- Downstream dependencies: shared DataCard, StatCard and CSS chart primitives

## UI States

- Loading state: 当前为本地静态分析数据；后续接真实分析 API 时需补页面级加载反馈。
- Empty state: 趋势、排行或收入拆分为空时应显式提示无统计样本，而不是展示空图表。
- Error state: 顶部 KPI、图表和排行口径不一致时应显式暴露分析失败。
- Mobile impact: KPI 栅格、双栏图表和排行表格需要验证窄屏下的滚动和折行行为。

## Health Signals

- Healthy signal: 数据分析看板在统一统计口径下稳定展示趋势、结构和排行，并能承接 `/analytics`。
- Failure signal: 看板图表与 KPI 冲突、排行失真，或兼容入口与源视图分叉。
- Verification proxy: lint 通过；行为改动时加 build 与分析看板人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证数据分析页可查看 KPI、趋势图、收入概况和员工效率排行

## Rollback

- Revert this delivery note and any future data dashboard route changes together.
- If regressions appear, fallback is the current local analytics dashboard composition.