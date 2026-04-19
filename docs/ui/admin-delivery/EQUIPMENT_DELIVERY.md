# Equipment Delivery Unit

## Scope

- Entry route: src/app/equipment/page.tsx
- Affected users: 设备运维、护理站、值班管理与后勤协同用户
- Rollout stage: equipment family live read-write 收口，切换到 Admin Web -> Next proxy -> Admin BFF -> Operations Service

## User Impact

- 设备列表页承担设备搜索、分类筛选、状态查看和进入详情/监控的主入口。
- 主区应只保留优先队列、筛选表格和关键 KPI，AI 巡检解释、维保叙述和培训性说明迁移到后置上下文和帮助页。
- 保持巡检和维保动作仍由人工安排，AI 只提供巡检顺序和备用建议。

## Data Source

- Route type: client page with local search, category filter, and pagination state
- Primary sources: Admin equipment list, detail, activate APIs and backend AI device-insights
- Downstream links: device detail, realtime monitor, and AI assistant context links

## UI States

- Loading state: 首屏等待设备列表返回时需保持显式加载反馈。
- Empty state: 搜索或分类筛选无结果时应保持搜索空态；live 返回空集合时显示真实空态。
- Error state: 设备状态、激活动作与 AI 巡检建议口径不一致时需局部暴露，并保留帮助入口；不再回退 equipmentList 本地样例。
- Mobile impact: 设备表格、优先队列、后置 AI 摘要和帮助卡在窄屏下需验证纵向堆叠与 CTA 可达性。

## Health Signals

- Healthy signal: KPI、筛选结果、设备列表和详情/监控入口围绕同一设备数据集保持一致，说明型内容不再挤入主表格上方。
- Failure signal: 告警统计和设备状态错位，或 AI 巡检建议越过人工维保边界。
- Verification proxy: lint 通过；行为改动时加 build 与设备列表人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、分类筛选、空态、详情入口、监控入口、后置 AI 卡片和帮助页入口

## Rollback

- Revert this delivery note together with equipment route、devices compatibility route、Next proxy、Admin BFF 和 operations equipment endpoints。
- If regressions appear, fallback is the previous local equipment list, filter logic, and AI summary composition.
