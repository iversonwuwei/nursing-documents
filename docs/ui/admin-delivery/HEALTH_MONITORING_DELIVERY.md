# Health Monitoring Delivery Unit

## Scope

- Entry route: src/app/health-monitoring/page.tsx
- Affected users: 监控值班、护理主管、异常升级协同用户
- Rollout stage: 第十六批健康监测主区收口与帮助页后置

## User Impact

- 健康监测页承担实时指标、异常筛选、趋势图、异常对象卡片和当班跟进动作板。
- AI 风险解释、趋势解读和长说明后置到右侧信息轨与帮助页，首屏主区只保留直接影响值班动作的区块。
- 保持全部/异常切换、趋势图和 AI 运营中心入口语义不变，但补齐异常筛选零结果表达和帮助页承接。

## Data Source

- Route type: client page with local state toggle
- Primary data: health-data mocks and admin AI helpers
- Downstream links: AI assistant contextual links for risk, trend, and follow-up boards, plus help route

## UI States

- Loading state: 当前为本地数据，后续接实时数据需补轮询或刷新中的可见状态。
- Empty state: 异常筛选后若无异常对象，需要在主工作区保持显式零结果空态，而不是静默留白。
- Error state: 趋势图和右轨 AI 卡片需要局部降级，不应因单一数据源异常导致整页不可用。
- Mobile impact: 图表、KPI、异常对象卡片、右轨 AI 卡片与帮助入口并存，后续变更需复核窄屏下阅读顺序。

## Health Signals

- Healthy signal: 异常计数、对象卡片、跟进动作板、右轨 AI 解释、帮助页入口与上下文链接指向同一健康对象集。
- Failure signal: 视图切换后统计口径错乱，异常筛选无结果时无清晰反馈，或 AI 上下文链接与对象不一致。
- Verification proxy: docs build、lint、build 通过；人工验证全部/异常切换、零结果空态、趋势图和帮助页回跳。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证全部/异常切换、异常零结果空态、趋势图、AI 风险解释、跟进动作板和帮助页跳转

## Rollback

- Revert this delivery note together with health-monitoring route and help route changes.
- If later changes regress, fallback is the previous dashboard composition and local toggle logic.