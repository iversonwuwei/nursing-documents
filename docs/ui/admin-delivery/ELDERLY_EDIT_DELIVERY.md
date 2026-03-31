# Elderly Edit Delivery Unit

## Scope

- Entry route: src/app/elderly/[id]/edit/page.tsx
- Affected users: 档案维护、护理主管、运营校正用户
- Rollout stage: 第十一批长者工作流子路由治理说明

## User Impact

- 长者编辑页当前承接既有档案资料修订，并在保存后返回详情页。
- 当前交付单元先固定编辑入口职责和验证门禁，不修改字段布局或默认值策略。
- 保持编辑页与详情页之间的返回关系，以及当前默认对象演示行为不变。

## Data Source

- Route type: client edit form page
- Primary source: in-file elderlyData mock defaults
- Downstream dependency: next/navigation router push to elderly detail route

## UI States

- Loading state: 提交保存时通过 loading 按钮反馈处理中。
- Empty state: 当前依赖默认档案样例填充；后续接真实对象加载时未命中 id 应显式提示无对象。
- Error state: 当前保留 form-level error 占位，后续真实保存失败时应显式暴露。
- Mobile impact: 编辑表单三段卡片和底部提交区需验证窄屏滚动和按钮可达性。

## Health Signals

- Healthy signal: 编辑页稳定承接详情页进入、展示既有资料并在提交后返回详情。
- Failure signal: 编辑对象与详情对象错位、提交后跳转错误，或默认值与字段定义分叉。
- Verification proxy: lint 通过；行为改动时加 build 与档案编辑流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证从详情进入编辑页后可看到预填数据，提交后返回长者详情页

## Rollback

- Revert this delivery note and any future elderly edit route changes together.
- If regressions appear, fallback is the current local edit form with static defaults.