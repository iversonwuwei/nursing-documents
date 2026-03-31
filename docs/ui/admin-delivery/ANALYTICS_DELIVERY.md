# Analytics Delivery Unit

## Scope

- Entry route: src/app/analytics/page.tsx
- Affected users: 运营分析、院长看板、报表查看用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- analytics 路由当前复用 data-dashboard 视图，承担兼容入口和分析导航承接职责。
- 当前交付单元先固定兼容入口说明和验证门禁，不修改 re-export 结构。
- 保持 `/analytics` 与 `/data-dashboard` 在当前阶段指向同一数据分析视图。

## Data Source

- Route type: route re-export to data dashboard page
- Primary dependency: src/app/data-dashboard/page.tsx
- Downstream links: inherited analytics KPI、图表和排行展示

## UI States

- Loading state: 当前继承 data-dashboard 的本地静态视图；后续接真实分析服务时需补共享加载反馈。
- Empty state: 继承 data-dashboard 的分析空态策略。
- Error state: `/analytics` 与 `/data-dashboard` 若出现口径分叉，应显式暴露。
- Mobile impact: 当前沿用 data-dashboard 的双栏分析布局，后续拆分需单独验证窄屏承载。

## Health Signals

- Healthy signal: `/analytics` 与 `/data-dashboard` 保持一致的数据分析视图、统计口径和导航语义。
- Failure signal: 双入口图表或指标定义分叉，兼容入口失去分析承接能力。
- Verification proxy: lint 通过；行为改动时加 build 与分析页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/analytics` 进入后展示与 `/data-dashboard` 一致的数据分析看板

## Rollback

- Revert this delivery note and any future analytics route changes together.
- If regressions appear, fallback is the previous re-export wiring to data dashboard page.