# Financial Delivery Unit

## Scope

- Entry route: src/app/financial/page.tsx
- Affected users: 财务负责人、运营管理、院长与经营分析协同用户
- Rollout stage: 第五批高频运营路由治理说明

## User Impact

- 财务收支页承担月度收支总览、利润结构和 AI 经营解读的统一入口。
- 当前交付单元先固定说明与验证门禁，不改现有 KPI、构成图和 AI 经营解读行为。
- 保持 AI 输出仍是经营解释与建议，不替代正式记账、预算审批或财务结论。

## Data Source

- Route type: client page with local monthly aggregates and breakdown data
- Primary sources: local monthly, category, and expense mocks plus AI financial helpers
- Downstream links: AI assistant context links for inference and logs

## UI States

- Loading state: 当前为本地同步 mock；后续接财务接口时需补月度汇总与图表反馈。
- Empty state: 当前静态样本非空；若未来月度数据为空，需保持局部空态而不是整页失真。
- Error state: KPI、收入支出构成与 AI 解读口径不一致时需局部暴露。
- Mobile impact: KPI 卡、双列分析卡与构成图较多，后续改动需验证窄屏堆叠顺序和按钮可达性。

## Health Signals

- Healthy signal: KPI、收入支出构成和 AI 解读围绕同一月度财务样本保持一致。
- Failure signal: 月度口径与构成图错位，或 AI 建议越过“需财务确认”的边界。
- Verification proxy: lint 通过；行为改动时加 build 与财务页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 KPI、收入支出构成、AI 经营解读和 AI 链路

## Rollback

- Revert this delivery note and any future financial route changes together.
- If regressions appear, fallback is the previous local finance dashboard composition and AI wording.
