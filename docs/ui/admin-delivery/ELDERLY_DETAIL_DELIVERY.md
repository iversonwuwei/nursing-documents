# Elderly Detail Delivery Unit

## Scope

- Entry route: src/app/elderly/[id]/page.tsx
- Affected users: 护理主管、档案管理、家属沟通与运营协同用户
- Rollout stage: 第二批高频路由治理说明

## User Impact

- 长者详情页继续承担对象主档、健康摘要、家属摘要草稿和管理动作建议的合流入口。
- 详情页现在不再只依赖本地硬编码对象，也能读取新建闭环中的对象并保持 AI 上下文一致。
- 保持现有 AI 摘要、家属摘要草稿和详情编辑入口不变。

## Data Source

- Route type: client detail route with params-based lookup and shared workflow subscription
- Primary sources: elderly-registry merged records, admin AI helpers, family AI profile helpers
- Downstream links: AI assistant contextual links and elderly list back navigation

## UI States

- Loading state: 当前为本地数据映射，后续接真实详情接口时需补首屏和切换对象加载反馈。
- Empty state: 当前仍保留默认对象回退，但新建对象应优先命中 merged registry，避免跳回静态默认对象。
- Error state: AI 摘要与详情数据不一致时应局部可见，不能全页静默。
- Mobile impact: 详情多卡片双栏布局，后续变更要确认窄屏堆叠顺序与 CTA 可达性。

## Health Signals

- Healthy signal: 对象主档、AI 摘要和管理动作卡片在同一对象上下文下保持一致，新建对象进入详情时不再丢失 ID。
- Failure signal: back link、AI 上下文链接或详情对象映射错位，或新建对象被错误回退到默认对象。
- Verification proxy: lint 通过；后续行为改动时加 build 与详情流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 elderly list -> detail -> AI context link 的对象一致性，以及新建对象详情命中 merged registry

## Rollback

- Revert this delivery note and any future detail-route changes together.
- If later detail behavior regresses, fallback is the previous detail composition and local data mapping。
