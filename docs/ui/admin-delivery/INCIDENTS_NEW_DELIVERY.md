# Incidents New Delivery Unit

## Scope

- Entry route: src/app/incidents/new/page.tsx
- Affected users: 质控、运营、值班管理与事故复盘用户
- Rollout stage: incidents family live read-write 收口中的创建入口

## User Impact

- 新增事件报告页负责录入事故初报，包括标题、级别、地点、报告人、时间、描述、附件和下一步动作。
- 提交后不会直接视为处理中，而是先回流事故列表并进入待分派闭环。
- 首批目标是让事故新建行为具备可见状态和后续推进入口，而不是停留在静态表单。

## Data Source

- Route type: client page with local form state
- Primary data sink: Admin incidents create API
- Downstream link: incidents list route with selected and entry query params

## UI States

- Loading state: 提交按钮进入 loading，避免重复提交同一条事故初报。
- Empty state: 不适用；表单默认以最小可提交字段集初始化。
- Error state: 必填项或严重事故缺少下一步动作时在表单顶部显式提示；服务端写入失败时保留显式错误信息。
- Mobile impact: 表单网格、闭环说明卡和提交 CTA 在窄屏下需保持单列可读。

## Health Signals

- Healthy signal: 事故初报提交后能回流列表并显示待分派提示卡。
- Failure signal: 新建事故未进入待分派状态，或选中提示卡未指向当前事故。
- Verification proxy: docs build、lint、build 通过；人工验证提交流程与回流状态。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证事故录入、校验错误、提交回流、待分派状态和后续开始处置动作

## Rollback

- Revert this delivery note together with incidents create route、Next proxy、Admin BFF 和 operations create endpoint。
- If regressions appear, fallback is to restore the previous local incident creation path.
