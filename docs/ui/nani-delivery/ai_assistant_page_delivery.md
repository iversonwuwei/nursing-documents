# AI 护理助手交付说明

## Scope

- Entry page: lib/app/modules/ai_assistant/ai_assistant_page.dart
- Affected users: 护工、主管、需要带对象上下文查看建议的协同用户
- Rollout stage: 第二批 AI 页面治理补齐

## User Impact

- AI 页面现在对来源和对象上下文提供稳定锚点，便于验证从不同入口带上下文进入后的表现。
- 当没有建议卡片时显示明确空态，而不是直接空白。
- 保持 AI 建议仍然只读，不引入自动执行动作。

## Data Source

- Controller: AiAssistantController
- Mock source: app/data/services/mock_nani_service.dart
- Upstream context: alerts, health-trend, handover, tasks or root AI tab

## UI States

- Loading state: 当前为本地 mock，同步渲染。
- Empty state: 当前没有可展示的 AI 建议。
- Error state: 当前无远程生成调用，主要验证上下文 banner、建议卡片和边界卡片稳定可见。
- Mobile impact: 上下文 banner、触发器和边界卡片可在小屏稳定回归。

## Health Signals

- Healthy signal: 带对象上下文进入时 banner 正确显示来源和对象，建议卡片与边界卡片可见。
- Failure signal: 上下文丢失、空态缺失或边界卡片不可见。
- Stable selectors: ai-context-banner, ai-insight-*, ai-trigger-shift-summary, ai-trigger-alerts, ai-trigger-handover, ai-boundary-card, ai-empty-state

## Verification

- flutter analyze
- flutter test
- Widget test covers AI tab visibility, contextual banner rendering, and empty state.

## Rollback

- Revert the stable keys, empty-state widget, and related tests.
- 页面回退为原建议列表展示，不改变 AI mock 数据。