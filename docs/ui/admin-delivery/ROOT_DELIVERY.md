# Root Delivery Unit

## Scope

- Entry route: src/app/page.tsx
- Affected users: admin 首页访问用户、院务运营看板用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- 根路由当前承接 admin 首页，聚合 KPI、AI 风险摘要、运营建议、护理任务和分院概览。
- 当前交付单元先固定首页职责和验证门禁，不修改首页聚合逻辑或导航行为。
- 保持根路由作为默认登录后落点，不改变与 dashboard 总览语义的衔接。

## Data Source

- Route type: client dashboard entry page
- Primary sources: elderlyList, equipmentAlarms, organizations, todayTasks, healthBrief
- Downstream dependencies: admission workflow mock store, getAiDashboardInsights, getAiDashboardActions

## UI States

- Loading state: 入住申请快照通过 external store 同步，后续接真实聚合接口时需补首页级加载反馈。
- Empty state: 今日任务、AI 摘要或健康简报为空时应保留首页框架并显式提示无数据。
- Error state: 首页聚合模块间口径不一致时应显式暴露，而不是继续显示成功态总览。
- Mobile impact: KPI 栅格、双栏模块和表格列表需要验证窄屏下的折行和滚动承载。

## Health Signals

- Healthy signal: 首页 KPI、AI 摘要和任务列表保持统一口径，并能稳定作为登录后的默认运营入口。
- Failure signal: 根路由与 dashboard 概念分裂、AI 摘要脱离首页指标，或首页首屏失去总览价值。
- Verification proxy: lint 通过；行为改动时加 build 与首页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证登录后进入 `/` 可看到首页 KPI、AI 摘要、任务列表和分院概览

## Rollback

- Revert this delivery note and any future root route changes together.
- If regressions appear, fallback is the current admin home dashboard composition.