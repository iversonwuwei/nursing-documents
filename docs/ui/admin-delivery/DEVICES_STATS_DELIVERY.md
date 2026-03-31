# Devices Stats Delivery Unit

## Scope

- Entry route: src/app/devices/stats/page.tsx
- Affected users: 设备运维分析、后勤管理、运营复盘用户
- Rollout stage: 第七批设备子路由治理说明

## User Impact

- 设备统计路由当前复用 equipment stats 视图，承担兼容入口和统计分析承接职责。
- 当前交付单元先固定说明和验证门禁，不修改 re-export 结构或统计展示行为。
- 保持 `/devices/stats` 与 `/equipment/stats` 在当前阶段指向同一统计分析视图。

## Data Source

- Route type: route re-export to equipment stats page
- Primary dependency: src/app/equipment/stats/page.tsx
- Downstream links: inherited statistics cards, charts, and AI context links from equipment stats view

## UI States

- Loading state: 当前继承 equipment stats 的本地统计视图；后续接真实统计数据时需补图表加载反馈。
- Empty state: 继承 equipment stats 的统计空态与缺省策略。
- Error state: `/devices/stats` 与 `/equipment/stats` 若出现统计口径分叉，应显式暴露。
- Mobile impact: 当前沿用 equipment stats 的图表与统计卡布局，后续拆分需单独验证窄屏承载。

## Health Signals

- Healthy signal: `/devices/stats` 与 `/equipment/stats` 保持一致的统计指标、图表和入口语义。
- Failure signal: 双入口统计口径错位，或兼容入口失去统计分析能力。
- Verification proxy: lint 通过；行为改动时加 build 与设备统计人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/devices/stats` 进入后展示统计分析视图与 `/equipment/stats` 一致

## Rollback

- Revert this delivery note and any future devices stats route changes together.
- If regressions appear, fallback is the previous re-export wiring to equipment stats page.
