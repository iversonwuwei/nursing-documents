# Elderly Visits New Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/new/page.tsx
- Affected users: 前台、护理主管、家属沟通协同人员
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- “预约探视”按钮改为真实预约入口。
- 新预约提交后先进入待审核，再由探视列表页执行审核通过。
- 首批仅承接 admin 端预约与审核提示，不接真实家属端排班系统。

## Data Source

- Route type: client form page
- Primary sink: care service workflow shared store
- Downstream link: `/elderly/visits?selected=...&entry=elderly-visits-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认空表单，允许现场和视频探视。
- Error state: 老人、访客、关系、联系电话、日期、时间和探视方式缺失时显式提示。
- Mobile impact: 预约页需在窄屏下维持清晰的时间和访客字段顺序。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 新建预约 -> 列表回流 -> 待审核提示 -> 审核通过状态切换

## Rollback

- Revert this note together with `/elderly/visits/new` and care service workflow visit changes.
- Fallback is to restore the previous static visits list behavior.