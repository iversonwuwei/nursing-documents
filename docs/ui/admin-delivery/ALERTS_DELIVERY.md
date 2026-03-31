# Alerts Route Delivery Unit

## Scope

- Entry route: src/app/alerts/page.tsx
- Affected users: 值班管理、护理站监控、运营协同用户
- Rollout stage: 第一批高频路由治理说明

## User Impact

- 报警中心需要同时承接筛选、处置推进和 AI 建议边界说明。
- 当前交付单元先固定范围、健康信号和回滚要求，不改已有交互。
- 保持现有报警卡片、筛选条件和状态推进按钮逻辑不变。

## Data Source

- Route type: client page with local state transitions
- Primary data: lib/data/alerts-data and mock admin AI helpers
- Downstream links: AI assistant context links and alert detail related actions

## UI States

- Loading state: 当前为本地数据，后续接真数据时要补可见加载反馈。
- Empty state: 过滤后无报警时必须维持明确空态。
- Error state: AI 建议和状态流转需要局部失败可见，不应静默失效。
- Mobile impact: 报警卡片动作密集，后续变更要复核按钮可点击面积和折行。

## Health Signals

- Healthy signal: 筛选结果、状态流转和 AI 建议区域保持一致，且紧急报警优先可见。
- Failure signal: 筛选混乱、状态推进与卡片信息不一致、AI 建议上下文错位。
- Verification proxy: lint 通过；行为变更时加 build 与人工报警流回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 pending -> processing -> resolved 转换与筛选组合仍正确

## Rollback

- Revert this route delivery note and any future alerts-specific behavior changes together.
- If later rollout introduces regressions, fallback is the previous alerts page composition and local transition rules.