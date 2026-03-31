# Incidents Delivery Unit

## Scope

- Entry route: src/app/incidents/page.tsx
- Affected users: 质控、运营、值班管理与事故复盘用户
- Rollout stage: 第二批高频路由治理说明

## User Impact

- 事故列表页承担事故搜索、级别筛选、复盘摘要和 AI 跟进行动建议。
- 当前交付单元先固定说明和验证门禁，不修改现有列表行为。
- 保持新增报告入口、事故列表、AI 摘要和复盘建议区域不变。

## Data Source

- Route type: client page with local search/filter state
- Primary data: local incidents mocks and admin AI helpers
- Downstream links: incident detail pages and AI assistant contextual links

## UI States

- Loading state: 当前为本地数据，后续接真实事故流需补搜索和筛选中的可见反馈。
- Empty state: 搜索或筛选无结果时保持 EmptyState 搜索空态。
- Error state: 事故列表与 AI 复盘建议不一致时应局部暴露，不能整页静默。
- Mobile impact: 列表卡片和 AI 区块并存，后续变更需确认窄屏滚动顺序与点击区域。

## Health Signals

- Healthy signal: 搜索、级别筛选、AI 摘要和详情链接围绕同一事故数据集保持一致。
- Failure signal: 列表数量、状态统计和 AI 复盘建议口径不一致，或详情入口错误。
- Verification proxy: lint 通过；行为改动时加 build 与事故列表流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、级别筛选、空态、AI 摘要、详情跳转五个基本路径

## Rollback

- Revert this delivery note and any future incidents route changes together.
- If later changes regress, fallback is the previous incidents list and local search/filter implementation.