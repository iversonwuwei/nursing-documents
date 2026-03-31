# 健康趋势交付说明

## Scope

- Entry page: lib/app/modules/health/health_page.dart
- Affected users: 护工、责任护士、需要结合趋势卡片判断下一步动作的协同用户
- Rollout stage: 第四批消息与个人工作台治理补齐

## User Impact

- 健康趋势页现在为长者切换、趋势卡片、观察重点和后续动作补充稳定锚点。
- 当趋势卡片或观察重点缺失时会显示显式空态，避免误判为渲染失败。
- 保持 AI、长者详情和健康录入仍由人工主动进入，不自动触发升级动作。

## Data Source

- Controller: HealthController
- Mock source: app/data/services/mock_nani_service.dart
- Upstream routes: resident detail, health entry, AI 上下文跳转

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有趋势指标，或当前没有观察重点。
- Error state: 当前无远程拉取，主要验证对象切换、趋势卡片、观察重点与动作入口稳定可见。
- Mobile impact: 多张趋势卡片与动作卡在窄屏下仍需保持对象切换和 CTA 可点。

## Health Signals

- Healthy signal: 长者切换后 hero、趋势卡片和动作入口与对象上下文一致。
- Failure signal: 对象切换后内容未刷新、空态缺失，或 AI/录入/详情入口上下文错位。
- Stable selectors: health-resident-*, health-hero-*, health-metric-*, health-watch-*, health-open-entry-*, health-open-resident-*, health-open-ai-*, health-metrics-empty-state, health-watch-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers health context switching, AI route entry, and empty states.

## Rollback

- Revert the stable keys, empty-state widgets, and related tests.
- 页面回退为原趋势展示，不改 mock 健康视图数据。