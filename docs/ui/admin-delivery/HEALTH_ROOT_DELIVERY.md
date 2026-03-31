# Health Root Delivery Unit

## Scope

- Entry route: src/app/health/page.tsx
- Affected users: 健康监控查看、护理主管、异常处置协同用户
- Rollout stage: 第十二批健康域路由治理说明

## User Impact

- health 根路由当前复用 health-monitoring 视图，承担兼容入口和健康域导航承接职责。
- 当前交付单元先固定兼容入口说明和验证门禁，不修改 re-export 结构。
- 保持 `/health` 与 `/health-monitoring` 在当前阶段指向同一健康监测视图。

## Data Source

- Route type: route re-export to health-monitoring page
- Primary dependency: src/app/health-monitoring/page.tsx
- Downstream links: inherited KPI、异常筛选、趋势图和 AI 上下文入口

## UI States

- Loading state: 当前继承 health-monitoring 的本地视图；后续接真实健康监测服务时需补共享加载反馈。
- Empty state: 继承 health-monitoring 的异常筛选空态和零结果表达。
- Error state: `/health` 与 `/health-monitoring` 若出现口径分叉，应显式暴露。
- Mobile impact: 当前沿用健康监测页布局，后续拆分需单独验证窄屏承载。

## Health Signals

- Healthy signal: `/health` 与 `/health-monitoring` 保持一致的健康监测视图、统计口径和导航语义。
- Failure signal: 双入口健康指标或异常统计分叉，兼容入口失去健康总览能力。
- Verification proxy: lint 通过；行为改动时加 build 与健康入口人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/health` 进入后展示与 `/health-monitoring` 一致的健康监测看板

## Rollback

- Revert this delivery note and any future health root route changes together.
- If regressions appear, fallback is the previous re-export wiring to health-monitoring page.