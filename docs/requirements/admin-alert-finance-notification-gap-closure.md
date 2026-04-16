# Admin 报警财务通知补齐

## Scope

- scope: 为 admin 现有系统设计补齐 Alert Service、Finance Service、Notification Service 三条服务线缺失的模块与 workflow。
- affected users: 值班主管、护理站、财务专员、机构运营、家属通知协同人员。
- changed behavior: 报警中心不再只承接统一告警列表，财务页不再只承接长护险评定结算语义，通知页不再只停留在本地提醒中心语义。
- dependent systems: Admin 前端路由、mock workflow store、backend 的 Operations/Billing/Notification 服务契约与后续 BFF 聚合。
- verification: 文档构建通过，admin lint/build 通过，三条页面入口可见新增模块与 workflow。
- rollback: 回退本页及对应设计/API/前端页面改动，恢复现有 alerts、financial、notifications 三页语义。

## Problem Statement

当前系统已经有报警页、财务页和通知页，但三条服务线都存在设计缺口：

- Alert Service 缺少“紧急呼叫、离床预警、异常预警、SOS 处置”四类模块化视角与处置闭环。
- Finance Service 缺少“费用计算、账单生成、欠费预警、票据管理”四类核心模块与从出账到回款的工作流。
- Notification Service 缺少“短信/推送、探视通知、定时提醒、公告广播”四类发送能力与从编排到回执的闭环。

## Service Matrix

| Service | Required modules | Required workflow outcome |
| --- | --- | --- |
| Alert Service | 紧急呼叫、离床预警、异常预警、SOS 处置 | 从事件触发到接单、升级、到场、结案、复盘的闭环 |
| Finance Service | 费用计算、账单生成、欠费预警、票据管理 | 从费用归集到出账、通知、回款、补偿、票据归档的闭环 |
| Notification Service | 短信/推送、探视通知、定时提醒、公告广播 | 从模板编排到发送、回执、失败补偿、广播追踪的闭环 |

## Required User Flows

### Alert Service

1. 设备、床位或健康事件触发告警。
2. 系统区分紧急呼叫、离床预警、异常预警和 SOS 事件类型。
3. 值班人员接单并指定现场责任人。
4. 必要时联动医生、家属或安保。
5. 结案时保留处置记录、升级原因和复盘动作。

### Finance Service

1. 系统按服务包、耗材、陪护或附加服务计算费用。
2. 财务生成账单并确认出账批次。
3. Notification Service 向家属或机构联系人发送账单通知。
4. 对逾期账单形成欠费预警和催缴动作。
5. 回款或异常后归档票据与补偿记录。

### Notification Service

1. 业务模块选择模板、渠道、发送对象和触发策略。
2. 系统区分短信/推送、探视通知、定时提醒和公告广播。
3. 发送后记录回执、失败原因和重试状态。
4. 高风险通知需支持升级与人工补发。
5. 广播和探视通知需保留触达范围与已读/回执结果。

## Acceptance

- Admin 中三条服务线都能看到模块入口、状态看板和 workflow 描述，而不是单一列表页。
- Alert Service 至少能区分四类告警模块，并保留从触发到结案的状态链。
- Finance Service 至少能区分费用计算、账单、欠费和票据四类模块，并展示账单闭环。
- Notification Service 至少能区分四类通知模块，并展示发送编排、回执和失败升级。
- 现有长护险、入住、护理计划等页面不应因为本次补齐而被移除。

## Constraints

- 本轮优先补齐 admin 设计与页面承载，不要求一次性完成所有 backend 实体化接口。
- AI 可提供解释、优先级建议和摘要，但不能替代报警结案、账单确认或正式通知审批。
- 若后端真实接口暂未到位，前端必须明确标示 mock 或 demo 语义。

## Phase 1.1 Finance Write Closure

- scope: 在 `/financial` 页面把“发起评估费结算”接到真实 Billing 开票写路径，其余长护险本地 workflow 保持不变。
- affected users: 财务专员、机构运营；主要影响对选中评定结算单发起真实账单的动作。
- changed behavior: 页面不再只有真实读模型和本地演示动作混用，而是允许对资料已齐备的评定结算单发起真实账单。
- dependent systems: Admin BFF `/api/admin/finance/invoices` 写代理、Billing Service `/api/billing/invoices`、财务页本地结算选择态。
- verification: `npm run docs:build`；`dotnet build nursing-backend-services.slnx && dotnet test nursing-backend-services.slnx`；`npm run lint && npm run build && CI=1 npm run test:smoke`。
- rollback: 回退财务页开票按钮联动、前端 finance service 写接口和 Admin BFF 新增 POST 代理，页面恢复为只读摘要 + 本地 workflow。

### Phase 1.1 Acceptance

- 财务页选择一个资料完整的结算单后，可发起真实账单并看到成功或失败反馈。
- 若开票成功，财务摘要或账单队列会重新拉取，确保用户能看到最新账单进入闭环。
- 若 Billing Service 不可用，页面必须显式提示失败原因，且不破坏现有本地结算视图。
- 资料待补充的结算单不得直接触发真实开票，避免把未过门禁的数据推进到 Billing。

## Phase 1.2 Live Read Preference And Empty State Closure

- scope: 让 `/alerts`、`/financial`、`/notifications` 在本地 backend 可达时优先保留真实读模型，即使返回空队列也展示 live 空态而不是自动回退 demo；同时把 admin 平台认证默认端口与 Next 代理路由对齐当前本地 backend 基线。
- affected users: 值班主管、财务专员、通知协同人员，以及本地联调 admin 团队。
- changed behavior: 页面不再把“真实接口已连通但当前无记录”误判为失败；`/api/content/*` 会真正转发报警、财务、通知路径；开启 platform auth 时也不再因为旧端口默认值直接掉回 demo 身份。
- dependent systems: Admin Next route handler `/api/content/[...segments]`、platform auth 到 identity/tenant 的默认端口、Admin BFF 的 alerts/finance/notifications 读写代理、本地 backend 端口基线。
- verification: `npm run docs:build`；`npm run lint && npm run build`；本地 backend 运行时打开三页应显示 Live API，空队列时展示空态而不是 Demo Fallback。
- rollback: 回退本节文档及 admin 代理/页面改动，恢复为当前“接口失败或空队列都降级 demo”的行为。

### Phase 1.2 Acceptance

- 当 Admin BFF 可达但报警、财务或通知当前没有记录时，页面仍显示 `Live API`，并展示明确空态说明。
- `/api/content/alerts/*`、`/api/content/finance/*`、`/api/content/notifications/*` 必须能正确转发到 Admin BFF，而不是统一返回未知路径。
- 开启 platform auth 时，identity/tenant 默认端口要与当前本地 backend 基线一致，避免因为陈旧的 `5301/5302` 默认值触发不必要的 demo fallback。
- 若真实接口不可达，页面仍需显式展示失败原因，并按当前设计回退到 demo 视图作为兜底。

