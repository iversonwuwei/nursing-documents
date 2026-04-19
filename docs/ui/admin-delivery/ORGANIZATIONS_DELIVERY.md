# Organizations Delivery Unit

## Scope

- Entry route: src/app/organizations/page.tsx
- Affected users: 机构运营、院长、床位协调和集团管理用户
- Rollout stage: organizations live vertical slice

## User Impact

- 机构列表页承担机构总览、展开查看局部指标和进入机构详情的主入口。
- 列表现在同时承接新建机构待启用闭环，可在列表页直接完成启用动作。
- 保持机构资源调配仍由人工决策，不自动执行机构间调配动作。

## Data Source

- Route type: client page with local expand/collapse state
- Primary sources: Next `/api/organizations` -> Admin BFF `/api/admin/organizations` -> Organization service persisted records + rooms live aggregation
- Downstream links: organization new page, organization detail and AI assistant context links

## UI States

- Loading state: 从 live organizations API 加载机构主档和床位摘要。
- Empty state: 当前机构样本非空；若未来无机构数据，应保持列表级空态。
- Error state: 下游 organization service 或 rooms aggregation 失败时显式展示 live error，不回退本地 organizations workflow。
- Mobile impact: 列表卡片与展开详情并存，后续改动需验证窄屏折叠顺序和详情入口可达性。

## Health Signals

- Healthy signal: 机构总览、待启用状态、详情入口和 AI 机构摘要围绕同一机构数据集保持一致。
- Failure signal: 新建机构无法启用、展开信息错位、详情入口错误，或页面重新读取本地 organizations mock / local AI helpers。
- Verification proxy: lint 通过；行为改动时加 build 与机构列表人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证机构新建、待启用提示、启用动作、展开详情和 AI 机构摘要链路

## Rollback

- Revert this delivery note and any future organizations route changes together.
- If regressions appear, rollback the live organizations list together; do not keep partial dual-read behavior.
