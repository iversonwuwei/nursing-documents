# Incident Detail Delivery Unit

## Scope

- Entry route: src/app/incidents/[id]/page.tsx
- Affected users: 质控、值班管理、运营复盘用户
- Rollout stage: 第十五批运营新建闭环治理说明

## User Impact

- 事故详情页承担事故说明、处理过程、附件材料和 AI 复盘/跟进建议的合流入口。
- 新建事故在待分派和处理中两个阶段都可在详情页继续推进状态，而不是只能回列表操作。
- 现有 tabs 和 AI contextual links 保留，但数据来源已切换到共享 workflow。

## Data Source

- Route type: client detail route with shared workflow lookup and local tab state
- Primary sources: shared operations workflow live incident records and admin AI helpers
- Upstream links: incidents list route and incidents new route
- Downstream links: AI assistant inference/logs contextual links and incidents list back navigation

## UI States

- Loading state: 当前为本地 workflow 数据映射，后续接真实详情接口时需补对象切换加载反馈。
- Empty state: 未命中 id 时当前回退首条事故；接真数据后需显式 not found 策略。
- Error state: 事故主信息、状态推进、处理过程与 AI 建议不一致时应局部暴露。
- Mobile impact: tabs、状态卡、AI 卡片和操作按钮并存，需验证窄屏切换与按钮触达。

## Health Signals

- Healthy signal: 同一事故的 tabs、待分派/处理中/已结案状态、AI 解释和 contextual links 保持一致。
- Failure signal: 列表已开始处置但详情仍显示待分派，或 tab 内容与当前事故错位。
- Verification proxy: docs build、lint、build 通过；人工验证待分派 -> 开始处置 -> 申请结案流程。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 incidents list -> detail -> 状态推进 -> tab switching -> AI link 的事故上下文一致性

## Rollback

- Revert this delivery note together with incidents detail route and shared operations workflow 接入。
- If later regressions appear, fallback is the previous detail layout, tabs, and local mock mapping.