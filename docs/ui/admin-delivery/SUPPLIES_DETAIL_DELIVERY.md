# Supplies Detail Delivery Unit

## Scope

- Entry route: src/app/supplies/[id]/page.tsx
- Affected users: 采购、仓储、护理站和值班管理协同用户
- Rollout stage: 物资详情 Fluent 收口批次

## User Impact

- 物资详情页继续承担对象级库存缺口、供应商信息、进出库记录和 AI 补货建议，但主区先展示对象事实和库存台账，再把 AI 解释、采购跟进和帮助入口后置。
- 当前交付单元把详情页从多张并列卡片收口到统一的 Fluent 纵向对象页，不改变补货与采购仍由人工决策的边界。
- 页面需补显式帮助承接面，避免补货与采购口径继续堆在详情页主工作区。

## Data Source

- Route type: client detail route with params-based local mock lookup
- Primary sources: local supply detail mock and AI supply detail/procurement helpers
- Downstream links: AI assistant context links for补货缺口 and 采购跟进

## UI States

- Loading state: 当前为本地同步对象；对象切换时详情骨架保持稳定。
- Empty state: 未命中对象时需显式 not found，而不是继续回退默认物资。
- Error state: 库存概览、进出库记录与 AI 建议口径不一致时需局部暴露。
- Mobile impact: 详情页保持对象总览、台账、后置上下文的单列顺序，不恢复多卡并排。

## Health Signals

- Healthy signal: 库存概览、进出库记录和后置上下文围绕同一物资对象保持一致。
- Failure signal: 对象映射错误、历史记录口径错位、未命中对象仍回退默认数据，或 AI 建议越过采购审批边界。
- Verification proxy: lint 通过；行为改动时加 build 与物资详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证物资详情、进出库记录和 AI 补货/采购跟进链路

## Rollback

- Revert this delivery note and any future supplies detail route changes together.
- If regressions appear, fallback is the previous local supply detail composition and AI summary wording.
