# AI Family App Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/family-app/page.tsx
- Affected users: 家属端产品、运营、AI 体验设计与质控用户
- Rollout stage: 第四批扩展路由治理说明

## User Impact

- 家属端 AI 预览页承担家属友好表达、探视建议和模块迁移前的原型核对。
- 当前交付单元只固定说明和验证门禁，不修改预览模块、状态摘要和探视建议 mock 行为。
- 保持这一路由仍是 Web 预览，不假装已经接入真实家属端发布链路。

## Data Source

- Route type: client page with local preview mocks
- Primary sources: app-ai family modules, family summaries, visit suggestion mocks
- Downstream dependency: future family app delivery alignment

## UI States

- Loading state: 当前为本地同步 mock；若未来接真实汇总能力需补摘要和建议加载反馈。
- Empty state: 模块、状态摘要或探视建议为空时，应保留局部空态并说明当前预览缺项。
- Error state: 家属友好文案与 underlying mock 口径不一致时需显式暴露。
- Mobile impact: 该页本身为 Web 原型，但后续变更需验证卡片结构能否平滑迁移到窄屏表达。

## Health Signals

- Healthy signal: 模块列表、状态摘要和探视建议保持家属友好、透明且结论导向。
- Failure signal: 预览输出过于技术化、过度承诺，或与真实家属沟通边界不一致。
- Verification proxy: lint 通过；行为改动时加 build 与家属端 AI 表达人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证模块列表、摘要文案、探视建议和表达边界

## Rollback

- Revert this delivery note and any future family-app AI preview route changes together.
- If regressions appear, fallback is the previous Web preview composition and mock wording.
