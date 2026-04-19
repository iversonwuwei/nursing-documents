# Elderly Detail Delivery Unit

## Scope

- Entry route: src/app/elderly/[id]/page.tsx
- Affected users: 护理主管、档案管理、家属沟通与运营协同用户
- Rollout stage: 第二批高频路由治理说明

## User Impact

- 长者详情页继续承担对象主档、健康摘要、家属摘要草稿和管理动作建议的合流入口。
- 详情页现在直接依赖 Elder Service 与 Health Service 的真实主档和健康摘要，不再回退本地硬编码对象或 shared workflow 补位。
- 保持现有 AI 摘要、家属摘要草稿和详情编辑入口不变。

## Data Source

- Route type: client detail route with params-based live profile and live health lookup
- Primary sources: Admin BFF `/api/admin/elders/{elderId}` and `/api/admin/elders/{elderId}/health-summary`
- Downstream links: AI assistant contextual links and elderly list back navigation

## UI States

- Loading state: profile 与 health 分别显示同步中，避免整页空白阻塞。
- Empty state: 当前不再保留默认对象回退，未同步字段统一显示待同步。
- Error state: profile / health 任一路径失败时要显式暴露状态，不能全页静默，也不能回退到本地默认对象。
- Mobile impact: 详情多卡片双栏布局，后续变更要确认窄屏堆叠顺序与 CTA 可达性。

## Health Signals

- Healthy signal: 对象主档、健康摘要和摘要卡片在同一对象上下文下保持一致，新建对象进入详情时不再丢失 ID。
- Failure signal: back link、AI 上下文链接或详情对象映射错位，或页面重新回退到本地默认对象。
- Verification proxy: lint 通过；后续行为改动时加 build 与详情流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 elderly list -> detail -> AI context link 的对象一致性，以及新建对象详情命中 live profile / live health

## Rollback

- Revert this delivery note and any future detail-route changes together.
- If later detail behavior regresses, fallback is the previous detail composition and BFF detail read path。
