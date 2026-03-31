# Devices Detail Delivery Unit

## Scope

- Entry route: src/app/devices/[id]/page.tsx
- Affected users: 设备运维、护理站值班、后勤协同用户
- Rollout stage: 第七批设备子路由治理说明

## User Impact

- 设备详情路由当前复用 equipment detail 视图，承担兼容入口和对象级设备查看职责。
- 当前交付单元先固定说明和验证门禁，不修改 detail re-export 结构或设备详情行为。
- 保持 `/devices/[id]` 与 `/equipment/[id]` 在当前阶段指向同一对象详情视图。

## Data Source

- Route type: dynamic route re-export to equipment detail page
- Primary dependency: src/app/equipment/[id]/page.tsx
- Downstream links: inherited AI context links and device detail actions from equipment detail view

## UI States

- Loading state: 当前继承 equipment detail 的本地对象映射行为；后续拆分域模型时需补对象切换反馈。
- Empty state: 继承 equipment detail 对未知 id 的回退策略；若后续接真实接口需显式 not found 策略。
- Error state: `/devices/[id]` 与 `/equipment/[id]` 若在对象映射或详情内容上分叉，应显式暴露。
- Mobile impact: 当前沿用 equipment detail 的卡片布局，后续若拆分需单独验证窄屏详情可达性。

## Health Signals

- Healthy signal: `/devices/[id]` 与 `/equipment/[id]` 对同一设备 id 展示一致的详情内容和入口链路。
- Failure signal: 兼容详情入口对象错位，或双入口内容分叉。
- Verification proxy: lint 通过；行为改动时加 build 与设备详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/devices/[id]` 与 `/equipment/[id]` 的详情内容和返回链路一致

## Rollback

- Revert this delivery note and any future devices detail route changes together.
- If regressions appear, fallback is the previous re-export wiring to equipment detail page.
