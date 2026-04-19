# Elderly Health New Delivery Unit

## Scope

- Entry route: src/app/elderly/health/new/page.tsx
- Affected users: 护理主管、护士、健康档案维护人员
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- 健康档案页的“新建档案”按钮改为真实建档入口。
- 新档案提交后直接写入 Health Service 真实库，并回流到健康档案列表页。
- 首批承接 admin 本地联调真实样本，不再把 care service workflow 作为健康建档事实源。

## Data Source

- Route type: client form page
- Primary sink: Admin BFF `/api/admin/health/archives` -> Health Service `/api/health/archives`
- Downstream link: `/elderly/health?selected=...&entry=elderly-health-new`
- Dependent systems: Elder Service 下拉对象来源、Health Service 写入接口、PostgreSQL seed 数据

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认空表单，若 Elder Service 暂无可选对象则显式提示当前无法建档。
- Error state: 长者列表读取失败或健康建档写入失败时显式提示。
- Mobile impact: 健康指标字段需在窄屏下保持纵向分组。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 新建健康档案 -> Health Service 写入成功 -> 列表回流并定位到刚创建对象

## Rollback

- Revert this note together with `/elderly/health/new` and care service workflow health archive changes.
- Fallback is to restore the previous static health archive list behavior，并撤回 Health Service 建档写接口接入。
