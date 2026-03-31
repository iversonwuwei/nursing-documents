# 排班页交付说明

## Scope

- Entry page: lib/app/modules/schedule/schedule_page.dart
- Affected users: 护工、班次协调与主管确认用户
- Rollout stage: 第二批排班页治理补齐

## User Impact

- 排班卡片具备稳定锚点，便于回归未来班次是否可见。
- 当没有未来排班时显示明确空态，避免误以为页面异常。
- 保持现有 mock 排班信息和主管确认边界文案不变。

## Data Source

- Controller: ScheduleController
- Mock source: app/data/services/mock_nani_service.dart
- Adjacent entry: 首页快捷动作 schedule

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有未来排班。
- Error state: 当前无远程排班接口，主要验证卡片稳定展示与空态。
- Mobile impact: 排班卡片在小屏下仍能通过稳定键定位。

## Health Signals

- Healthy signal: 首页可以进入排班页，未来班次列表可见。
- Failure signal: 首页入口失效、卡片缺失或空态未展示。
- Stable selectors: home-quick-action-schedule, schedule-card-*, schedule-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers schedule entry chain and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面将回退为原排班列表显示，不改变 mock 数据。