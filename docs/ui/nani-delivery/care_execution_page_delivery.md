# 护理执行交付说明

## Scope

- Entry page: lib/app/modules/care_execution/care_execution_page.dart
- Affected users: 一线护工、接班复核用户、护理组长
- Rollout stage: 第六批执行留证重设计

## User Impact

- 护理执行页从“只有备注框”升级为“步骤确认 -> 留证上传 -> 执行结果”的顺序化执行流。
- 需要图片留证的任务会明确展示留证要求、必传张数、每张图拍摄对象，以及“现场拍照 / 相册补传”的操作入口。
- 如果现场不适合拍照，页面会显式提供留证异常说明入口，而不是让护工自行猜测是否可以跳过。
- 提交前会校验步骤是否完成、必传留证是否齐全、执行备注是否已补充，避免只提交“已完成”而没有证据链。
- 提交成功后会生成结构化执行摘要，并显式提供“送入交接班草稿”与“升级为异常跟进”两个后续动作。
- 不应变化的流程: 任务仍从任务中心、重点长者和长者详情进入；AI 仍只提供建议，不自动创建正式事件或替代人工交接确认。

## Data Source

- Controller: CareExecutionController
- Mock source: app/data/services/mock_nani_service.dart
- 关联模型: CareTask, CareEvidenceRequirement, CareClockInDraft
- Upstream targets: care check-in page, tasks page, residents page, resident detail page
- 当前阶段为本地 mock 交互；后续真实接口需要沿用相同的留证槽位、执行摘要对象和下游联动结构接入上传 API 与 workflow API。

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有待执行步骤时，仍显示明确空态；若任务本身无需图片留证，也需显示“本任务无需上传图片”的说明卡，而不是直接省略模块。
- Error state: 当步骤未完成、必传图片缺失或执行备注为空时，页面要显示明确阻断提示，不能让失败只体现在 Snackbar 一闪而过；下游联动卡只应在成功提交后出现。
- Weak-network fallback: 当前无真实上传链路，先以本地占位留证卡承接；接入真实上传失败时应保留已拍证据缩略卡和重试入口。
- Mobile impact: 留证槽位、按钮、异常说明和提交按钮都要在窄屏下可达，不依赖 hover 或隐藏操作。

## Execution Stability

- 任务上下文必须跨返回保持，至少保留当前任务头、步骤勾选和本地留证占位状态。
- 执行页内的留证要求必须从任务模型读取，而不是写死在页面里，避免不同任务进入后提示错位。
- 返回任务页或长者页后，再次进入相同任务时至少应能看到一致的留证要求结构。

## Health Signals

- Healthy signal: 执行页能加载正确任务；需留证的任务能明确展示必传槽位；护工可完成拍照或补传占位；缺失证据时提交被阻断；补齐后可完成提交，并继续送入交接班或异常跟进链路。
- Failure signal: 用户看不出本任务是否需要留证、缺图仍能提交、没有异常说明入口、提交后无法继续进入交接班或异常跟进、或者页面只留下无结构备注。
- Stable selectors: care-task-header-*, care-step-*, care-evidence-summary, care-evidence-slot-*, care-evidence-capture-*, care-evidence-gallery-*, care-evidence-exception-input, care-note-card, care-note-input, care-submit-blocker, care-submit-button, care-followup-summary, care-followup-handover, care-followup-alert, care-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers checklist toggling,留证槽位展示、缺少留证时的阻断、补齐留证后的提交、交接班/报警处理联动，以及空态。

## Rollback

- Revert the evidence-requirement model, care execution page redesign, and related widget tests.
- 页面可回退为原有“步骤 + 备注 + 提示提交”的轻量原型，不影响任务入口与 mock 路由。