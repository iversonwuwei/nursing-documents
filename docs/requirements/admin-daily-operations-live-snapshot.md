# Admin 日班工作台 Live Snapshot 边界收敛

## Scope

- scope: 为 nursing-admin-v2 的 `/operations/daily` 首屏收口区补齐真实数据边界，让值班工作台能够明确区分哪些指标已经来自后端服务，哪些模块仍是本地 workflow 快照。
- affected users: 值班主管、机构运营、居家协同人员、护理执行协调人员。
- changed behavior: 日班工作台的总览卡、首屏 KPI 和优先队列中的服务计划信号不再默认沿用全部本地聚合，而是优先读取 dashboard aggregate 与护理 workflow board；事故、活动、资源补位等尚未接入后端读模型的区域继续保留 local snapshot 标识。
- dependent systems: nursing-admin-v2 的 `/operations/daily`、`/api/dashboard/overview`、`/api/nursing/workflow/board`、scene 透传入口、admin delivery 文档归档。
- verification: nursing-documents `npm run docs:build`，nursing-admin-v2 `npm run lint`、`npm run build`，浏览器手工验证 `/operations/daily` 的 live 状态标签、优先队列与 scene 跳转。
- rollback: 回退本页 live snapshot 接线、状态标签文案和相关文档，使工作台恢复为原本地 workflow 聚合视图。

## 目标

- 让日班工作台能诚实回答“当前页面哪些数据已经接到服务端”。
- 把首屏最关键的风险与待办总览优先切到已落地的 BFF 聚合或护理 workflow 读链路。
- 对尚未真实化的事故、资源、活动和补位信息保留可访问性，但不再伪装成全链路 live 数据。

## 用户影响

- 值班主管进入工作台后，首屏即可区分 live aggregate、护理 workflow live 或 fallback、以及 local snapshot 三类来源。
- scene 为 `institutional` 或 `home` 时，工作台仍保持原有场景语义与 CTA，不改变入口链路。
- 当 dashboard aggregate 或护理 workflow BFF 不可达时，页面保留现有工作台结构，但显式展示 unavailable 或 fallback，而不是继续显示“看起来真实”的成功态总览。

## 真实数据边界

| 页面区域 | 数据来源 | 当前要求 |
| --- | --- | --- |
| 当班收口总览 | `/api/dashboard/overview` + `/api/nursing/workflow/board` | 必须优先使用 live 读模型，并显示状态标签 |
| 首屏 KPI | `/api/dashboard/overview` | 必须优先使用 live aggregate，不可静默退回本地假数 |
| 优先队列中的服务计划项 | `/api/nursing/workflow/board` 或 workflow fallback | 必须显示 live / fallback / demo 模式 |
| 告警、事故、活动、资源补位卡片 | 本地 alerts、operations、resource、master-data store | 允许继续使用 local snapshot，但需要显式说明未接后端读模型 |

## 验收方向

- `/operations/daily` 首屏必须能看到当前 aggregate 与 workflow 的状态标签，而不是只有业务数字。
- dashboard aggregate 成功时，首屏 KPI 与总览卡中的告警、工作流待办口径需和首页/数据看板一致。
- 护理 workflow 处于 demo 或 fallback 时，优先队列与总览信号必须清楚提示模式，不得让用户误判为真实派案数据。
- 事故、活动、资源补位区域仍需可访问，且 scene 参数继续透传，不因增加状态标签破坏工作台导航。

## 边界说明

- 本次不新增 API、数据库字段或新路由，只复用现有 `/api/dashboard/overview` 与 `/api/nursing/workflow/board`。
- 本次不改事故、活动、资源补位等本地 store 的写路径，也不把它们伪装成已持久化数据。
- 若后续需要继续真实化事故、活动或资源模块，应在新的交付单元中分别补充对应 requirement、architecture 与 delivery 文档。