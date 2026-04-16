# Admin Create Flows Solution

## Scope

- Source repo: nursing-admin-v2
- Affected modules: staff, equipment, supplies, elderly health archive, elderly vitals, elderly visits
- Affected users: 人力运营、设备管理员、采购与后勤、护理主管、医护班组、家属沟通协同用户
- Rollout stage: 第十六批 admin 新建能力统一治理方案

## Problem Statement

- admin 里已有多个列表页暴露“新增”或“录入”入口，但只有老人、机构、房间、活动、事件几组真正完成了从表单到列表回流的闭环。
- 剩余模块大多停留在按钮层，导致用户可以看到入口，却无法留下可追踪的数据状态，也无法验证新建行为是否健康。
- 如果继续按页面各自堆本地数组，会进一步放大统计口径不一致、详情页接不住新记录、以及 `useSyncExternalStore` 快照不稳定的风险。

## Shared Design

- 统一把剩余新建能力分成两个 shared mock workflow store：资源域和健康服务域。
- 资源域覆盖 staff、equipment、supplies，负责主数据录入、待确认状态和入册动作。
- 健康服务域覆盖 health archive、vitals、visits，负责健康建档、体征录入和探视预约的回流状态。
- 每条新建链路统一采用同一模式：列表 CTA -> `/new` 表单页 -> shared store 落库 -> 跳回列表页并带 `selected` + `entry` 查询参数 -> 列表页展示来源提示卡 -> 需要时执行人工确认动作。
- 所有 shared store 必须返回引用稳定的快照对象，避免再次出现 `getSnapshot should be cached` 和最大更新深度问题。

## Module Matrix

| Module | Entry route | New route | Shared store | First lifecycle | List confirmation |
| --- | --- | --- | --- | --- | --- |
| Staff | `/staff` | `/staff/new` | resource workflow | 待入职 | 确认入职 |
| Equipment | `/equipment` | `/equipment/new` | resource workflow | 待验收 | 完成验收 |
| Supplies | `/supplies` | `/supplies/new` | resource workflow | 待上架 | 确认上架 |
| Health Archive | `/elderly/health` | `/elderly/health/new` | care service workflow | 待建档 | 完成建档 |
| Vitals | `/elderly/vitals` | `/elderly/vitals/new` | care service workflow | 已录入 | 无需二次确认 |
| Visits | `/elderly/visits` | `/elderly/visits/new` | care service workflow | 待审核 | 通过预约 |

## User Impact

- 人力、设备、采购和护理相关页面不再只有“会点击但没结果”的按钮，而是能留下可见状态并回流列表。
- 运营与护理主管可以在列表页完成最小人工确认动作，避免新记录直接混入稳定台账。
- 对已有只读 detail 路由，优先保证新建记录至少能在列表页稳定展示；若 detail 已存在，则应能识别新建对象而不是回退到错误默认值。

## Data And Contract Strategy

- 当前仍为 admin 前端 demo，不新增真实 API，不改现有后端契约。
- shared store 使用 localStorage 持久化，保留可回滚、可观察和可复现的演示特性。
- 列表页 KPI 与 AI 摘要必须统一消费合并后的 live data，避免新建记录存在但统计口径遗漏。
- 详情页如果接入 shared store，必须优先查 live record，再回退旧静态 mock。

## UI States

- Loading state: 提交按钮进入 loading，避免重复提交。
- Empty state: 列表筛选无结果时保持原有空态；新建页默认以最小可提交字段集初始化。
- Error state: 所有新建页都需提供显式表单级错误提示，不把错误吞掉在按钮动作里。
- Mobile impact: 新建页说明卡、表单网格和底部 CTA 在窄屏下必须退化成单列。

## Health Signals

- Healthy signal: 新建后能回到对应列表，看到同一对象、对应生命周期标签和人工确认入口。
- Healthy signal: 新建或导入页主区优先保留表单与提交动作，边界说明和帮助入口后置后，仍不影响回流闭环表达。
- Failure signal: 表单提交后对象丢失、统计口径未更新、详情页接不住新对象，或 shared store 触发无限更新。
- Observable signal: 列表提示卡、生命周期标签、稳定快照 smoke 断言、以及 build 可复现路由输出。

## Verification

- nursing-admin-v2 minimum gate: `npm run lint`
- nursing-admin-v2 required gate for this batch: `npm run lint` and `npm run build`
- Recommended regression gate: extend Playwright smoke with至少一条资源域和一条健康服务域新建链路
- nursing-documents gate: `npm run docs:build`

## Rollback

- 回滚顺序：先回退新增 `/new` 路由和 shared store 接入，再回退列表页 CTA 与提示卡，最后回退本方案文档和 route delivery notes。
- 若只发生局部问题，可按域回滚：资源域回退不影响健康服务域，健康服务域回退不影响已落地的老人/机构/房间/活动/事件闭环。
