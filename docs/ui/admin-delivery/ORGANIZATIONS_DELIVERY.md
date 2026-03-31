# Organizations Delivery Unit

## Scope

- Entry route: src/app/organizations/page.tsx
- Affected users: 机构运营、院长、床位协调和集团管理用户
- Rollout stage: 第八批详情与根路由治理说明

## User Impact

- 机构列表页承担机构总览、展开查看局部指标和进入机构详情的主入口。
- 当前交付单元先固定说明和验证门禁，不改现有展开交互、统计卡和 AI 机构摘要行为。
- 保持机构资源调配仍由人工决策，不自动执行机构间调配动作。

## Data Source

- Route type: client page with local expand/collapse state
- Primary sources: organizations and totalStats data plus AI organization helpers
- Downstream links: organization detail and AI assistant context links

## UI States

- Loading state: 当前为本地同步 mock；后续接真实机构列表接口时需补列表反馈。
- Empty state: 当前机构样本非空；若未来无机构数据，应保持列表级空态。
- Error state: 统计卡、机构展开内容与 AI 机构摘要口径不一致时需局部暴露。
- Mobile impact: 列表卡片与展开详情并存，后续改动需验证窄屏折叠顺序和详情入口可达性。

## Health Signals

- Healthy signal: 机构总览、展开指标、详情入口和 AI 机构摘要围绕同一机构数据集保持一致。
- Failure signal: 展开信息错位、详情入口错误，或 AI 调配建议越过人工经营边界。
- Verification proxy: lint 通过；行为改动时加 build 与机构列表人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证机构展开、详情入口和 AI 机构摘要链路

## Rollback

- Revert this delivery note and any future organizations route changes together.
- If regressions appear, fallback is the previous local organizations list and expand/collapse behavior.
