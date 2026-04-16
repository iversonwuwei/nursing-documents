# AI Assistant Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/page.tsx
- Affected users: 运营主管、护理主管、AI 治理与审计协同用户
- Rollout stage: 第十三批 AI 根入口治理说明

## User Impact

- AI 运营入口当前只承接 Admin 端 AI 总览、上下文透传和子页导航。
- 当前交付单元把即时问答从根页拆到独立 `/ai-assistant/qa`，避免根页同时承担入口和工具执行。
- 保持 AI 入口为问答页、推理详情、规则治理、问答日志、员工端预览和家属端预览的总导航页。

## Data Source

- Route type: client AI hub page with query-context propagation
- Primary sources: AI tracking context helpers, admission workflow external store, admin AI dashboard summary helpers
- Downstream dependencies: AdminAiNav, appendAiTrackingContext, readAiTrackingContext and AI child-route links including `/ai-assistant/qa`

## UI States

- Loading state: 当前根页只处理 AI 总览摘要；即时问答加载反馈转移到 `/ai-assistant/qa`。
- Empty state: 无 trackingContext 时应退化为通用 AI 总览页；无 dashboardInsights 时应显式提示暂无摘要信号。
- Error state: 上下文透传、AI 总览和子页导航口径不一致时需显式暴露，而不是静默跳到错误目标页。
- Mobile impact: 顶部上下文卡与子页导航区在窄屏下需要验证折叠顺序和按钮可达性；问答页移动端单独验证。

## Health Signals

- Healthy signal: AI 根入口稳定展示总览摘要，并将来源上下文正确透传到 inference、rules、logs 等子页。
- Failure signal: trackingContext 丢失、目标子页跳错，或根页重新混入即时问答执行逻辑。
- Verification proxy: lint 通过；行为改动时加 build 与 AI 根入口人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证普通进入 `/ai-assistant` 可看到总览；带 query 上下文进入时可继续透传到子页

## Rollback

- Revert this delivery note and any future ai-assistant root route changes together.
- If regressions appear, fallback is the current AI hub layout and local context propagation behavior.
