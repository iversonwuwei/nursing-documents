# Organizations New Delivery Unit

## Scope

- Entry route: src/app/organizations/new/page.tsx
- Affected users: 机构运营、集团管理、院长与筹备开业协同用户
- Rollout stage: 主数据新建闭环第一批扩展路由

## User Impact

- 新增机构页现在承接机构主数据录入，并在提交后进入待启用闭环，而不是直接把机构计入经营台账。
- 运营人员可在机构列表页对新建机构执行人工启用，再决定是否纳入集团资源视图。
- 保持新增机构仍是本地 mock 闭环，不直接触发真实开业流程或配置下发。

## Data Source

- Route type: client form page
- Primary source: master-data-workflow shared store
- Downstream dependency: addOrganizationDraft 写入 shared store，并跳转 `/organizations?selected=...&entry=organizations-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 当前依赖默认空表单；后续接真实 API 时补字段级提示。
- Error state: 名称、地址、电话、床位数、负责人或负责人电话缺失时，显式展示表单级错误。
- Mobile impact: 单页表单卡片布局，需保证底部提交区在窄屏可达。

## Health Signals

- Healthy signal: 新增机构提交后进入待启用闭环，且可在机构列表页看到同一对象。
- Failure signal: 提交后对象丢失、机构列表无法启用，或新建机构被直接计入经营口径。
- Verification proxy: lint 通过；行为改动时加 build 与机构新建人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证机构录入、跳转列表、待启用提示和启用动作

## Rollback

- Revert this delivery note and any future organizations new route changes together.
- If regressions appear, fallback is移除新建页入口并回退到只读机构列表。
