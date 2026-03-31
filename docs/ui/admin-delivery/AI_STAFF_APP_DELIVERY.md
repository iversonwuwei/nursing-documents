# AI Staff App Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/staff-app/page.tsx
- Affected users: 员工端产品、运营、护理主管、AI 体验设计与质控用户
- Rollout stage: 第四批扩展路由治理说明

## User Impact

- 员工端 AI 预览页承担首页摘要、重点任务、报警提示和表达原则的原型核对。
- 当前交付单元只固定说明和验证门禁，不修改预览模块、重点任务或表达原则内容。
- 保持员工端 AI 仍强调“更短、更可执行、更强调 SLA”的原型边界。

## Data Source

- Route type: client page with local preview mocks
- Primary sources: app-ai staff modules and focus mocks
- Downstream dependency: future staff app or nani app AI 能力承接

## UI States

- Loading state: 当前为本地同步 mock；若未来接真实任务/报警聚合，需补首页摘要加载反馈。
- Empty state: 模块列表或重点任务为空时，应保持局部空态并说明当前原型缺项。
- Error state: 首页重点任务、报警提示和表达原则口径不一致时需显式暴露。
- Mobile impact: 该页虽为 Web 原型，但后续改动需验证内容是否能压缩到员工端移动首屏。

## Health Signals

- Healthy signal: 模块列表、重点任务和表达原则保持动作导向且不丢 SLA 语义。
- Failure signal: 信息过长、无法执行，或报警建议没有明确下一步动作。
- Verification proxy: lint 通过；行为改动时加 build 与员工端 AI 表达人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证模块列表、重点任务、报警提示和表达原则文案

## Rollback

- Revert this delivery note and any future staff-app AI preview route changes together.
- If regressions appear, fallback is the previous Web preview composition and mock wording.
