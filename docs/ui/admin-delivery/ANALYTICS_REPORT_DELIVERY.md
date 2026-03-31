# Analytics Report Delivery Unit

## Scope

- Entry route: src/app/analytics/report/page.tsx
- Affected users: 院长、运营主管、AI 报表查看用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- AI 报表中心当前承接周报、月报摘要草稿、异常解释和导出送达建议。
- 当前交付单元先固定报表中心职责和验证门禁，不修改摘要生成或导出交互行为。
- 保持 AI 报表中心只做草稿和人工确认出口，不直接自动发布正式经营报表。

## Data Source

- Route type: client report summary page
- Primary sources: admission workflow external store, getAiOpsReport, getAiDashboardInsights
- Downstream dependencies: shared PageHeader, StatCard, DataCard and report period state

## UI States

- Loading state: 周报与月报切换目前为本地状态切换；后续接真实报表服务时需补生成与导出反馈。
- Empty state: 若无可纳入报表的申请或 AI 风险信号，应显式提示当前周期暂无摘要内容。
- Error state: 报表摘要、风险解释与 dashboard 摘要口径不一致时应显式暴露。
- Mobile impact: 双栏摘要区、导出建议区和周期按钮组需要验证窄屏折行表现。

## Health Signals

- Healthy signal: 报表中心在周报、月报切换下稳定生成摘要草稿，并保持与 dashboard AI 风险口径一致。
- Failure signal: 周期切换失效、异常解释与 dashboard 分叉，或导出建议脱离当前摘要上下文。
- Verification proxy: lint 通过；行为改动时加 build 与 AI 报表页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证报表中心可切换周报和月报，并展示摘要、异常解释及导出建议

## Rollback

- Revert this delivery note and any future analytics report route changes together.
- If regressions appear, fallback is the current local AI report draft flow.