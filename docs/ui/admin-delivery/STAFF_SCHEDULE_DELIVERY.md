# Staff Schedule Delivery Unit

## Scope

- Entry route: src/app/staff/schedule/page.tsx
- Affected users: 排班主管、值班管理、护理团队协调用户
- Rollout stage: 派案排期 live read model 收口

## User Impact

- 排班管理页承担周视图排班、班次密度摘要和 AI 调整建议的统一入口。
- 主工作区优先保留真实周视图总览、真实排班矩阵和真实每日汇总，把 live 边界和帮助入口后置到信息轨。
- 页面当前阶段只读展示真实 workflow board，不再回退本地 demo 派案板，也不再在排班页混用本地 AI 调整摘要。
- 班次调整仍由人工发布和确认；排班页只承担读模型，不承接前端 demo reset 或本地修补动作。

## Data Source

- Route type: client page with live workflow board read model
- Primary source: `/api/nursing/workflow/board` 的真实排班快照
- Supporting source: 页面内仅保留基于真实排班快照的 deterministic 摘要，不再读取 local schedule mocks 或 local AI helpers
- Downstream links: AI assistant context links for schedule density and adjustment

## UI States

- Loading state: 首屏等待真实 workflow board 返回时显示明确同步态。
- Empty state: 若某周真实排班为空，应保持网格级 live 空态而不是回退 demo 数据。
- Error state: workflow board 不可用时，页面保留显式 live 错误态，不再转成 demo workflow。
- Mobile impact: 周视图网格列多，后续改动需验证横向滚动、信息轨堆叠和发布/导出 CTA 可达性。

## Health Signals

- Healthy signal: 周视图、班次统计、重点计划和模板缺口都围绕同一条真实 workflow board 快照保持一致。
- Failure signal: 页面重新混入 `nursing-service-workflow` demo fallback、`Demo Workflow` 状态或本地 AI 排班摘要。
- Verification proxy: lint 通过；行为改动时加 build 与排班页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 live workflow board 加载、排班网格、日统计、帮助入口，以及链路失败时的 live 错误态

## Rollback

- Revert this delivery note and any future staff schedule route changes together.
- If regressions appear, rollback is the previous mixed live-plus-demo schedule page, but that will restore local fallback and dual source-of-truth risk.
