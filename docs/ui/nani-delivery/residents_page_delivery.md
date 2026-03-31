# 重点长者交付说明

## Scope

- Entry page: lib/app/modules/residents/residents_page.dart
- Affected users: 责任护工、接班护工
- Rollout stage: 第三批对象列表治理补齐

## User Impact

- 重点长者列表现在具备稳定锚点，可直接验证对象详情、健康录入和护理执行入口。
- 当本班次没有重点对象时，会显示明确空态而不是空白列表。
- 进入护理执行时改为传递真实 linkedTaskId，而不是展示文案。

## Data Source

- Controller: ResidentsController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation targets: residentDetail, healthEntry, careExecution

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有重点长者。
- Error state: 当前无远程请求，主要验证对象入口和参数传递正确性。
- Mobile impact: 列表卡片和三个动作入口均具备稳定键，便于小屏回归。

## Health Signals

- Healthy signal: 从重点长者页可稳定进入详情、健康录入和护理执行，且护理执行命中真实任务。
- Failure signal: 入口失效、空态缺失或护理执行参数错传。
- Stable selectors: resident-card-*, resident-open-detail-*, resident-open-health-entry-*, resident-open-care-execution-*, residents-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers residents entry chain and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原列表展示，不影响详情与执行路由结构。