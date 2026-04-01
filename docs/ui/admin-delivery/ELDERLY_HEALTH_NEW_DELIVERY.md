# Elderly Health New Delivery Unit

## Scope

- Entry route: src/app/elderly/health/new/page.tsx
- Affected users: 护理主管、护士、健康档案维护人员
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- 健康档案页的“新建档案”按钮改为真实建档入口。
- 新档案提交后先进入待建档状态，再由列表页确认完成建档。
- 首批仅承接 admin demo 数据，不替代正式电子健康档案系统。

## Data Source

- Route type: client form page
- Primary sink: care service workflow shared store
- Downstream link: `/elderly/health?selected=...&entry=elderly-health-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认空表单，允许最小健康基线录入。
- Error state: 姓名、房间、核心指标缺失时显式提示。
- Mobile impact: 健康指标字段需在窄屏下保持纵向分组。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 新建健康档案 -> 列表回流 -> 待建档提示 -> 建档确认

## Rollback

- Revert this note together with `/elderly/health/new` and care service workflow health archive changes.
- Fallback is to restore the previous static health archive list behavior.