# AI Rules Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/rules/page.tsx
- Affected users: AI 治理、运营管理、质控用户
- Rollout stage: 第三批高频路由治理说明

## User Impact

- AI 规则页承担规则启停、上下文治理边界和回滚路径展示。
- 当前交付单元先固定说明和验证要求，不修改规则切换演示行为。
- 保持 contextual governance banner、规则启停列表和发布/回滚卡片不变。

## Data Source

- Route type: client page with query-param context and local rule toggle state
- Primary sources: AI rule mocks, ai-context helpers, related logs helpers
- Downstream link: logs audit route with appended tracking context

## UI States

- Loading state: 当前为本地规则 mock，后续接配置中心需补拉取和切换中反馈。
- Empty state: 当前若无相关规则或相关日志，应保持局部空态而不破坏治理框架。
- Error state: rule context 与 related logs/rules mapping 错位时需显式暴露。
- Mobile impact: 规则卡片信息量大，后续变更需验证窄屏下启停按钮与回滚说明可读性。

## Health Signals

- Healthy signal: tracking context、治理边界、规则启停和转到日志审计的路径保持一致。
- Failure signal: 规则映射与日志建议错位，或治理边界缺乏可回滚说明。
- Verification proxy: lint 通过；行为改动时加 build 与规则治理流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证带 context 进入、规则启停、治理边界提示和跳转日志审计

## Rollback

- Revert this delivery note and any future ai rules route changes together.
- If regressions appear, fallback is the previous mock governance state and contextual link behavior.