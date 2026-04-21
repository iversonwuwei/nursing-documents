# Elderly Vitals New Delivery Unit

## Scope

- Entry route: src/app/elderly/vitals/new/page.tsx
- Affected users: 当班护士、护理主管
- Rollout stage: 体征录入从 mock `care-service-workflow` 切换到真实 Health Service 观察值写通道

## Change Summary

- 新建体征改为 `createAdminVitalObservation()` → Admin BFF `POST /api/admin/vitals` → Health Service `POST /api/health/vitals`。
- Elder 下拉改为从 `fetchAdminElderList` 读真实老人；取消 mock `elderlyList` 依赖。
- 提交成功后跳转 `/elderly/vitals?selected=<observationId>&entry=elderly-vitals-new`；失败保留表单并展示错误。

## Data Source

- Route type: client form with async submit
- Primary sinks:
  - `createAdminVitalObservation(payload)` in `src/lib/services/admin-vitals-services.ts`
  - Elder picker: `fetchAdminElderList()`
- Transport chain: 前端 POST `/api/admin-vitals/vitals` → Next 代理 → Admin BFF `POST /api/admin/vitals` → Health Service `POST /api/health/vitals`
- Payload contract: `VitalObservationCreateRequest { elderId, bloodPressure, heartRate, temperature, bloodSugar, oxygen, recordedBy, recordedAtUtc }`
- Response contract: `VitalObservationResponse`

## UI States

- Loading state: 提交按钮禁用并显示“保存中…”。
- Empty state: 初次进入表单为空；elder 下拉在加载完成前禁用。
- Error state: 字段校验失败提示；服务端失败在顶部错误条展示，保留已填内容。
- Mobile impact: 字段按单列堆叠。

## Health Signals

- Healthy: 提交 200，跳转列表后可见刚插入的记录。
- Failure: 400/500；列表页找不到 observationId。
- Verification proxy: `npm run lint` + `npm run build`；`dotnet build` + migration + 前端提交写入落库。

## Verification

- Minimum gate: `npm run lint`
- Behavior gate: `npm run lint` + `npm run build`
- Backend gate: `dotnet build` + EF migration
- Manual: 提交 → 列表可见 → 新纪录时间与 recordedBy 一致。

## Rollback

- 还原 `src/app/elderly/vitals/new/page.tsx`。
- 删除 `createAdminVitalObservation` 导出。
- 回滚后录入走回 mock `addVitalsEntry`。
