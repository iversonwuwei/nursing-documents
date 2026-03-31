# Rooms Detail Delivery Unit

## Scope

- Entry route: src/app/rooms/[id]/page.tsx
- Affected users: 前台入住协调、床位运营、护理主管与机构协同用户
- Rollout stage: 第八批详情与根路由治理说明

## User Impact

- 房间详情页承担房间状态、床位占用、设施信息和 AI 房间建议的对象级查看入口。
- 当前交付单元先固定说明与验证门禁，不改现有房间信息、床位展示和 AI 建议行为。
- 保持床位安排和照护动作仍由人工决策，不自动完成入住分配。

## Data Source

- Route type: client detail route with params-based local mock lookup
- Primary sources: local room detail mock and AI room detail/care helpers
- Downstream links: elderly detail links and AI assistant context links

## UI States

- Loading state: 当前为本地同步 mock；后续接真实房间详情接口时需补对象切换反馈。
- Empty state: 当前未知 id 回退默认房间；若接真实数据需显式 not found 策略。
- Error state: 房间概览、床位占用与 AI 建议口径不一致时需局部暴露。
- Mobile impact: 床位卡片、房间信息和设施标签并存，后续改动需验证窄屏堆叠与 CTA 可达性。

## Health Signals

- Healthy signal: 房间概览、床位占用、对象链接和 AI 建议围绕同一房间对象保持一致。
- Failure signal: 房间对象映射错误、床位占用错位，或 AI 建议越过人工排房边界。
- Verification proxy: lint 通过；行为改动时加 build 与房间详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证房间详情、床位对象链接和 AI 房间建议链路

## Rollback

- Revert this delivery note and any future rooms detail route changes together.
- If regressions appear, fallback is the previous local room detail composition and AI summary wording.
