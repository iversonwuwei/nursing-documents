# 健康录入交付说明

## Scope

- Entry page: lib/app/modules/health_entry/health_entry_page.dart
- Affected users: 需要补录体征与复测数据的护工
- Rollout stage: 第三批录入页治理补齐

## User Impact

- 健康录入页的对象切换、体征输入、趋势入口和暂存按钮具备稳定锚点。
- 当没有可录入体征模板时，会显示明确空态而不是空白区域。
- 录入仍然只是暂存和生成解释，不自动升级报警。

## Data Source

- Controller: HealthEntryController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation target: health trend page

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有可录入体征项。
- Error state: 当前无远程提交链路，主要验证对象切换与暂存提示。
- Mobile impact: 对象芯片、输入项和暂存按钮均具备稳定键，便于小屏回归。

## Health Signals

- Healthy signal: 传入对象参数后默认命中正确对象，可切换对象、继续进入趋势页并执行暂存提示。
- Failure signal: 当前对象显示错误、空态缺失或暂存动作不可达。
- Stable selectors: health-entry-resident-*, health-entry-draft-*, health-entry-input-*, health-entry-current-*, health-entry-open-health-*, health-entry-save-button, health-entry-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers resident switching, save action, and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原录入表单展示，不改 mock 体征模板。