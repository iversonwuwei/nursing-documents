# 交接详情交付说明

## Scope

- Entry page: lib/app/modules/handoff/handoff_page.dart
- Affected users: 护工、主管、需要沿交接详情继续执行确认动作的协同用户
- Rollout stage: 第五批次级详情流程治理补齐

## User Impact

- 交接详情页现在对 hero、人工确认步骤、升级边界和三个后续动作提供稳定锚点。
- 当交接详情尚未生成确认步骤时，会显示明确空态而不是只留下空列表。
- 保持交接最终确认仍然是人工决定，AI 只生成草稿和解释。

## Data Source

- Controller: HandoffController
- Mock source: app/data/services/mock_nani_service.dart
- Upstream route: handover 列表

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有人工确认项。
- Error state: 当前无远程拉取，主要验证 hero、确认步骤、升级边界与动作入口稳定可见。
- Mobile impact: hero 信息、确认清单和动作卡在窄屏下仍需保持上下文完整和 CTA 可达。

## Health Signals

- Healthy signal: 从交接班进入详情后，对象、确认步骤、升级边界和对象/AI 跳转保持一致。
- Failure signal: 步骤为空但无空态，或对象详情、AI 草稿入口丢失当前交接上下文。
- Stable selectors: handoff-hero-*, handoff-meta-owner, handoff-meta-due, handoff-meta-updated, handoff-step-*-*, handoff-steps-empty-state, handoff-escalation-*, handoff-record-confirm-*, handoff-open-resident-*, handoff-open-ai-*

## Verification

- flutter analyze
- flutter test
- Widget test covers handover -> handoff detail -> resident detail chain, step toggle, and handoff empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原交接详情结构，不改交接 mock 数据结构。