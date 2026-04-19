# Rooms Delivery Unit

## Scope

- Entry route: src/app/rooms/page.tsx
- Affected users: 前台入住协调、床位运营、护理主管与机构协同用户
- Rollout stage: rooms live vertical slice

## User Impact

- 房间管理页承担搜索房间、查看入住率、判断可入住资源和进入房间详情的主入口。
- 主工作区优先保留承接总览、优先队列、房间表格和启用动作，把推荐路径、AI 说明和完整帮助后置到信息轨。
- 列表现在同时承接新建房间待启用闭环，可在列表页直接完成启用动作。
- 保持排房仍由人工决策，AI 只提供解释与分配建议。

## Data Source

- Route type: client page with local search and pagination state
- Primary sources: Next `/api/rooms` -> Admin BFF `/api/admin/rooms` -> Rooms service persisted room records；入住对象由 Admin BFF 聚合真实 elder list
- Downstream links: room new page, room detail and AI assistant context links

## UI States

- Loading state: 从 live rooms API 加载房间主档与入住摘要。
- Empty state: 搜索无结果时应保持搜索空态。
- Error state: 下游 rooms service 或 elder aggregation 失败时显式展示 live error，不回退本地 room workflow。
- Mobile impact: 房间表格和入住率展示在窄屏下需验证滚动顺序、信息轨堆叠和查看 CTA 可达性。

## Health Signals

- Healthy signal: 房间统计、待启用状态、房间列表和查看详情入口围绕同一房间数据集保持一致。
- Failure signal: 待启用房间被直接计入可入住、入住率和房间状态错位，或页面重新读取本地 room workflow / mock AI。
- Verification proxy: lint 通过；行为改动时加 build 与房间管理人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证房间新建、待启用提示、启用动作、搜索空态、帮助入口、房间详情入口和 AI 链路

## Rollback

- Revert this delivery note and any future rooms route changes together.
- If regressions appear, rollback the live rooms routes together; do not keep partial dual-read behavior.
