# 个人页交付说明

## Scope

- Entry page: lib/app/modules/profile/profile_page.dart
- Affected users: 当前班次护工、主管、需要在班前确认排班和交接重点的协同用户
- Rollout stage: 第四批消息与个人工作台治理补齐

## User Impact

- 个人页现在对班次摘要、排班入口、交接班入口和退出登录提供稳定锚点。
- 当没有未来排班或没有交接班重点时显示明确空态，而不是留空。
- 保持退出登录仍然是显式操作，不自动清除其他业务数据。

## Data Source

- Controller: ProfileController
- Mock source: app/data/services/mock_nani_service.dart
- Downstream routes: schedule, handover, login

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有未来排班，或当前没有交接班重点。
- Error state: 当前无远程拉取，主要验证摘要、预览卡、空态和退出入口稳定可见。
- Mobile impact: 排班预览、交接重点和退出按钮在底部导航场景下仍需可见可点。

## Health Signals

- Healthy signal: 从底部导航进入个人页后，排班、交接班和退出登录链路稳定可达。
- Failure signal: 入口卡片不可见、空态缺失，或退出登录后未回到登录页。
- Stable selectors: root-nav-profile, profile-summary-card, profile-open-schedule, profile-open-handover, profile-schedule-*, profile-handover-summary, profile-handover-*, profile-schedule-empty-state, profile-handover-empty-state, profile-logout-button

## Verification

- flutter analyze
- flutter test
- Widget test covers root -> profile navigation, schedule/handover route entry, logout, and empty states.

## Rollback

- Revert the stable keys, empty-state widgets, and related tests.
- 页面回退为原班次摘要、排班预览与交接预览展示，不改登录服务与 mock 数据。