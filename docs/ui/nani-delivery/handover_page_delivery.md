# 交接班交付说明

## Scope

- Entry page: lib/app/modules/handover/handover_page.dart
- Affected users: 责任护工、接班护工、主管复核用户
- Rollout stage: 第二批交接流治理补齐

## User Impact

- 交接摘要、交接项详情入口和保存草稿动作现在具备稳定锚点，方便覆盖责任链路回归。
- 当本班没有待交接项时，会显示明确空态，不再只剩空白区域。
- 保持现有 mock 交接数据和“保存草稿”行为不变。

## Data Source

- Controller: HandoverController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation target: handoff detail route

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有待交接项。
- Error state: 当前无远程提交链路，主要验证详情入口与保存提示是否稳定。
- Mobile impact: 交接卡片和保存草稿按钮具备稳定键，便于小屏回归。

## Health Signals

- Healthy signal: 可从交接列表进入交接详情，并能保存草稿提示人工确认边界。
- Failure signal: 交接详情入口失效、空态缺失或草稿按钮不可达。
- Stable selectors: handover-summary-card, handover-card-*, handover-open-detail-*, handover-save-draft, handover-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers handover entry chain, draft save prompt, and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原交接列表展示，不影响 handoff 详情路由。