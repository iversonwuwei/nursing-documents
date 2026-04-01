# Staff New Delivery Unit

## Scope

- Entry route: src/app/staff/new/page.tsx
- Affected users: 人力运营、护理主管、值班管理协同用户
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- 员工列表页的“添加员工”按钮改为真实的新建入口，而不是停留在按钮层。
- 新员工先进入待入职状态，再由列表页执行人工确认入职。
- 首批只做 admin demo 闭环，不接真实人事系统或排班服务。

## Data Source

- Route type: client form page
- Primary sink: resource workflow shared store
- Downstream link: `/staff?selected=...&entry=staff-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 表单以最小字段集初始化，不依赖预加载。
- Error state: 姓名、角色、部门、联系电话等缺失时显式提示。
- Mobile impact: 单页表单卡在窄屏下退化为单列。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 新建员工 -> 列表提示卡 -> 确认入职 -> 详情页识别新员工

## Rollback

- Revert this note together with `/staff/new` and resource workflow staff changes.
- Fallback is to restore the previous static staff list-only behavior.