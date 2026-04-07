# 服务打卡交付说明

## Scope

- Entry page: lib/app/modules/care_checkin/care_checkin_page.dart
- Affected users: 一线护工、接班复核用户
- Rollout stage: 第七批打卡闭环治理补齐

## User Impact

- 护工从首页、任务中心和长者详情进入任务时，先完成服务打卡，而不是直接跳到护理执行。
- 打卡页会显式要求确认任务对象、打卡方式、服务位置和异常说明。
- 打卡成功后把摘要带到护理执行页，后续交接班或报警链路也能看到到场上下文。

## Data Source

- Controller: CareCheckinController
- Mock source: app/data/services/mock_nani_service.dart
- Related models: CareTask, CareClockInDraft
- Downstream targets: careExecution, alerts

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 若任务不存在，显示明确空态和返回入口。
- Error state: 未选择打卡方式、未确认到场或异常说明缺失时阻断提交。
- Mobile impact: 方式选择、确认项、异常说明和 CTA 在窄屏下均可直接操作。

## Health Signals

- Healthy signal: 进入页面后能看到正确任务；完成打卡后能进入护理执行并显示打卡摘要。
- Failure signal: 错误任务进入、未打卡也误认为已完成、或护理执行页看不到打卡上下文。
- Stable selectors: care-checkin-header-*, care-checkin-method-*, care-checkin-location-input, care-checkin-confirm-arrival, care-checkin-exception-input, care-checkin-submit, care-checkin-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers 打卡方式选择、阻断校验、成功进入护理执行和异常备注分支。

## Rollback

- Revert the new page, route registration, navigation entry changes, and related tests.
- 入口将回退为直接进入护理执行页的旧路径。