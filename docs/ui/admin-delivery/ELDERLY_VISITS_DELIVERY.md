# Elderly Visits Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/page.tsx
- Affected users: 前台接待、家属沟通、探视审核用户
- Rollout stage: 第十一批长者工作流子路由治理说明

## User Impact

- 探视记录页当前承接探视统计、搜索筛选、AI 探视建议和记录审核入口。
- 当前交付单元先固定探视入口职责和验证门禁，不修改探视状态或审核按钮行为。
- 保持探视记录与 AI 建议在同一页面联动展示。

## Data Source

- Route type: client visits page with local search state
- Primary sources: VISITS mock data and family AI visit suggestions
- Downstream dependency: local action buttons for audit or detail flow

## UI States

- Loading state: 当前搜索和统计为本地即时响应；后续接真实预约服务时需补提交和审核反馈。
- Empty state: 搜索后无匹配探视记录时应显式提示无结果，而不是保留空表格。
- Error state: 统计卡、AI 建议和探视记录状态不一致时需显式暴露。
- Mobile impact: 四列统计卡、AI 建议卡片和探视表格在窄屏下需要验证阅读顺序和横向滚动。

## Health Signals

- Healthy signal: 探视统计、AI 建议和记录列表在同一探视口径下保持一致。
- Failure signal: 今日统计、待审核数量与列表状态不一致，或 AI 建议脱离当前探视记录。
- Verification proxy: lint 通过；行为改动时加 build 与探视流程人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索过滤、AI 探视助手和探视记录表可共同表达当前探视状态

## Rollback

- Revert this delivery note and any future elderly visits route changes together.
- If regressions appear, fallback is the current local visits dashboard behavior.