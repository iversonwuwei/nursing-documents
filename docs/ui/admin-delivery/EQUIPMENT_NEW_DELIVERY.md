# Equipment New Delivery Unit

## Scope

- Entry route: src/app/equipment/new/page.tsx
- Affected users: 设备管理员、后勤、值班护理协同用户
- Rollout stage: equipment family live read-write 收口中的建设备入口

## User Impact

- 设备列表页的“添加设备”按钮改为真实的新建设备入口。
- 新设备提交后先进入待验收状态，再由列表页或详情页完成验收入册。
- 当前不接外部资产系统，但已改为真实持久化 operations 数据，而不是前端演示闭环。

## Data Source

- Route type: client form page
- Primary sink: Admin equipment create API
- Downstream link: `/equipment?selected=...&entry=equipment-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认以最小必填字段初始化。
- Error state: 设备名称、分类、型号、位置、采购日期等校验失败时显式提示。
- Mobile impact: 表单需保持单列可读，避免设备字段在窄屏错位。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 新建设备 -> 列表回流 -> 待验收提示 -> 验收后状态切换

## Rollback

- Revert this note together with `/equipment/new`、Next proxy、Admin BFF 和 operations equipment create endpoint。
- Fallback is to restore the previous local equipment draft behavior.
