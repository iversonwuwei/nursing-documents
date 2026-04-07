# 首页交付说明

## Scope

- Entry page: lib/app/modules/home/home_page.dart
- Affected users: 已登录护工、责任护工
- Rollout stage: 第一批首页总览治理补齐

## User Impact

- 首页消息入口、快捷动作和重点对象入口具备稳定锚点，便于验证关键跳转链路。
- 保持原有 mock 首页信息不变，不调整首页数据结构。
- 快捷动作继续只做路由跳转，不在首页直接执行护理动作。

## Data Source

- Controller: HomeController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation targets: notifications, residents, health, handover, schedule, careCheckin, careExecution

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 本批不改数据契约，保持现有首页卡片展示。
- Error state: 当前无远程请求，主要验证导航链路稳定性。
- Mobile impact: 首页快捷动作与通知按钮可被稳定定位并在滚动后触发。

## Health Signals

- Healthy signal: 登录后可从首页进入消息中心、快捷动作目标页，并通过底部导航切换主标签。
- Failure signal: 关键入口无法命中目标页或 IndexedStack 标签切换失效。
- Stable selectors: home-open-notifications, home-quick-action-residents, home-quick-action-health, home-quick-action-handover, home-quick-action-care-checkin, home-quick-action-schedule, root-nav-home, root-nav-tasks, root-nav-alerts

## Verification

- flutter analyze
- flutter test
- Widget test covers notifications entry, health quick action, and bottom navigation switching.

## Rollback

- Revert the stable keys and route-chain tests.
- 首页布局和 mock 数据将回退到原状态，不影响路由定义。