# Health Monitoring Delivery Unit

## Scope

- Entry route: src/app/health-monitoring/page.tsx
- Affected users: 监控值班、护理主管、异常升级协同用户
- Rollout stage: 第二批高频路由治理说明

## User Impact

- 健康监测页承担实时指标、异常筛选、AI 风险解释和 AI 跟进动作板。
- 当前交付单元先固化范围和验证标准，不修改现有筛选与卡片行为。
- 保持全部/异常切换、趋势图和 AI 运营中心入口不变。

## Data Source

- Route type: client page with local state toggle
- Primary data: health-data mocks and admin AI helpers
- Downstream links: AI assistant contextual links for risk, trend, and follow-up boards

## UI States

- Loading state: 当前为本地数据，后续接实时数据需补轮询或刷新中的可见状态。
- Empty state: 异常筛选后若无异常对象，需要保持可见空态或清晰零结果表达。
- Error state: 趋势图和 AI 卡片需要局部降级，不应因单一数据源异常导致整页不可用。
- Mobile impact: 图表、KPI 与 AI 卡片并存，后续变更需复核窄屏下阅读顺序。

## Health Signals

- Healthy signal: 异常计数、对象卡片、AI 解释与 follow-up board 指向同一健康上下文。
- Failure signal: 视图切换后统计口径错乱，或 AI 上下文链接与对象不一致。
- Verification proxy: lint 通过；行为改动时加 build 和健康监测流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证全部/异常切换、趋势图、AI 风险解释和跟进动作板

## Rollback

- Revert this delivery note and any future health-monitoring route changes together.
- If later changes regress, fallback is the previous dashboard composition and local toggle logic.