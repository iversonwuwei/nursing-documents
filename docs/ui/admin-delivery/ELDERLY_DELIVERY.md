# Elderly Route Delivery Unit

## Scope

- Entry route: src/app/elderly/page.tsx
- Affected users: 前台接待、护理主管、运营与档案管理用户
- Rollout stage: 老人台账 Fluent 收敛与主入口强化

## User Impact

- 老人列表页继续承担搜索、筛选、分页和进入详情的主入口职责，但首屏进一步收敛为“台账总览 + 关键治理闭环 + 列表处理区”。
- 新增老人与资料导入继续进入统一治理链路，但说明性闭环块需要采用 Fluent 风格的轻量治理卡，而不是旧式统计块。
- 人脸录入状态与快捷动作继续保留在主列表，避免前台在详情页和人脸页之间反复跳转查找对象。
- 页面上下文说明收敛到后置区域或帮助页，不再与主筛选和列表并排竞争首屏注意力。

## Data Source

- Route type: client page with server-backed list query plus local filter state
- Primary data: Admin BFF `/api/admin/elders` 聚合的 Elder Service 真实主档；人脸状态在本轮不再以 shared mock store 作为事实源，而是显式标记为待接通
- Downstream links: elderly detail pages, elderly new page, elderly import page, checkin workflow entry, and face enrollment workflow entry
- Dependent systems: Admin Next route proxy, Admin BFF, Elder Service, 本地 PostgreSQL seed 数据

## UI States

- Loading state: 首屏与筛选切换需要显式展示 live list 加载反馈，避免把旧台账误认为已同步。
- Empty state: 搜索或筛选无结果时维持 EmptyState 搜索空态。
- Error state: Elder Service / Admin BFF 返回失败时显式暴露列表读取错误，不再静默回退到静态老人台账。
- Mobile impact: KPI、治理闭环卡和筛选区继续保持纵向阅读顺序；表格在较窄宽度下仍需确认横向滚动和操作列可达性。

## Health Signals

- Healthy signal: 首屏先完成“判断当前台账压力 -> 进入审核、导入或详情处理”的闭环；搜索、护理等级筛选、状态筛选和分页全部建立在 Elder Service 实时返回之上。
- Failure signal: 列表总数、分页结果和详情入口对应不上同一批 elderId，或页面重新落回静态台账数据。
- Verification proxy: lint 通过；行为变更时加 build 与手工列表流回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证新增老人入口、资料导入入口、入住审核入口、搜索筛选分页、详情进入，以及新建对象写入 Elder Service 后回流列表

## Rollback

- Revert this route delivery note and any future elderly list route changes together.
- If later list behavior regresses, fallback is the previous static 台账列表实现，并回退 Admin BFF / Elder Service 的列表接入。
