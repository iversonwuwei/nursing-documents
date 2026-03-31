# Supplies Detail Delivery Unit

## Scope

- Entry route: src/app/supplies/[id]/page.tsx
- Affected users: 采购、仓储、护理站和值班管理协同用户
- Rollout stage: 第八批详情与根路由治理说明

## User Impact

- 物资详情页承担对象级库存缺口、供应商信息、进出库记录和 AI 补货建议的合流入口。
- 当前交付单元先固定说明和验证门禁，不改现有详情卡片、历史记录和 AI 建议行为。
- 保持补货与采购跟进仍由人工决策，不把详情页变成自动采购执行页。

## Data Source

- Route type: client detail route with params-based local mock lookup
- Primary sources: local supply detail mock and AI supply detail/procurement helpers
- Downstream links: AI assistant context links for补货缺口 and 采购跟进

## UI States

- Loading state: 当前为本地同步 mock；后续接真实库存详情接口时需补对象切换反馈。
- Empty state: 当前未知 id 回退默认物资；若接真实数据需显式 not found 策略。
- Error state: 库存概览、进出库记录与 AI 建议口径不一致时需局部暴露。
- Mobile impact: 多信息卡和进出库表格并存，后续改动需验证窄屏堆叠和 CTA 可达性。

## Health Signals

- Healthy signal: 库存概览、进出库记录和 AI 补货建议围绕同一物资对象保持一致。
- Failure signal: 对象映射错误、历史记录口径错位，或 AI 建议越过采购审批边界。
- Verification proxy: lint 通过；行为改动时加 build 与物资详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证物资详情、进出库记录和 AI 补货/采购跟进链路

## Rollback

- Revert this delivery note and any future supplies detail route changes together.
- If regressions appear, fallback is the previous local supply detail composition and AI summary wording.
