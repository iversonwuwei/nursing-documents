# AI QA Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/qa/page.tsx
- Affected users: 运营主管、护理主管、AI 治理与审计协同用户
- Rollout stage: AI 根入口拆页后的即时问答独立交付单元

## User Impact

- 即时问答从 `/ai-assistant` 根页拆到独立 `/ai-assistant/qa`，用户不再需要在总览页中寻找具体提问入口。
- 问答页只承载“输入问题 -> 获取回答 -> 判断是否回到业务页执行”这一条闭环，不再混入 AI 总览、治理摘要或审计列表。
- AI 根页保留为总览与分流入口，日志页继续保留为审计复盘，不承担即时回答生成。

## Data Source

- Route type: client AI Q&A page with query-context propagation
- Primary sources: AI tracking context helpers, `/api/dashboard/overview`, `sendAdminAiChat`
- Downstream dependencies: `/api/ai/*` Next proxy, Admin BFF AI chat route, `AdminAiNav`

## UI States

- Loading state: 提交问题后按钮进入生成中状态，避免重复触发。
- Empty state: 无上下文时退化为通用问答页；未发起第一轮对话前显示占位回答。
- Error state: AI 模块未启用、链路失败或服务不可用时需显式展示错误，不静默回退到 mock 回答。
- Mobile impact: 预设问题、输入区和回答区保持单列顺序，右侧仅保留上下文和链路状态。

## Health Signals

- Healthy signal: 问答页稳定展示预设问题、自定义输入和真实回答结果，并保留来源上下文。
- Failure signal: 问答逻辑重新回流到 AI 根页，或问答页开始承载与回答无关的总览/审计内容。
- Verification proxy: lint 通过；行为改动时加 build 与问答页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 进入 `/ai-assistant/qa`，确认可提交预设问题和自定义问题；带 query 上下文进入时右侧能看到来源与关注点

## Rollback

- Revert this delivery note together with `/ai-assistant/qa` route and the root-page split changes.
- If regressions appear, fallback is restoring the old root-page Q&A panel and removing the dedicated QA page.
