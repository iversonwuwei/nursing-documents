# Supplies Delivery Unit

## Scope

- Entry route: src/app/supplies/page.tsx
- Affected users: 采购、仓储、护理站和值班管理协同用户
- Rollout stage: 第六批设备与组织扩展路由治理说明

## User Impact

- 物资管理页承担物资搜索、分类筛选、库存不足识别和进入入库动作的主入口。
- 当前交付单元先固定说明与验证门禁，不改现有物资列表、筛选和 AI 补货摘要行为。
- 保持补货和采购仍由人工审批与执行，AI 只做优先级与缺口建议。

## Data Source

- Route type: client page with local search, category filter, and pagination state
- Primary sources: local supplies mocks and AI supply helpers
- Downstream links: supply detail/inbound route and AI assistant context links

## UI States

- Loading state: 当前为本地同步 mock；后续接库存系统时需补筛选和分页反馈。
- Empty state: 搜索或分类筛选无结果时应保持搜索空态。
- Error state: 库存统计、列表和 AI 补货摘要口径不一致时需局部暴露。
- Mobile impact: 物资表格列较多，后续改动需验证窄屏滚动与入库 CTA 可达性。

## Health Signals

- Healthy signal: KPI、筛选结果、物资列表和入库入口围绕同一库存数据集保持一致。
- Failure signal: 库存不足统计与列表错位，或 AI 补货摘要越过采购审批边界。
- Verification proxy: lint 通过；行为改动时加 build 与物资管理人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、分类筛选、空态、入库入口和 AI 链路

## Rollback

- Revert this delivery note and any future supplies route changes together.
- If regressions appear, fallback is the previous local supplies list, filter logic, and AI summary composition.
