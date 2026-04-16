# AI Logs Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/logs/page.tsx
- Affected users: AI 运营、审计、质控和值班管理用户
- Rollout stage: 第十七批分析与详情页主区收口

## User Impact

- AI 日志页承担上下文追踪、按场景收窄日志、关键词检索和结果审计。
- 主区应只保留追踪 banner、筛选条和日志明细，页面定位、过滤边界和帮助说明迁移到后置上下文与帮助页。
- 保持 contextual banner、场景过滤和日志明细结构不变。

## Data Source

- Route type: client page with query-param context and local filter state
- Primary sources: ai-context helpers and admin AI mock logs
- Downstream dependency: AdminAiNav and AI tracking context helpers

## UI States

- Loading state: 当前为本地日志，后续接真实审计中心需补首屏和过滤中的可见反馈。
- Empty state: 当前筛选条件下无日志时维持明确空态文案。
- Error state: tracking context 解析错误或日志缺失时应保留页面骨架并显式失败。
- Mobile impact: 过滤条、日志明细、后置上下文卡和帮助入口需验证小屏下检索与场景选择可用性。

## Health Signals

- Healthy signal: 来源上下文、默认关键词、场景过滤和日志明细保持一致，说明型内容不再压缩审计主区。
- Failure signal: context 映射错位、默认过滤失真或空态缺失。
- Verification proxy: lint 通过；行为改动时加 build 与日志检索流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证带 query context 进入、默认关键词建议、场景过滤、零结果空态和帮助入口

## Rollback

- Revert this delivery note and any future ai logs route changes together.
- If regressions appear, fallback is the previous log filtering and context rendering behavior.