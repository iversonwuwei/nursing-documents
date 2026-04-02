# 报警详情交付说明

## Scope

- Entry page: lib/app/modules/alert_detail/alert_detail_page.dart
- Affected users: 护工、责任护士、需要沿事件链补充处理结果的协同用户
- Rollout stage: 第五批次级详情流程治理补齐

## User Impact

- 报警详情页现在对摘要卡、建议动作、时间线和两个后续动作提供稳定锚点。
- 当从报警处理页以“正式事件草稿”模式进入时，详情页会切换为责任人/到场时限确认门禁，而不是直接落到普通摘要视图。
- 当事件尚未沉淀处理时间线时，会显示明确空态而不是只剩空白区域。
- 保持 AI 解释和人工补充结果仍由人工主动触发，不自动结案。

## Data Source

- Controller: AlertDetailController
- Mock source: app/data/services/mock_nani_service.dart
- Upstream route: alerts 列表
- Additional upstream draft: alert escalation draft from alerts page

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有处理时间线。
- Error state: 当前无远程拉取，主要验证普通详情模式与正式事件草稿确认模式都能稳定显示。
- Mobile impact: 时间线与双 CTA 在窄屏下仍需保持可读和可点。

## Health Signals

- Healthy signal: 从报警列表进入详情后，事件摘要、时间线和 AI/人工动作入口围绕同一报警对象保持一致；从草稿模式进入时，责任人和到场时限默认可见且可确认。
- Failure signal: 时间线缺失但无空态，或详情页进入 AI 后丢失对象上下文，或草稿模式下无法确认责任人与到场时限。
- Stable selectors: alert-detail-summary-*, alert-detail-action-*, alert-timeline-*, alert-timeline-empty-state, alert-open-ai-*, alert-manual-result-*, alert-draft-detail-summary, alert-draft-trace-card, alert-draft-assignment-card, alert-draft-owner-*, alert-draft-arrival-*, alert-draft-note-input, alert-draft-confirm-button, alert-draft-confirmed-card, alert-draft-open-ai

## Verification

- flutter analyze
- flutter test
- Widget test covers alerts -> alert detail -> AI chain, alert detail empty state, and escalation draft confirmation flow.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原摘要和时间线展示，不改报警 mock 数据结构。