# Supplies Delivery Unit

## Scope

- Entry route: src/app/supplies/page.tsx
- Affected users: 采购、仓储、护理站和值班管理协同用户
- Rollout stage: supplies family live read-write 收口，切换到 Admin Web -> Next proxy -> Admin BFF -> Operations Service

## User Impact

- 物资管理页承担物资搜索、分类筛选、库存不足识别和进入入库动作的主入口。
- 主工作区优先保留补货总览、待处理队列、筛选和物资表格，把推荐路径、AI 补货解释和帮助入口后置到信息轨。
- 保持补货和采购仍由人工审批与执行，AI 只做优先级与缺口建议。

## Data Source

- Route type: client page with local search, category filter, and pagination state
- Primary sources: Admin supplies list/detail/intake/activate APIs and backend AI resource-insights
- Downstream links: supply detail/inbound route and AI assistant context links

## UI States

- Loading state: 首屏等待物资列表返回时需保持显式加载反馈。
- Empty state: 搜索或分类筛选无结果时应保持搜索空态；live 返回空集合时显示真实空态。
- Error state: 库存统计、列表和 AI 补货摘要口径不一致时需局部暴露，并且不回退本地库存 mock。
- Mobile impact: 物资表格列较多，后续改动需验证窄屏滚动、信息轨堆叠顺序与入库 CTA 可达性。

## Health Signals

- Healthy signal: KPI、筛选结果、物资列表和入库入口围绕同一库存数据集保持一致。
- Failure signal: 库存不足统计与列表错位，或 AI 补货摘要越过采购审批边界。
- Verification proxy: lint 通过；行为改动时加 build 与物资管理人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证总览指标、优先队列、搜索筛选、空态、入库入口、帮助入口和 AI 链路

## Rollback

- Revert this delivery note together with supplies route、Next proxy、Admin BFF 和 operations supplies endpoints。
- If regressions appear, fallback is the previous local supplies list, filter logic, and AI summary composition.
