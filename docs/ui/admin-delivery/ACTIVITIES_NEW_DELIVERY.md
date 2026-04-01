# Activities New Delivery Unit

## Scope

- Entry route: src/app/activities/new/page.tsx
- Affected users: 活动运营、前台协同、护理主管
- Rollout stage: 第十五批运营新建闭环治理说明

## User Impact

- 新建活动页负责录入活动名称、分类、时间、地点、容量、负责人和活动说明。
- 提交后不会直接进入执行，而是先回流活动列表并进入待发布闭环。
- 首批目标是让活动新建行为留下可见状态，而不是停留在孤立表单。

## Data Source

- Route type: client page with local form state
- Primary data sink: shared operations workflow mock store with localStorage persistence
- Downstream link: activities list route with selected and entry query params

## UI States

- Loading state: 提交按钮进入 loading，避免重复提交同一条活动初稿。
- Empty state: 不适用；表单默认以最小可提交字段集初始化。
- Error state: 必填项、时长和容量校验失败时在表单顶部显式提示。
- Mobile impact: 表单网格、闭环说明卡和提交 CTA 在窄屏下需保持单列可读。

## Health Signals

- Healthy signal: 活动初稿提交后能回流列表并显示待发布提示卡。
- Failure signal: 表单提交后未产生新活动，或新活动未进入待发布状态。
- Verification proxy: docs build、lint、build 通过；人工验证提交流程与回流状态。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证活动录入、校验错误、提交回流、待发布状态和后续发布动作

## Rollback

- Revert this delivery note together with activities new route and shared operations workflow changes.
- If regressions appear, fallback is to remove activities new route and restore the previous static list-only behavior.