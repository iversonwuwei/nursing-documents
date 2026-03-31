# Equipment Delivery Unit

## Scope

- Entry route: src/app/equipment/page.tsx
- Affected users: 设备运维、护理站、值班管理与后勤协同用户
- Rollout stage: 第五批高频运营路由治理说明

## User Impact

- 设备列表页承担设备搜索、分类筛选、状态查看和进入详情/监控的主入口。
- 当前交付单元先固定说明和验证门禁，不改现有设备列表、分类筛选和 AI 巡检摘要行为。
- 保持巡检和维保动作仍由人工安排，AI 只提供巡检顺序和备用建议。

## Data Source

- Route type: client page with local search, category filter, and pagination state
- Primary sources: equipmentList, equipmentAlarms, and AI equipment helpers
- Downstream links: device detail, realtime monitor, and AI assistant context links

## UI States

- Loading state: 当前为本地同步 mock；后续接设备实时状态时需补筛选和分页反馈。
- Empty state: 搜索或分类筛选无结果时应保持搜索空态。
- Error state: 设备状态、告警数量与 AI 巡检建议口径不一致时需局部暴露。
- Mobile impact: 设备表格和状态标签在窄屏下需验证列压缩与 CTA 可达性。

## Health Signals

- Healthy signal: KPI、筛选结果、设备列表和详情/监控入口围绕同一设备数据集保持一致。
- Failure signal: 告警统计和设备状态错位，或 AI 巡检建议越过人工维保边界。
- Verification proxy: lint 通过；行为改动时加 build 与设备列表人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、分类筛选、空态、详情入口、监控入口和 AI 链路

## Rollback

- Revert this delivery note and any future equipment route changes together.
- If regressions appear, fallback is the previous local equipment list, filter logic, and AI summary composition.
