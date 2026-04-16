# Data Dashboard Delivery Unit

## Scope

- Entry route: src/app/data-dashboard/page.tsx
- Affected users: 运营分析、院务管理、绩效查看用户
- Rollout stage: 第十七批分析与详情页主区收口

## User Impact

- 数据分析看板当前承接聚合 KPI、模块分布、结构图表和员工效率排行。
- 主区应只保留 KPI、图表和排行表，说明型文案、入口解释与页面定位迁移到后置上下文和帮助页。
- 保持数据分析看板作为 `/analytics` 兼容入口的源视图不变。

## Data Source

- Route type: client analytics dashboard page
- Primary sources: organizations, occupancyData, serviceExecution, incomeItems, staffRanking
- Downstream dependencies: shared DataCard, StatCard and CSS chart primitives

## UI States

- Loading state: 顶部状态卡需显式说明 Syncing Snapshot / Live Snapshot，不让空图表代替加载反馈。
- Empty state: 趋势、排行或收入拆分为空时应显式提示无统计样本，而不是展示空图表。
- Error state: 顶部 KPI、图表和排行口径不一致时应显式暴露分析失败，并保留帮助入口。
- Mobile impact: KPI 栅格、图表区、排行表格和帮助卡需要验证窄屏下的纵向堆叠与横向滚动行为。

## Health Signals

- Healthy signal: 数据分析看板在统一统计口径下稳定展示趋势、结构和排行，且说明型信息不再挤占图表主区。
- Failure signal: 看板图表与 KPI 冲突、排行失真，或兼容入口与源视图分叉。
- Verification proxy: lint 通过；行为改动时加 build 与分析看板人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证数据分析页可查看 KPI、趋势图、收入概况和员工效率排行，并能进入显式帮助页

## Rollback

- Revert this delivery note and any future data dashboard route changes together.
- If regressions appear, fallback is the current local analytics dashboard composition.