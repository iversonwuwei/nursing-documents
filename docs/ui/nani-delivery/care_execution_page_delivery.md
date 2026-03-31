# 护理执行交付说明

## Scope

- Entry page: lib/app/modules/care_execution/care_execution_page.dart
- Affected users: 一线护工、接班复核用户
- Rollout stage: 第三批执行页治理补齐

## User Impact

- 护理执行页的任务头、步骤勾选、备注框和提交通道具备稳定锚点。
- 当没有步骤模板时，会显示明确空态而不是只有备注区。
- 提交结果仍然只给提示，不直接自动联动交接班或报警链路。

## Data Source

- Controller: CareExecutionController
- Mock source: app/data/services/mock_nani_service.dart
- Upstream targets: tasks page, residents page, resident detail page

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有待执行步骤。
- Error state: 当前无远程提交链路，主要验证步骤勾选和提交提示。
- Mobile impact: 步骤卡片、备注框和提交按钮具备稳定键，可在小屏回归。

## Health Signals

- Healthy signal: 执行页能加载正确任务，步骤可勾选，提交后提示同步责任链路确认。
- Failure signal: 任务头错位、步骤不可交互、空态缺失或提交按钮不可达。
- Stable selectors: care-task-header-*, care-step-*, care-note-card, care-note-input, care-submit-button, care-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers checklist toggling, submit action, and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原执行表单，不改变任务 mock 数据。