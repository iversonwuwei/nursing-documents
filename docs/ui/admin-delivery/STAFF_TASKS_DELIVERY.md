# Staff Tasks Delivery Unit

## Scope

- Entry route: src/app/staff/tasks/page.tsx
- Affected users: 班次主管、护理执行协调、入住协同用户
- Rollout stage: 第六批设备与组织扩展路由治理说明

## User Impact

- 员工任务页承担入住评估派生任务、筛选、任务备注保存和状态推进的统一执行面板。
- 当前交付单元先固定说明与验证门禁，不改现有任务筛选、任务动作和 AI 优先级建议行为。
- 保持任务状态推进与备注保存仍停留在本地 mock store，不引入真实工单契约变更。

## Data Source

- Route type: client page with local filters plus useSyncExternalStore subscription
- Primary sources: admission-workflow store, derived staff task items, and AI task recommendation helpers
- Downstream links: AI assistant context links and local task mutation helpers

## UI States

- Loading state: 当前依赖本地共享 store，同步渲染；若未来接真实任务系统需补筛选和保存反馈。
- Empty state: 搜索或筛选后无任务时应保持列表级空态。
- Error state: 派生任务、统计卡和任务动作状态不一致时需局部暴露。
- Mobile impact: 筛选条、表格和备注输入并存，后续改动需验证窄屏下编辑与动作按钮可达性。

## Health Signals

- Healthy signal: 入住评估派生任务、统计卡、筛选结果和任务状态推进围绕同一任务数据集保持一致。
- Failure signal: 任务来源状态和执行状态错位，或 AI 优先级建议直接改写执行结果。
- Verification proxy: lint 通过；行为改动时加 build 与员工任务人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、优先级/状态筛选、备注保存、状态推进和 AI 链路

## Rollback

- Revert this delivery note and any future staff tasks route changes together.
- If regressions appear, fallback is the previous local derived-task view and task mutation behavior.
