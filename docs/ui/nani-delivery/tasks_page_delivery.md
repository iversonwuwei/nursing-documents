# 任务中心交付说明

## Scope

- Entry page: lib/app/modules/tasks/tasks_page.dart
- Affected users: 需要按优先级处理护理任务的护工
- Rollout stage: 第一批任务流治理补齐

## User Impact

- 原本只展示文案的筛选视图改为真实筛选，可按全部、P1、即将到点、待补录查看任务。
- 当前筛选同步约束关联长者列表，避免任务与对象信息脱节。
- 当筛选结果为空时显示明确空态，不再只剩空白页面。

## Data Source

- Controller: TasksController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation targets: healthEntry, careExecution, residentDetail

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前筛选下暂无任务；当前没有关联长者。
- Error state: 当前无远程请求，主要风险是筛选逻辑与按钮跳转不一致。
- Mobile impact: 筛选芯片、任务按钮、对象入口都具备稳定键，可覆盖小屏滚动验证。

## Health Signals

- Healthy signal: 筛选结果与关联长者同步变化，优先级和到点顺序稳定。
- Failure signal: 筛选后仍显示无关任务，或空态未出现。
- Stable selectors: task-filter-all, task-filter-p1, task-filter-due-soon, task-filter-needs-record, task-card-*, task-open-health-entry-*, task-open-care-execution-*, task-open-resident-*

## Verification

- flutter analyze
- flutter test
- Widget test covers task filtering and task empty states.

## Rollback

- Revert the filter state, empty-state widgets, and related tests.
- 页面将回退为静态任务列表，不改 mock 数据与目标路由。