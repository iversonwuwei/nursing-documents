# 长者详情交付说明

## Scope

- Entry page: lib/app/modules/elder_detail/elder_detail_page.dart
- Affected users: 责任护工、主管复核用户
- Rollout stage: 第三批对象详情治理补齐

## User Impact

- 长者详情页的重点观察项、信号卡片和下一步动作具备稳定锚点，可验证对象上下文是否贯穿健康、执行、交接和 AI。
- 保持原有风险、观察重点和家属关注内容不变。
- 详情页仍然只负责串联动作入口，不自动改变业务状态。

## Data Source

- Controller: ResidentDetailController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation targets: health, healthEntry, careExecution, handover, aiAssist

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前通过参数缺省回退到首位对象，不单独新增无对象态。
- Error state: 主要验证对象上下文与动作入口一致性。
- Mobile impact: 多个动作卡片可通过稳定键回归，不依赖滚动中的文案定位。

## Health Signals

- Healthy signal: 同一对象在详情页、健康录入、护理执行和 AI 页面之间保持上下文一致。
- Failure signal: 动作入口跳到错误对象或错误任务，或重点卡片不可见。
- Stable selectors: resident-detail-hero-*, resident-watch-*-*, resident-signal-*, resident-focus-task-*, resident-open-health-*, resident-open-health-entry-*, resident-open-care-execution-*, resident-open-handover-*, resident-open-ai-*

## Verification

- flutter analyze
- flutter test
- Widget test covers resident detail actions into health entry and AI assistant.

## Rollback

- Revert the stable keys and route-chain tests.
- 页面回退为原详情布局，不影响对象 mock 数据。