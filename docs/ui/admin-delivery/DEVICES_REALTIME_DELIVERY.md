# Devices Realtime Delivery Unit

## Scope

- Entry route: src/app/devices/realtime/page.tsx
- Affected users: 实时监控值班、设备运维、护理站协同用户
- Rollout stage: 第七批设备子路由治理说明

## User Impact

- 设备实时监控路由当前复用 equipment monitor 视图，承担兼容入口和实时监控承接职责。
- 当前交付单元先固定说明和验证门禁，不修改 re-export 结构或实时监控展示行为。
- 保持 `/devices/realtime` 与 `/equipment/monitor` 在当前阶段指向同一实时监控视图。

## Data Source

- Route type: route re-export to equipment monitor page
- Primary dependency: src/app/equipment/monitor/page.tsx
- Downstream links: inherited monitor actions and AI context links from equipment monitor view

## UI States

- Loading state: 当前继承 equipment monitor 的本地实时视图；后续接真实流式数据时需补刷新与轮询反馈。
- Empty state: 继承 equipment monitor 的监控空态与缺省策略。
- Error state: `/devices/realtime` 与 `/equipment/monitor` 若出现监控内容漂移，应显式暴露。
- Mobile impact: 当前沿用 equipment monitor 的监控卡片与图表布局，后续拆分需单独验证窄屏可读性。

## Health Signals

- Healthy signal: `/devices/realtime` 与 `/equipment/monitor` 保持一致的实时监控内容和入口语义。
- Failure signal: 双入口内容分叉、实时状态口径不一致，或兼容入口失去监控能力。
- Verification proxy: lint 通过；行为改动时加 build 与实时监控人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/devices/realtime` 进入后展示实时监控视图与 `/equipment/monitor` 一致

## Rollback

- Revert this delivery note and any future devices realtime route changes together.
- If regressions appear, fallback is the previous re-export wiring to equipment monitor page.
