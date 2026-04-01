# Equipment New Delivery Unit

## Scope

- Entry route: src/app/equipment/new/page.tsx
- Affected users: 设备管理员、后勤、值班护理协同用户
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- 设备列表页的“添加设备”按钮改为真实的新建设备入口。
- 新设备提交后先进入待验收状态，再由列表页或详情页完成验收入册。
- 不接真实资产系统；当前只保证 admin 前端演示闭环可追踪。

## Data Source

- Route type: client form page
- Primary sink: resource workflow shared store
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

- Revert this note together with `/equipment/new` and resource workflow equipment changes.
- Fallback is to restore the previous static equipment list-only behavior.