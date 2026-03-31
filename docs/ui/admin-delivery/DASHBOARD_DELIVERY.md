# Dashboard Delivery Unit

## Scope

- Entry route: src/app/dashboard/page.tsx
- Affected users: 管理端值班、运营、护理主管
- Rollout stage: 第一批高频路由治理说明

## User Impact

- Dashboard 作为管理端着陆页，需要把当天总览、风险和待处理事项压缩成可快速判断的入口。
- 当前交付单元先固化说明和验证标准，不修改页面行为。
- 保持现有首页入口和卡片布局不变。

## Data Source

- Route type: Next.js client route re-exporting root landing page
- Upstream dependencies: 当前沿用现有 mock 或本地数据聚合
- Adjacent routes: src/app/page.tsx, src/app/alerts/page.tsx, src/app/elderly/page.tsx

## UI States

- Loading state: 仪表盘卡片应能表达数据尚未就绪的占位或缺省策略。
- Empty state: 当日无新增风险或运营动作时，需要有明确“无待办”表达，而不是纯空白。
- Error state: 卡片级错误不应让整页失效，应保留局部降级能力。
- Mobile impact: 卡片密度较高，后续改动需要确认窄屏换行和滚动顺序。

## Health Signals

- Healthy signal: 管理用户可以在首屏看到关键 KPI、待处理项和跳转入口。
- Failure signal: 着陆页信息过载、关键风险不可见或卡片缺少可回归的稳定状态。
- Verification proxy: lint 通过，后续行为变更时补 build 和可复现 UI 流。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 校验 dashboard 与 landing route 的卡片顺序、总览文案和关键跳转

## Rollback

- Revert this delivery note and any future dashboard-specific route changes together.
- If runtime behavior changes later, rollback should restore the previous landing composition without touching adjacent routes.