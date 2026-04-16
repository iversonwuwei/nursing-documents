# Admin 日班工作台 Live Snapshot 设计

## Scope

- scope: 设计 nursing-admin-v2 `/operations/daily` 首屏的 live snapshot 接入方式、来源标识、降级策略与验证门禁。
- boundaries: 覆盖总览卡、首屏 KPI、优先队列、来源状态标签与 scene 透传；不修改事故、活动、资源补位等本地 store 契约，也不新增后端 API。
- dependencies: `src/app/operations/daily/page.tsx`、`src/lib/dashboard/admin-dashboard-api.ts`、`src/lib/mock/nursing-service-workflow.ts`、`/api/dashboard/overview`、`/api/nursing/workflow/board`。
- failure modes: dashboard aggregate 不可达但首屏继续显示成功态数字；护理 workflow 已 fallback 但页面仍宣称 live；scene 路径在新增状态标签后丢失透传；local snapshot 区域被误读为后端持久化数据。
- verification: nursing-documents `npm run docs:build`，nursing-admin-v2 `npm run lint`、`npm run build`，浏览器手工验证 `/operations/daily` 首屏状态与队列。
- rollback: 回退页面 live 读链路、状态标签、delivery 说明与 requirement/architecture 索引。

## Design Strategy

- 复用 dashboard aggregate 作为日班工作台首屏的统一 KPI 来源，避免继续从告警静态数据和本地任务派生出“看起来真实”的总数。
- 继续复用护理 workflow store 作为服务计划读入口，但页面要根据 `demo`、live 或 fallback 结果明确标注来源，而不是假设所有任务都已持久化。
- 把页面数据源拆成三层：
  - live aggregate: dashboard aggregate 返回的总览数字。
  - live or fallback workflow: 护理 workflow board 通过 BFF 读到的任务与排班信号，或 BFF 不可达时的 fallback 快照。
  - local snapshot: 事故、活动、房间、物资、人员补位等仍由本地 store 聚合的区域。
- 保持当前 scene 筛选与 CTA 行为不变，避免在数据真实化过程中破坏机构/居家入口链路。

## State Model

- `dashboardOverview`: `/api/dashboard/overview` 成功响应后的 aggregate snapshot。
- `dashboardLoading` / `dashboardError`: 首屏 aggregate 的加载和失败信号；失败时 KPI 与总览卡用 unavailable 文案替代假数。
- `nursingWorkflowMode`: 由 `isNursingWorkflowDemoMode()` 与 `nursingSnapshot.error` 派生出 `Demo`、`Live Snapshot` 或 `Fallback Active`。
- `localSnapshotSections`: 对事故、活动、资源补位等仍然来自本地 store 的模块统一打上 local snapshot 标签，形成页面级来源边界。

## Rendering Rules

- 总览卡的告警待闭环、工作流待办、通知待发送与财务动作等高频总数优先取 dashboard aggregate。
- 优先队列中的服务计划项继续使用护理 workflow 任务，但 queue 本身需要在标题或说明中展示 workflow 来源模式。
- 当 dashboard aggregate 失败时，总览卡和 KPI 不得退回原本的本地统计值，而是显示 unavailable 与错误信号。
- 当护理 workflow 处于 fallback 时，页面允许继续显示任务列表，但必须同步显示 fallback 标签与错误详情。
- local snapshot 区域继续保留原交互和跳转，但在卡片或说明文字中提示这些模块尚未接到后端读模型。

## Healthy Signals

- `/operations/daily` 首屏可同时看见 dashboard aggregate 状态、护理 workflow 状态和 local snapshot 边界。
- 首页、数据看板和日班工作台对同一批 aggregate 指标的口径保持一致。
- scene 页面从首页进入工作台后，新增状态标签不会破坏现有 CTA 与 query 透传。
- 本地 BFF 不可达时，页面继续可访问，且错误只停留在显式 signal，不出现未处理异常或整页空白。

## Evolution Rules

- 后续真实化事故、活动、资源补位时，优先为对应模块增加独立后端读模型，再替换本页 local snapshot 区域。
- 若护理 workflow store 后续补充 generatedAt 等字段，可把队列的状态标签进一步升级为时间戳快照；在此之前仅展示模式与错误。
- 新增 live 区域时，必须同步更新 requirement、architecture 和 admin delivery 归档，避免页面真实边界再次失真。

## Residual Risks

- 当前页面仍混合多个本地 workflow store，首屏真实化并不等于整页已持久化。
- 护理 workflow 在 `NEXT_PUBLIC_NURSING_WORKFLOW_MODE=demo` 下仍只代表本地演示数据；页面只能诚实暴露该状态，不能替代后端联调。
- dashboard aggregate 与 local snapshot 区域暂时存在并存口径，后续仍需继续拆解事故、活动和资源的真实读模型。