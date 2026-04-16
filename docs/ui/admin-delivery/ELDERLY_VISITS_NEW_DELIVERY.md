# Elderly Visits New Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/new/page.tsx
- Affected users: 前台、护理主管、家属沟通协同人员
- Rollout stage: 探视预约新建页主区收口与帮助后置批次

## User Impact

- “预约探视”按钮改为真实预约入口。
- 新预约提交后先进入待审核，再由探视列表页执行审核通过。
- admin 新建预约继续作为前台手工录入入口存在，但它现在要和 family 端回流到同一待审核队列共用状态口径。
- 首批仍不接真实家属端排班系统，但 mock 结构需允许区分 admin 手工录入与 family 回流预约来源。
- 主区只保留预约闭环说明和表单录入，完整说明与操作边界后置到信息轨和帮助入口。

## Data Source

- Route type: client form page
- Primary sink: care service workflow shared store
- Downstream link: `/elderly/visits?selected=...&entry=elderly-visits-new`
- Shared review boundary: same visits workflow store as family-origin pending approvals

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认空表单，允许现场和视频探视。
- Error state: 老人、访客、关系、联系电话、日期、时间和探视方式缺失时显式提示。
- Mobile impact: 预约页需在窄屏下维持清晰的时间和访客字段顺序，同时不和 family 回流审核信息混淆。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: admin 新建预约 -> 列表回流 -> 待审核提示 -> 审核通过状态切换；并确认 family 回流预约与 admin 手工预约共享同一审核口径但保留来源区分

## Rollback

- Revert this note together with `/elderly/visits/new` and care service workflow visit changes.
- Fallback is to restore the previous static visits list behavior.
