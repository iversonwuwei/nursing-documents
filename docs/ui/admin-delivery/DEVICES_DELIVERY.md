# Devices Delivery Unit

## Scope

- Entry route: src/app/devices/page.tsx
- Affected users: 设备运维、护理站值班、后勤协同用户
- Rollout stage: 第六批设备与组织扩展路由治理说明

## User Impact

- 设备总览路由当前直接复用 equipment 路由内容，承担设备入口兼容和导航承接职责。
- 当前交付单元先固定说明和验证门禁，不修改 re-export 结构或设备展示行为。
- 保持 `/devices` 与 `/equipment` 入口在当前原型阶段指向同一设备总览视图。

## Data Source

- Route type: route re-export to equipment page
- Primary dependency: src/app/equipment/page.tsx
- Downstream links: device detail, realtime monitor, and AI assistant context links inherited from equipment view

## UI States

- Loading state: 当前为复用本地设备列表视图；若后续拆分为独立设备域，需补独立加载反馈。
- Empty state: 继承 equipment 视图的搜索空态与列表空态行为。
- Error state: `/devices` 和 `/equipment` 若出现内容漂移，应显式暴露而不是静默分叉。
- Mobile impact: 当前沿用 equipment 视图的表格与 CTA 表现，后续拆分时需单独验证窄屏承载。

## Health Signals

- Healthy signal: `/devices` 路由与 `/equipment` 路由在当前阶段保持一致的设备总览和入口语义。
- Failure signal: 双入口内容分叉、链接漂移，或兼容入口失去设备总览能力。
- Verification proxy: lint 通过；行为改动时加 build 与设备入口人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/devices` 进入后展示设备总览、详情入口、监控入口与 AI 链路

## Rollback

- Revert this delivery note and any future devices route changes together.
- If regressions appear, fallback is the previous re-export wiring to equipment page.
