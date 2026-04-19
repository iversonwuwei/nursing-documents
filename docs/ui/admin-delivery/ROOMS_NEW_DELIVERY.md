# Rooms New Delivery Unit

## Scope

- Entry route: src/app/rooms/new/page.tsx
- Affected users: 前台入住协调、床位运营、保洁主管与机构协同用户
- Rollout stage: rooms live vertical slice

## User Impact

- 新增房间页现在承接房间编号、机构归属、房型、床位和设施录入，并在提交后进入待启用闭环。
- 房间列表页可对新建房间执行人工启用，再决定是否加入可入住资源池。
- 主区只保留主数据录入和提交动作，完整流程说明与帮助入口后置到信息轨。
- 当前机构覆盖范围、启用边界与帮助说明统一后置到信息轨，不再与表单字段混排。
- 保持排房仍由人工决策，启用动作只改变房间资源可见性，不自动分配入住对象。

## Data Source

- Route type: client form page
- Primary source: Next `/api/rooms` -> Admin BFF `/api/admin/rooms`
- Downstream dependency: create live room record，并跳转 `/rooms?selected=...&entry=rooms-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认空表单；若没有已启用机构，应提示先完成机构建档与启用。
- Error state: 编号、名称、机构、楼层或床位数非法时，显式展示表单级错误。
- Mobile impact: 单页表单卡片布局，需保证机构选择和提交区在窄屏可达。

## Health Signals

- Healthy signal: 新增房间提交后进入待启用闭环，且房间列表和详情页都能命中同一对象。
- Failure signal: 房间编号冲突未拦截、提交后对象丢失，或新建页未命中真实机构选择器仍允许自由文本机构提交。
- Verification proxy: lint 通过；行为改动时加 build 与房间新建人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证房间录入、跳转列表、待启用提示、帮助入口和启用动作

## Rollback

- Revert this delivery note and any future rooms new route changes together.
- If regressions appear, rollback the live new-room flow together; do not keep partial live create with mock list hydration.
