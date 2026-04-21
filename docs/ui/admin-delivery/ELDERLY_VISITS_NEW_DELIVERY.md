# Elderly Visits New Delivery Unit

## Scope

- Entry route: src/app/elderly/visits/new/page.tsx
- Affected users: 前台、护理主管、家属沟通协同人员
- Rollout stage: 预约创建从 mock `care-service-workflow` 切换到真实 Visit Service 写通道

## Change Summary

- 新建预约改为通过 Admin BFF `POST /api/admin/visits` → Visit Service `POST /api/visits/appointments` 持久化。
- Elder 字段不再依赖 mock 长者列表，改为从现有 `fetchAdminElderSummaries` （`/api/admin/elders`) 读取。
- 提交成功后跳回 `/elderly/visits?selected=<visitId>&entry=elderly-visits-new`，列表页基于真实查询刷新选中。
- 本期审核能力仍挂起（Visit Service 审批 API 未开放），页面继续明示“审批接入中”。

## Data Source

- Route type: client form with async submit
- Primary sinks:
  - `createAdminVisit(payload)` in `src/lib/services/admin-visit-services.ts`
  - Elder picker: `fetchAdminElderSummaries()`
- Transport chain: 前端 POST `/api/admin-visits/visits` → Next 代理 → Admin BFF `POST /api/admin/visits` → Visit Service `POST /api/visits/appointments`
- Payload contract: `VisitAppointmentCreateRequest { elderId, visitorName, relation, phone?, plannedAtUtc, visitType, notes? }`
- Response contract: `AdminVisitAppointmentResponse`

## UI States

- Loading state: 提交按钮禁用并显示“保存中…”。
- Empty state: 默认空表单；elder 下拉在加载完成前禁用。
- Error state: 字段校验失败本地提示；服务端失败显示顶部错误条，保留表单内容。
- Mobile impact: 字段按单列堆叠，提交按钮全宽。

## Health Signals

- Healthy: 提交后接口 200，跳转列表页可见新纪录；Visit 仓库新增一行。
- Failure: 提交 400/500；跳转后列表查询不到对应 visitId。
- Verification proxy: `npm run lint` + `npm run build` + 后端 `dotnet build`；手工点击“保存预约”验证 Visit Service 记录。

## Verification

- Minimum gate: `npm run lint`
- Behavior gate: `npm run lint` + `npm run build`
- Backend gate: `dotnet build`
- Manual path: 填表 → 保存 → 列表显示并选中。

## Rollback

- 还原 `src/app/elderly/visits/new/page.tsx` 回 mock 版本。
- 删除 `src/lib/services/admin-visit-services.ts` 中 `createAdminVisit`。
- 回滚后新建走回 mock `submitVisitAppointment`。

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
