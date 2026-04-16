# Organization Detail Delivery Unit

## Scope

- Entry route: src/app/organizations/[id]/page.tsx
- Affected users: 机构运营、床位协调、院长与人力协同用户
- Rollout stage: 机构详情 Fluent 收口批次

## User Impact

- 机构详情页继续承担机构概览、床位管理、员工管理与对象级 AI 组织摘要，但首屏先展示对象总览和当前 tab，再把状态、AI 解释和帮助入口后置。
- 详情页继续能读取新建待启用机构，不再只依赖本地硬编码对象。
- 页面不再保留无实际落点的假编辑主按钮；待启用机构保留启用动作，其余对象以只读治理与下钻为主。

## Data Source

- Route type: client detail route with local tab state and shared workflow subscription
- Primary sources: master-data-workflow merged organization、派生床位数据、员工名册和 AI organization helpers
- Downstream links: AI assistant context links for overview, beds, and staff roster

## UI States

- Loading state: 当前为本地同步对象；tab 切换不应打断详情骨架。
- Empty state: 未命中机构时需显式提示对象不存在，而不是继续回退默认对象。
- Error state: 概览、床位、员工与 AI 摘要口径不一致时需在当前 tab 或上下文区局部暴露。
- Mobile impact: 详情页保持单列纵向结构，tab、KPI、台账与帮助入口都要可达。

## Health Signals

- Healthy signal: 机构概览、待启用状态、床位管理、员工管理和后置上下文围绕同一机构上下文保持一致。
- Failure signal: tab 内容错位、机构对象映射错误、详情页仍保留无落点主按钮，或帮助边界重新回到首屏长说明。
- Verification proxy: lint 通过；行为改动时加 build 与机构详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证机构概览、床位 tab、员工 tab 和 AI 链路

## Rollback

- Revert this delivery note and any future organization detail route changes together.
- If regressions appear, fallback is the previous local organization detail tabs and AI summary composition.
