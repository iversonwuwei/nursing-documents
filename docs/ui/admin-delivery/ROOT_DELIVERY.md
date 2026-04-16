# Root Delivery Unit

## Scope

- Entry route: src/app/page.tsx
- Affected users: admin 首页访问用户、院务运营看板用户
- Rollout stage: 首页 Fluent 试点与入口收敛

## User Impact

- 根路由继续作为默认登录后落点，但页面职责进一步收敛为“运营优先级入口”。
- 首页首屏只保留总览、优先动作、专页下钻和少量高频入口；AI 摘要、趋势图和分析型模块从首页撤出，避免首页再次变成混合工作台。
- Fluent 试点只在首页落一轮更明确的企业工作台视觉层级，用来验证这套语言是否适合后续复制到其他高频页。

## Data Source

- Route type: client dashboard entry page
- Primary sources: dashboard aggregate, equipmentAlarms, admission workflow snapshot, session platform state
- Downstream dependencies: admission workflow mock store, dashboard aggregate proxy, tenant package/session state

## UI States

- Loading state: 聚合快照同步中时首页仍保留总览框架与主入口，但显式显示 `Syncing`，不渲染装饰性占位分析块。
- Empty state: 没有高优先级积压时，首页仍保留动作卡并提示可进入常规巡检或复盘。
- Error state: 聚合失败时保留入口闭环并显示 `Live Unavailable`，不再回退到 AI 摘要或趋势图类伪补位。
- Mobile impact: 首页继续是单列纵向阅读；Fluent 试点下需要确保主 CTA、专页下钻和场景切换都能在窄屏直接点达。

## Health Signals

- Healthy signal: 首页在首屏即可完成“判断优先级 -> 进入专页”的闭环，且视觉层级明显优先主动作而非说明块。
- Failure signal: 根路由重新混入分析型图表、AI 长摘要或执行明细，导致首页再次承担多个心智模型。
- Verification proxy: lint 通过；行为改动时加 build 与首页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证登录后进入 `/` 只看到总览、优先动作、专页入口和轻量上下文；AI 与趋势解释不再出现在首页主闭环中

## Rollback

- Revert this delivery note and any future root route changes together.
- If regressions appear, fallback is the previous homepage composition before the Fluent 试点收敛。
