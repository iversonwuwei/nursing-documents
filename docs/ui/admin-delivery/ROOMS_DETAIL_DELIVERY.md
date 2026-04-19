# Rooms Detail Delivery Unit

## Scope

- Entry route: src/app/rooms/[id]/page.tsx
- Affected users: 前台入住协调、床位运营、护理主管与机构协同用户
- Rollout stage: rooms live vertical slice

## User Impact

- 房间详情页承担房间状态、床位占用、设施信息和 AI 房间建议的对象级查看入口。
- 主工作区优先保留对象状态、床位占用、房间事实和设施信息，把 AI 解释、照护建议和帮助入口后置到信息轨。
- 详情页现在也能读取新建待启用房间，不再只依赖本地硬编码对象。
- 保持床位安排和照护动作仍由人工决策，不自动完成入住分配。

## Data Source

- Route type: client detail route with params-based live fetch
- Primary sources: Next `/api/rooms/{id}` -> Admin BFF `/api/admin/rooms/{id}` -> Rooms service + elder aggregation
- Downstream links: elderly detail links and AI assistant context links

## UI States

- Loading state: 详情页在切换房间时展示 live loading。
- Empty state: 未找到房间时显式 not found/empty，不回退默认房间。
- Error state: 房间概览、床位占用与入住对象聚合失败时需显式暴露。
- Mobile impact: 床位卡片、房间信息和设施标签并存，后续改动需验证窄屏堆叠与 CTA 可达性。

## Health Signals

- Healthy signal: 房间概览、待启用状态、床位占用、对象链接和 AI 建议围绕同一房间对象保持一致。
- Failure signal: 房间对象映射错误、床位占用错位，或 live 详情页重新回退到前端默认对象。
- Verification proxy: lint 通过；行为改动时加 build 与房间详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证房间详情、床位对象链接、帮助入口和 AI 房间建议链路

## Rollback

- Revert this delivery note and any future rooms detail route changes together.
- If regressions appear, rollback the live detail route together; do not keep mixed live/mock room detail composition.
