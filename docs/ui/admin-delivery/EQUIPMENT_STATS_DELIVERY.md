# Equipment Stats Delivery Unit

## Scope

- Entry route: src/app/equipment/stats/page.tsx
- Affected users: 设备运营、院务分析、后勤管理用户
- Rollout stage: 第九批设备源路由治理说明

## User Impact

- 设备统计页当前承接设备总量、使用趋势、类型分布和高频设备排行的汇总分析。
- 当前交付单元先固定统计页职责和验证门禁，不修改统计口径或图表展示行为。
- 保持本周与本月切换按钮、趋势图和 TOP 设备排序的现有表达方式不变。

## Data Source

- Route type: client analytics summary page
- Primary sources: in-file WEEKLY, TYPE_DATA, top device ranking mock data
- Downstream dependencies: shared DataCard layout and CSS-based chart components

## UI States

- Loading state: 当前为本地静态统计；后续接真实统计服务时需补周期切换加载反馈。
- Empty state: 周趋势、类型分布或排行为空时应显式提示无统计样本，而不是展示空图表骨架。
- Error state: 周期切换、排行和类型分布口径不一致时需显式暴露统计失败。
- Mobile impact: 四列 KPI、双栏图表和排行列表需要验证窄屏压缩后的可读性。

## Health Signals

- Healthy signal: 设备统计页稳定展示统一统计周期下的趋势、分布和排行数据。
- Failure signal: 周期切换无效、排行与概览统计冲突，或图表布局在窄屏失真。
- Verification proxy: lint 通过；行为改动时加 build 与统计页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证统计页可查看趋势图、类型分布和 TOP 设备排行，且周期按钮语义清晰

## Rollback

- Revert this delivery note and any future equipment stats route changes together.
- If regressions appear, fallback is the current local统计展示 without backend coupling.