# 报警处理交付说明

## Scope

- Entry page: lib/app/modules/alerts/alerts_page.dart
- Affected users: 负责接单、到场和复核的护工
- Rollout stage: 第一批报警流治理补齐

## User Impact

- 报警列表新增状态筛选，可按全部、待到场、处理中、已结案查看事件。
- 列表默认把未结案和更高等级事件排在前面，便于先处理高风险项。
- 当从护理执行页进入时，页面会在顶部展示“来自护理执行的异常跟进草稿”，把本次留证与备注带入人工判定入口。
- 当从健康录入页进入时，页面也会展示“来自健康录入的异常跟进草稿”，把高风险体征带入人工判定入口。
- 对于护理执行带入的异常草稿，页面必须提供两条明确动作：创建正式事件、仅保留观察，避免用户只看到草稿却不知道下一步怎么做。
- 选择“创建正式事件”后，页面会立即进入报警详情确认页，要求继续确认责任人与到场时限，而不是停留在模糊的升级成功提示上。
- 当前筛选无结果时显示明确空态，避免误解为页面加载失败。

## Data Source

- Controller: AlertsController
- Mock source: app/data/services/mock_nani_service.dart
- Navigation targets: alertDetail, aiAssist
- Additional upstream source: care execution follow-up draft, health entry follow-up draft

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前筛选下没有报警。
- Error state: 当前无远程请求，主要验证状态筛选、护理执行/健康录入上下文草稿、分流动作和 AI/详情入口是否稳定。
- Mobile impact: 报警筛选和动作按钮都有稳定键，方便小屏回归。

## Health Signals

- Healthy signal: 状态筛选后仅显示对应报警，且待到场/处理中优先于已结案；从护理执行或健康录入进入时能看见异常跟进草稿，并能明确选择创建正式事件或仅保留观察；升级后会继续进入责任链确认页。
- Failure signal: 筛选失效、上下文草稿丢失、分流动作不明确、升级后没有进入责任链确认、空态缺失或详情/AI 入口指向错误。
- Stable selectors: alert-care-execution-banner, alert-health-entry-banner, alert-draft-promote, alert-draft-observe, alert-filter-all, alert-filter-待到场, alert-filter-处理中, alert-filter-已结案, alert-card-*, alert-open-detail-*, alert-open-ai-*

## Verification

- flutter analyze
- flutter test
- Widget test covers alert filtering, care execution and health-entry context banners, explicit draft branching, escalation handoff into alert detail confirmation, and alert empty states.

## Rollback

- Revert the filter state, empty-state widget, and related tests.
- 页面回退为原始静态报警列表，不改变 mock 事件数据。