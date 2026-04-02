# Elderly Visits Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/page.tsx
- Affected users: 前台接待、家属沟通、探视审核用户
- Rollout stage: Family 预约探视回流到 admin 审核闭环的首批治理扩展

## User Impact

- 探视记录页不再只承接 admin 端手工新建预约，也需要承接 family 端家属预约回流后的待审核队列。
- 审核用户需要在同一页看到预约来源、是否命中 AI 风险、是否已经在家属端自动通过，以及哪些预约仍需人工放行或驳回。
- 保持探视记录、AI 探视建议和审核动作在同一页面联动展示，但把“待审核入口”升级为真正的审核工作台而不是只有一个通过按钮。

## Data Source

- Route type: client visits page with local search state
- Primary sources: care service workflow visit store, family AI visit suggestions, family visit workflow backflow mock
- Downstream dependency: local approve or reject actions, family-origin visit context banner, selected record review state

## UI States

- Loading state: 当前搜索、筛选和审核为本地即时响应；后续接真实预约服务时需补 family 回流延迟与审核提交反馈。
- Empty state: 搜索后无匹配探视记录时显式提示无结果；若当前没有待审核预约，也要明确显示“待审核队列为空”。
- Error state: 待审核统计、family 回流来源、AI 风险摘要和列表状态不一致时需显式暴露。
- Mobile impact: 待审核摘要卡、family 来源 banner、审核动作和探视表格在窄屏下需要维持清晰顺序和可点击性。

## Health Signals

- Healthy signal: family 端回流的待审核预约会进入同一 visits 审核队列，统计卡、来源标签、AI 风险摘要和审核结果保持一致；审核通过或驳回后，当前记录状态和摘要区同步刷新。
- Failure signal: family 端预约未进入待审核队列、来源信息丢失、AI 风险与审核动作脱节、待审核数量与列表不一致，或审核后仍停留在旧状态。
- Verification proxy: lint 通过；行为改动时加 build；手工回归覆盖 family-origin 预约进入待审核、审核通过、驳回三条路径。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 family-origin 待审核预约进入 visits 队列、选中后显示 family 来源和 AI 风险、执行通过或驳回后状态回流；验证搜索过滤、AI 探视助手和探视记录表仍可共同表达当前探视状态

## Rollback

- Revert this delivery note and any future elderly visits route changes together.
- If regressions appear, fallback is the current local visits dashboard behavior with only手工新建 -> 待审核 -> 通过预约，不展示 family 回流上下文或驳回动作。
