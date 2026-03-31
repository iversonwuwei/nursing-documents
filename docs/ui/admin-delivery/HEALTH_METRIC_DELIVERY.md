# Health Metric Delivery Unit

## Scope

- Entry route: src/app/health/[metric]/page.tsx
- Affected users: 健康专题查看、运营配置、护理分析用户
- Rollout stage: 第十二批健康域路由治理说明

## User Impact

- 健康指标子路由当前根据 metric 参数映射标准模块页配置，用于承接不同健康专题视图。
- 当前交付单元先固定参数路由职责、验证门禁和 notFound 边界，不修改标准模块配置。
- 保持合法 metric 命中标准模块页，非法 metric 返回 notFound 的当前行为不变。

## Data Source

- Route type: server route with async params and standard config lookup
- Primary source: healthMetricPages config map
- Downstream dependency: shared StandardModulePage component and next/navigation notFound

## UI States

- Loading state: 当前标准模块页按配置直接渲染；后续若 metric 页面接真实数据，应沿用标准页加载反馈。
- Empty state: 配置存在但无模块内容时应保持标准页级空态一致性。
- Error state: 未命中 metric 时通过 notFound 显式失败；命中后若配置漂移也应显式暴露。
- Mobile impact: 由标准模块页统一承担窄屏布局责任，但不同 metric 仍需验证模块密度。

## Health Signals

- Healthy signal: 合法 metric 稳定映射到对应标准模块页，非法 metric 稳定返回 notFound。
- Failure signal: 参数映射错位、配置缺失静默回退，或不同 metric 渲染到错误专题页。
- Verification proxy: lint 通过；行为改动时加 build 与健康 metric 路由人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证一个合法 metric 可正常渲染标准模块页，非法 metric 返回 notFound

## Rollback

- Revert this delivery note and any future health metric route changes together.
- If regressions appear, fallback is the current config lookup and notFound boundary.