# Alerts Route Delivery Unit

## Scope

- Entry route: src/app/alerts/page.tsx
- Affected users: 值班管理、护理站监控、运营协同用户
- Rollout stage: 报警中心 live read-write 收口

## User Impact

- 报警中心需要同时承接筛选、处置推进和 AI 建议边界说明。
- 页面主视图只保留真实报警摘要、真实优先队列和真实状态推进动作，不再回退到本地 `alertRecords`。
- 每条告警的解释和建议改为读取后端 AI `alert-suggestion` 能力，链路异常时展示显式 AI 不可用状态，而不是继续渲染 mock AI 文案。
- 筛选、优先级排序和卡片布局保持不变，变化点只在事实源与失败边界。

## Data Source

- Route type: client page with live queue reads and live status writes
- Primary data: `src/lib/services/admin-module-services.ts` 读取真实报警摘要与队列
- Supporting data: `src/lib/ai/admin-ai-api.ts` 的后端 `alert-suggestion` 能力
- Downstream links: AI assistant context links and alert detail related actions

## UI States

- Loading state: 首屏等待真实报警摘要、优先队列和 AI 建议返回时，要保持可见同步态。
- Empty state: 过滤后无报警时必须维持明确空态。
- Error state: 报警链路不可用时页面保留 live 错误态；AI 建议不可用时保留局部 AI 错误态，但不回退 mock 数据。
- Mobile impact: 报警卡片动作密集，后续变更要复核按钮可点击面积和折行。

## Health Signals

- Healthy signal: 筛选结果、状态流转和 AI 建议区域都围绕同一条真实报警记录保持一致，且紧急报警优先可见。
- Failure signal: 页面重新混入 `alertRecords`、mock AI 文案或 `Demo Fallback` 状态。
- Verification proxy: lint 通过；行为变更时加 build 与人工报警流回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 live 队列加载、pending -> processing -> resolved 转换、筛选组合、以及 AI 建议失败时的局部错误态

## Rollback

- Revert this route delivery note and any future alerts-specific behavior changes together.
- If later rollout introduces regressions, rollback is the previous mixed live-plus-demo alerts page, but that will restore dual source-of-truth risk.
