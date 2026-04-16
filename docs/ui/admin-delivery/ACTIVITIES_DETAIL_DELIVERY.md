# Activities Detail Delivery Unit

## Scope

- Entry route: src/app/activities/[id]/page.tsx
- Affected users: 活动运营、前台协同、护理主管
- Rollout stage: 第十七批分析与详情页主区收口

## User Impact

- 活动详情页承担从活动列表进入后的单活动核对、待发布确认和编辑入口承接。
- 新建活动在发布前会先在详情页显示待发布状态与说明，避免只在列表里短暂可见。
- 返回列表、活动 KPI 卡片和详情字段仍保留，但页面说明、推荐路径和帮助入口需后置到上下文区与帮助页。

## Data Source

- Route type: client detail route with dynamic route param and shared workflow subscription
- Primary data: shared operations workflow live activity records keyed by id
- Upstream links: activities list route and activities new route

## UI States

- Loading state: 当前为本地同步 workflow store，后续接远程详情接口时需补拉取反馈与 skeleton。
- Empty state: 当前未知 id 会回退到首条记录；接真实接口时需显式区分 not found 与空态。
- Error state: 动态 id、待发布状态和详情字段若不一致，需局部暴露，不应静默掩盖。
- Mobile impact: 详情页头、状态卡、统计卡、对象事实卡和帮助入口在窄屏下仍需保持主操作可见。

## Health Signals

- Healthy signal: 从列表或新建回流进入后，标题、状态卡、发布动作和详情字段保持一致，说明型内容不会挤占对象核对主区。
- Failure signal: 列表状态已发布但详情仍待发布，或动态 id 与内容错位。
- Verification proxy: docs build、lint、build 通过；人工验证待发布活动在详情页发布后同步回写列表。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证从活动列表进入详情、待发布活动发布、返回列表、统计卡、对象事实和帮助入口展示

## Rollback

- Revert this delivery note together with activities detail route and shared operations workflow 接入。
- If regressions appear, fallback is the previous local detail mock and list-to-detail link behavior.
