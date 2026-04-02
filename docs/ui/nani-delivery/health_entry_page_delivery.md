# 健康录入交付说明

## Scope

- Entry page: lib/app/modules/health_entry/health_entry_page.dart
- Affected users: 需要补录体征与复测数据的护工
- Rollout stage: 第三批录入页治理补齐

## User Impact

- 健康录入页的对象切换、图片识别入口、体征输入、趋势入口和暂存按钮具备稳定锚点。
- 护工可以先上传或拍摄读数图片，再触发 OCR/识别建议，并一键回填到体征表单，减少手工抄录。
- 当识别或回填结果命中高风险阈值时，页面会显式提供“查看 AI 解释”和“送入异常跟进”两条后续动作。
- 高风险阈值已从页面内联规则抽到共享配置，后续接真实接口或做机构差异化配置时只需要调整统一规则文件。
- 当没有可录入体征模板时，会显示明确空态而不是空白区域。
- 录入仍然只是暂存和生成解释，不自动升级报警。

## Data Source

- Controller: HealthEntryController
- Mock source: app/data/services/mock_nani_service.dart
- Recognition helper: mock health image recognition helper
- Shared thresholds config: lib/app/data/config/health_risk_rules.dart
- Navigation target: health trend page
- Downstream follow-up: ai assistant and alerts page via health-entry follow-up draft

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有可录入体征项。
- Error state: 当前无远程提交链路，主要验证对象切换、识别输入缺失时的阻断、识别结果回填、高风险后续动作和暂存提示。
- Mobile impact: 对象芯片、识别槽位、识别结果卡、输入项和暂存按钮均具备稳定键，便于小屏回归。

## Execution Stability

- 图片识别只是辅助录入，不能绕过人工复核；识别结果必须显式点击回填后才写入表单。
- 切换录入对象后，上一位长者的识别草稿不应继续污染当前对象。
- 若没有上传图片但录入了 OCR 摘要，也允许生成识别建议，便于低配场景补录。
- 高风险识别结果只能触发 AI 解释或异常跟进草稿，不应在健康录入页直接自动创建正式事件。
- 高风险阈值由共享配置统一管理，页面只消费评估结果，不在页面内重复维护阈值常量。

## Health Signals

- Healthy signal: 传入对象参数后默认命中正确对象，可切换对象、上传或拍摄读数图、生成识别建议并回填；若出现高风险值，可继续进入 AI 解释或异常跟进链路。
- Failure signal: 当前对象显示错误、识别入口不明确、识别后不能回填、高风险后续动作缺失、切换对象后草稿串值、空态缺失或暂存动作不可达。
- Stable selectors: health-entry-resident-*, health-scan-summary, health-scan-slot-*, health-scan-capture-*, health-scan-gallery-*, health-scan-ocr-input, health-scan-recognize, health-scan-result, health-scan-apply, health-followup-summary, health-followup-ai, health-followup-alert, health-entry-draft-*, health-entry-input-*, health-entry-current-*, health-entry-open-health-*, health-entry-save-button, health-entry-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers resident switching, image-recognition autofill, high-risk follow-up actions, save action, and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原录入表单展示，不改 mock 体征模板。