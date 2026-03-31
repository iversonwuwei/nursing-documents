# Incident Detail Delivery Unit

## Scope

- Entry route: src/app/incidents/[id]/page.tsx
- Affected users: 质控、值班管理、运营复盘用户
- Rollout stage: 第三批高频路由治理说明

## User Impact

- 事故详情页承担事故说明、处理过程、附件材料和 AI 复盘/跟进建议的合流入口。
- 当前交付单元先固化范围和验证要求，不修改标签页、按钮或 AI 建议行为。
- 保持现有详情 tabs、进度动作和 AI contextual links 不变。

## Data Source

- Route type: client detail route with local mock lookup and local tab state
- Primary sources: incident detail mocks and admin AI helpers
- Downstream links: AI assistant inference/logs contextual links and incidents list back navigation

## UI States

- Loading state: 当前为本地数据映射，后续接真实详情接口时需补对象切换加载反馈。
- Empty state: 未命中 id 时当前回退默认事故；接真数据后需显式 not found 策略。
- Error state: 事故主信息、处理过程与 AI 建议不一致时应局部暴露。
- Mobile impact: tabs、AI 卡片和操作按钮并存，后续变更需验证窄屏切换与按钮触达。

## Health Signals

- Healthy signal: 同一事故的 tabs、AI 解释、follow-up 建议和 contextual links 保持一致。
- Failure signal: tab 内容错位、对象映射错误或 AI 上下文链接偏离当前事故。
- Verification proxy: lint 通过；行为改动时加 build 与详情流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 incidents list -> detail -> tab switching -> AI link 的事故上下文一致性

## Rollback

- Revert this delivery note and any future incident-detail route changes together.
- If later regressions appear, fallback is the previous detail layout, tabs, and local mock mapping.