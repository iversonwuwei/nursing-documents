# Admin 交互静态分栏原则

## Scope

- scope: 统一 admin 端“交互内容在左、静态内容在右”的页面布局原则，并把该原则下沉到共享页骨架，覆盖所有复用 `InteractionRailLayout`、`StandardModulePage`、`dashboard-grid-2` 的页面。
- affected users: 值班主管、财务专员、运营专员，以及所有需要在 admin 端高频查看状态并执行动作的后台角色。
- rollout stage: Phase 1 先完成 `/alerts`、`/financial`、`/notifications` 三个服务型页面；Phase 2 把同一套规则下沉到标准页和共享双栏样式，让更多页面自动继承；Phase 3 收口高频工作台与经营总览页，包括 `/`、`/organizations`、`/staff`、`/operations/daily`、`/ai-assistant`。
- dependent systems: `nursing-admin-v2` 页面壳层、设计系统卡片组件、页面级加载/空态/错误态表达。
- rollback: 回退共享布局组件、标准页组件和全局双栏样式定义，恢复当前等宽双栏或顺序排布。

## Changed Behavior

- changed behavior: 可执行内容优先留在左侧主工作区，包括筛选、搜索、列表、详情、编辑备注、状态推进和主操作按钮。
- changed behavior: 静态或低交互内容集中到右侧信息轨，包括 API 对接状态、服务模块说明、workflow 解释、口径说明、健康信号和只读摘要。
- changed behavior: 当页面采用左右布局时，主工作区与信息轨按 7:3 分配宽度；当屏幕不足以稳定承载双栏时，自动改为单列，并保持交互内容先于静态内容。
- changed behavior: 页面默认不再保留大段说明型静态内容；如果说明不会直接影响当前一步操作，应从首屏移除。
- changed behavior: 仍需长期保存的模块边界、workflow 解释、异常口径和操作说明，迁移到页面级帮助入口，而不是继续堆在右侧信息轨。
- changed behavior: 高频工作台与经营总览页也必须遵循同样原则，首页、机构、员工、运营日班、AI 总览不再把路径说明、AI 解释和培训式文案与主任务混排在首屏主工作区。

## Rules

- 右侧信息轨不承载页面唯一主 CTA，不放必须立即点击才能完成任务的关键动作。
- 与当前任务直接相关的 loading、empty、error 状态放在左侧主工作区，避免用户在右侧说明区寻找下一步。
- 右侧信息轨优先承载“为什么这样做”和“当前系统状态是什么”，左侧主工作区优先承载“我要做什么”和“我现在能做什么”。
- 同一页面内若存在可编辑列表和只读说明，不再使用等宽双栏混排，避免用户把说明卡片误判为主工作流。
- 右侧信息轨默认只保留极少量必要上下文，例如 API 对接状态、风险边界、简短帮助入口；长 workflow、模块说明和培训性文案不默认展开。
- 如果说明内容不能帮助用户在当前 30 秒内做出下一步动作，就不应占据首屏卡片位；优先删除，其次迁移到帮助页。
- 帮助页是说明类内容的主承载面，页面内只保留简短摘要和跳转入口，不再把整套操作手册嵌入工作台。
- 共享页骨架必须优先承载这一规则，而不是要求每个页面单独手写一次；标准页帮助入口统一通过通用帮助路由承接。

## Loading Empty Error Mobile

- loading: 主工作区保持任务相关骨架屏或状态文案；右侧信息轨允许显示静态说明或数据源状态占位。
- empty: 空结果若会影响用户下一步动作，放在左侧主工作区；右侧只保留固定能力说明和范围边界。
- error: 接口失败、鉴权失败、写入失败等与操作结果直接相关的错误必须在左侧主工作区显式出现；右侧只补充数据源和回滚说明。
- mobile: 小屏下改为单列顺序，先主工作区后信息轨，不能为了保留 7:3 而强行挤压成不可读双栏。

## Verification

- docs: `npm run docs:build`
- frontend: `npm run lint && npm run build`
- manual:
  1. 打开 `/alerts`、`/financial`、`/notifications`，确认服务型页面的主工作区和信息轨保持 7:3，长说明通过帮助页承接。
  2. 打开 `/settings`、`/settings/roles`、`/devices/assets`、`/alerts/history`，确认标准页默认也使用主内容区 + 右侧信息轨，并出现帮助入口。
  3. 打开任一标准页对应的 `/help/...` 路由，确认完整说明、关键能力和流程解释已迁入帮助页。
  4. 抽查复用 `dashboard-grid-2` 的自定义页，确认桌面端默认不再是等宽双栏。
  5. 打开 `/`、`/organizations`、`/staff`、`/operations/daily`、`/ai-assistant`，确认主工作区优先承载任务、列表、总览和主要 CTA，右侧只保留状态、路径、边界和帮助入口。

## Rollback

- 回退 `nursing-admin-v2` 的共享分栏组件、标准页组件和全局双栏样式调整。
- 保留原有业务读写逻辑与 API 层，不做数据层回退。
- 回退本文及其在索引、导航中的入口。