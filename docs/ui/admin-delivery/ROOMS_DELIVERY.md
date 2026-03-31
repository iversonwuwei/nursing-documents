# Rooms Delivery Unit

## Scope

- Entry route: src/app/rooms/page.tsx
- Affected users: 前台入住协调、床位运营、护理主管与机构协同用户
- Rollout stage: 第五批高频运营路由治理说明

## User Impact

- 房间管理页承担搜索房间、查看入住率、判断可入住资源和进入房间详情的主入口。
- 当前交付单元先固定说明与验证门禁，不改现有房间列表、入住率和 AI 排房摘要行为。
- 保持排房仍由人工决策，AI 只提供解释与分配建议。

## Data Source

- Route type: client page with local search and pagination state
- Primary sources: local room mocks, organizations data, and AI room helpers
- Downstream links: room detail and AI assistant context links

## UI States

- Loading state: 当前为本地同步 mock；后续接实时床位接口时需补搜索与分页反馈。
- Empty state: 搜索无结果时应保持搜索空态。
- Error state: 房间入住率、机构数量与 AI 分配建议口径不一致时需局部暴露。
- Mobile impact: 房间表格和入住率展示在窄屏下需验证滚动顺序与查看 CTA 可达性。

## Health Signals

- Healthy signal: 房间统计、入住率、房间列表和查看详情入口围绕同一房间数据集保持一致。
- Failure signal: 入住率和房间状态错位，或 AI 摘要越过“建议而非自动排房”的边界。
- Verification proxy: lint 通过；行为改动时加 build 与房间管理人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、空态、入住率、房间详情入口和 AI 链路

## Rollback

- Revert this delivery note and any future rooms route changes together.
- If regressions appear, fallback is the previous local room list, occupancy visualization, and AI summary composition.
