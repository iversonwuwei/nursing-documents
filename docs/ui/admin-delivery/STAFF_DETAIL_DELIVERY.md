# Staff Detail Delivery Unit

## Scope

- Entry route: src/app/staff/[id]/page.tsx
- Affected users: 人力主管、护理主管、值班管理与员工协同用户
- Rollout stage: 第十九批物资、房间与员工页面主区收口

## User Impact

- 员工详情页承担个人基本信息、排班、绩效、资质与 AI 班次摘要的合流入口。
- 主工作区优先保留人员事实、排班、绩效和资质信息，把 AI 动作摘要、交接草稿和帮助入口后置到信息轨。
- 保持 AI 只提供班次与交接建议，不把详情页变成自动绩效结论页。

## Data Source

- Route type: client detail route with params-based local mock lookup
- Primary sources: local staff detail mock, app-ai staff profile helpers, and admin AI detail helpers
- Downstream links: AI assistant context links for staff action, shift summary, and handover draft

## UI States

- Loading state: 当前为本地同步 mock；后续接真实员工详情接口时需补对象切换反馈。
- Empty state: 当前未知 id 回退默认员工；若接真实数据需显式 not found 策略。
- Error state: 基本信息、排班、绩效和 AI 摘要口径不一致时需局部暴露。
- Mobile impact: 多信息卡和双列布局并存，后续改动需验证窄屏堆叠和 CTA 可达性。

## Health Signals

- Healthy signal: 基本信息、排班、绩效和 AI 摘要围绕同一员工对象保持一致。
- Failure signal: 对象映射错误、AI 链路错位，或详情页越过人工管理边界输出结论。
- Verification proxy: lint 通过；行为改动时加 build 与员工详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证员工详情、排班展示、帮助入口、AI 班次摘要和交接草稿链路

## Rollback

- Revert this delivery note and any future staff detail route changes together.
- If regressions appear, fallback is the previous local staff detail composition and AI summary wording.
