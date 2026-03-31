# Staff Delivery Unit

## Scope

- Entry route: src/app/staff/page.tsx
- Affected users: 人力运营、护理主管、值班管理与排班协同用户
- Rollout stage: 第五批高频运营路由治理说明

## User Impact

- 员工列表页承担员工搜索、部门筛选、状态浏览与进入详情/排班的主入口职责。
- 当前交付单元先固定说明和验证门禁，不改现有列表、筛选与 AI 摘要行为。
- 保持详情和排班入口仍由人工选择，不把 AI 摘要变成员工绩效判断。

## Data Source

- Route type: client page with local search, filter, and pagination state
- Primary sources: local staff mocks and admin AI workforce helpers
- Downstream links: staff detail, schedule, and AI assistant context links

## UI States

- Loading state: 当前为本地同步 mock；后续接人事与排班接口时需补筛选和分页反馈。
- Empty state: 搜索或部门筛选无结果时应保持搜索空态。
- Error state: 列表、KPI 和 AI 摘要口径不一致时需局部暴露。
- Mobile impact: 表格列较多，后续改动需验证窄屏下搜索、筛选和详情 CTA 可达性。

## Health Signals

- Healthy signal: KPI、筛选结果、员工列表和详情/排班入口围绕同一员工数据集保持一致。
- Failure signal: 列表与统计口径错位，或 AI 摘要越过“结构建议而非绩效判断”的边界。
- Verification proxy: lint 通过；行为改动时加 build 与员工列表人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、部门筛选、空态、详情入口、排班入口和 AI 链路

## Rollback

- Revert this delivery note and any future staff route changes together.
- If regressions appear, fallback is the previous local staff list, filter, and AI summary composition.
