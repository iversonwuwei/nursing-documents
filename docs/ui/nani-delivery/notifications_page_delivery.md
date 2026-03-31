# 消息中心交付说明

## Scope

- Entry page: lib/app/modules/notifications/notifications_page.dart
- Affected users: 护工、值班主管、需要核对交接与任务提醒的协同用户
- Rollout stage: 第四批消息与个人工作台治理补齐

## User Impact

- 消息中心现在提供稳定摘要卡片和消息卡片锚点，便于从首页消息入口回归。
- 当当前班次没有待处理消息时显示明确空态，而不是只留下空白列表。
- 保持消息中心只读，不在此页直接完成任务或报警处理。

## Data Source

- Controller: NotificationsController
- Mock source: app/data/services/mock_nani_service.dart
- Upstream route: home 消息入口

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有待处理消息。
- Error state: 当前无远程拉取，主要验证摘要卡片、消息卡片和空态稳定可见。
- Mobile impact: 列表和摘要卡片在小屏下仍需保持标题、分类和已读状态清晰可见。

## Health Signals

- Healthy signal: 首页进入消息中心后摘要卡片、消息列表或空态稳定可见。
- Failure signal: 列表为空但无空态，或消息卡片锚点不稳定导致回归路径失效。
- Stable selectors: notifications-summary-card, notification-card-*, notifications-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers home -> notifications route and notifications empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原只读消息列表，不改 mock 数据结构。