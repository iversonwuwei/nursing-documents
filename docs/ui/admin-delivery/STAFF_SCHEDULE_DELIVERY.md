# Staff Schedule Delivery Unit

## Scope

- Entry route: src/app/staff/schedule/page.tsx
- Affected users: 排班主管、值班管理、护理团队协调用户
- Rollout stage: 第十九批物资、房间与员工页面主区收口

## User Impact

- 排班管理页承担周视图排班、班次密度摘要和 AI 调整建议的统一入口。
- 主工作区优先保留周视图总览、排班矩阵和每日汇总，把 AI 摘要、live 只读边界和帮助入口后置到信息轨。
- 保持班次调整仍由人工发布和确认，AI 只提供密度与风险解释。

## Data Source

- Route type: client page with local week offset state and local schedule matrix
- Primary sources: local schedule mocks and AI schedule helpers
- Downstream links: AI assistant context links for schedule density and adjustment

## UI States

- Loading state: 当前为本地同步 mock；后续接真实排班系统时需补周切换与发布反馈。
- Empty state: 当前静态样本非空；若未来某周无排班数据，应保持网格级空态而不是只剩空白。
- Error state: KPI、排班网格和 AI 调整建议口径不一致时需局部暴露。
- Mobile impact: 周视图网格列多，后续改动需验证横向滚动、信息轨堆叠和发布/导出 CTA 可达性。

## Health Signals

- Healthy signal: 周视图、班次统计和 AI 调整建议围绕同一排班矩阵保持一致。
- Failure signal: 周切换后统计错位，或 AI 调整建议越过排班主管人工确认边界。
- Verification proxy: lint 通过；行为改动时加 build 与排班页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证周切换、排班网格、日统计、帮助入口和 AI 链路

## Rollback

- Revert this delivery note and any future staff schedule route changes together.
- If regressions appear, fallback is the previous local schedule matrix and AI summary composition.
