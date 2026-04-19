# Operations Daily Delivery Unit

## Scope

- Entry route: src/app/operations/daily/page.tsx
- Affected users: 值班主管、机构运营、居家协同人员、护理执行协调用户
- Rollout stage: 日班工作台 Fluent 试点与首屏收敛

## User Impact

- 日班工作台承担当班风险收口、任务优先级排序和跨页入口聚合，是 admin 高频入口之一。
- 本次交付在既有真实化基础上，把页面进一步收敛为 live aggregate 驱动的班次入口：主区只保留收口总览、优先队列和真实入口，不再展示本地事故、活动和资源假数。
- 来源边界与待接通模块说明不再拆成多张并列说明卡，而是压到后置上下文区，避免值班用户在首屏同时处理多个信息轴。
- AI 运营视角不再作为日班工作台头部主动作；需要分析时改由专门 AI 页面承接。

## Data Source

- Route type: client page with live dashboard aggregate only
- Live sources: `/api/dashboard/overview`
- Local snapshot sources: none
- Downstream links: `/alerts`, `/staff/tasks`, `/staff/schedule`, `/notifications`, `/financial`, `/elderly`
- Visual scope: Microsoft Fluent 2 inspired styling at page level only; no new component-library dependency introduced

## UI States

- Loading state: dashboard aggregate 加载中时，首屏总览和 KPI 需要显示 syncing，而不是先落回本地假数。
- Empty state: aggregate 返回空队列时，工作台仍需保留框架和入口，不得渲染空白页。
- Error state: dashboard aggregate 不可达时显示 unavailable；不再用本地 workflow 或本地资源数据填回首页。
- Mobile impact: Fluent 试点下继续保持单列纵向顺序；总览、优先队列、入口卡和上下文块都不能依赖双栏或宽表才能读完。

## Health Signals

- Healthy signal: 值班用户进入页面后，能先完成“判断当班优先级 -> 进入真实处理页”的闭环，再回看来源与待接通上下文。
- Failure signal: 页面重新混入本地事故、活动、房间、物资、人员假数，或重新出现 `Local Snapshot` / `Demo Snapshot` 作为首屏事实源。
- Verification proxy: lint 与 build 通过；浏览器验证 `/operations/daily` 首屏状态标签、KPI 和优先队列。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 登录后验证 `/operations/daily` 的总览卡、KPI、优先队列、真实入口和后置上下文区；首屏不再展示本地事故/活动/资源数量

## Rollback

- Revert this delivery note and the corresponding `/operations/daily` page changes together.
- If regressions appear, fallback is the previous mixed-source workbench composition.
