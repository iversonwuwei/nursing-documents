# 报警处理交付说明

## Scope

- Entry page: lib/app/modules/alerts/alerts_page.dart
- Affected users: 负责接单、到场和复核的护工
- Rollout stage: 第一批报警流治理补齐

## User Impact

- 报警列表新增状态筛选，可按全部、待到场、处理中、已结案查看事件。
- 列表默认把未结案和更高等级事件排在前面，便于先处理高风险项。
- 当前筛选无结果时显示明确空态，避免误解为页面加载失败。

## Data Source

- Controller: AlertsController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation targets: alertDetail, aiAssist

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前筛选下没有报警。
- Error state: 当前无远程请求，主要验证状态筛选和 AI/详情入口是否稳定。
- Mobile impact: 报警筛选和动作按钮都有稳定键，方便小屏回归。

## Health Signals

- Healthy signal: 状态筛选后仅显示对应报警，且待到场/处理中优先于已结案。
- Failure signal: 筛选失效、空态缺失或详情/AI 入口指向错误。
- Stable selectors: alert-filter-all, alert-filter-待到场, alert-filter-处理中, alert-filter-已结案, alert-card-*, alert-open-detail-*, alert-open-ai-*

## Verification

- flutter analyze
- flutter test
- Widget test covers alert filtering and alert empty states.

## Rollback

- Revert the filter state, empty-state widget, and related tests.
- 页面回退为原始静态报警列表，不改变 mock 事件数据。