# Supplies New Delivery Unit

## Scope

- Entry route: src/app/supplies/new/page.tsx
- Affected users: 采购、仓储、后勤主管
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- “采购入库”按钮改为真实入库入口，支持补货现有物资或新增物资品类。
- 新采购对象先进入待上架或已入库状态，再回流物资列表更新库存口径。
- 首批不接采购审批系统，只保证物资台账在 admin 内部闭环。

## Data Source

- Route type: client form page
- Primary sink: resource workflow shared store
- Downstream link: `/supplies?selected=...&entry=supplies-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 允许新物资录入和现有物资补货两种模式。
- Error state: 名称、分类、数量、最低库存、供应商等缺失时显式提示。
- Mobile impact: 采购字段较多，需要维持单列表单和明确分组。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 新增或补货物资 -> 列表库存变化 -> 待上架提示 -> 详情页历史记录

## Rollback

- Revert this note together with `/supplies/new` and resource workflow supplies changes.
- Fallback is to restore the previous static supplies list-only behavior.