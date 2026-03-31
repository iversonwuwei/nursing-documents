# Devices Status Delivery Unit

## Scope

- Entry route: src/app/devices/status/page.tsx
- Affected users: 设备值班、后勤监控、护理站协同用户
- Rollout stage: 第七批设备子路由治理说明

## User Impact

- 设备状态路由当前复用 equipment status 视图，承担兼容入口和状态总览承接职责。
- 当前交付单元先固定说明和验证门禁，不修改 re-export 结构或状态展示行为。
- 保持 `/devices/status` 与 `/equipment/status` 在当前阶段指向同一状态总览视图。

## Data Source

- Route type: route re-export to equipment status page
- Primary dependency: src/app/equipment/status/page.tsx
- Downstream links: inherited status filters, cards, and AI context links from equipment status view

## UI States

- Loading state: 当前继承 equipment status 的本地状态视图；后续接实时状态流时需补刷新反馈。
- Empty state: 继承 equipment status 的状态空态与缺省策略。
- Error state: `/devices/status` 与 `/equipment/status` 若出现状态口径分叉，应显式暴露。
- Mobile impact: 当前沿用 equipment status 的状态卡片和筛选布局，后续拆分需单独验证窄屏承载。

## Health Signals

- Healthy signal: `/devices/status` 与 `/equipment/status` 保持一致的状态视图、筛选结果和入口语义。
- Failure signal: 双入口状态口径错位，或兼容入口失去状态总览能力。
- Verification proxy: lint 通过；行为改动时加 build 与设备状态人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/devices/status` 进入后展示状态总览视图与 `/equipment/status` 一致

## Rollback

- Revert this delivery note and any future devices status route changes together.
- If regressions appear, fallback is the previous re-export wiring to equipment status page.
