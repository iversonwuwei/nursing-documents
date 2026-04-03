# Care Workflow Realization For Admin

## Scope

- Entry points: `NursingBackend.Services.Care`, `NursingBackend.Bff.Admin`, `nursing-admin-v2` 的套餐、计划、任务、排班四个入口页与对应 route handlers
- Affected users: 集团运营、机构运营、护理主管、排班协同人员
- Rollout stage: 第二十批 Admin 护理工作流真实化竖切

## Boundaries

- Care service 负责服务套餐、服务计划、执行任务、排班分派读模型、审计记录与运行指标。
- Admin BFF 负责把 Care service 暴露为 admin 可直接消费的聚合契约，不让前端直接依赖领域服务地址。
- Admin frontend 只通过 Next route handlers 访问 Admin BFF，并在页面层保留加载态、空态、错误态和显式回滚面。
- 本批不接入真实 Staffing service、HR 主数据、计费服务或 Feature Flag 平台；责任人和排班分派仍以护理工作流内的受控数据为准。

## User Impact

- `/nursing/packages` 不再只依赖浏览器 localStorage，而是通过真实后端契约创建、定价、发布和下线套餐。
- `/nursing/plans` 将通过后端完成计划创建、主管复核、异常插单与归档，并把这些动作落到审计记录中。
- `/staff/tasks` 将消费真实任务执行状态、处理备注和闭环时间，而不是仅依赖前端推导状态。
- `/staff/schedule` 将从“负荷摘要”升级为真实的按日期与班次展开的分派板，明确展示每个员工每天承接了哪些计划。

## Contract Shape

### Care service

- Read model: 一个工作流聚合查询，返回 `packages`、`plans`、`tasks`、`schedule`、`observability`。
- Write model:
  - 创建套餐草稿
  - 套餐动作: 提交定价、完成定价、发布、下线
  - 创建计划草稿 / 从套餐生成计划
  - 计划动作: 复核、转异常、归档
  - 任务动作: 开始执行、完成执行、保存备注
- Observability:
  - counter: 任务完成次数
  - counter: 计划归档次数
  - gauge: 当前待分派 backlog
  - persistent audit: 套餐、计划、任务动作审计

### Admin BFF

- 对前端暴露稳定的 admin 合同路径，不泄露 Care service 的真实地址。
- 负责透传租户、用户、关联 ID，并把下游失败收敛为 admin 可消费的问题响应。

### Admin frontend

- 使用 Next route handlers 封装后端 dev token 获取与 BFF 访问。
- 页面只消费本地 `/api/nursing/workflow/*` 路径，避免在 client component 暴露内部服务地址。

## Data Model

- `ServicePackageEntity`: 套餐主记录
- `ServicePlanEntity`: 服务计划主记录
- `ServicePlanTaskExecutionEntity`: 计划级执行状态与操作备注
- `ServicePlanAssignmentEntity`: 真实排班分派记录，至少包含日期、班次、责任人、计划与老人信息
- `CareWorkflowAuditEntity`: 套餐、计划、任务动作审计

## Loading, Empty, Error, Mobile States

- Loading: 页面初次加载和动作提交时要显式展示“正在同步工作流数据”或“正在提交动作”。
- Empty: 当租户尚无套餐或计划时，页面继续展示当前 CTA，而不是回退成假数据。
- Error: 后端不可用、鉴权失败、动作校验失败时，页面保留卡片级错误提示，并避免 silent failure。
- Mobile: 套餐和计划页保持单列表单优先；任务与排班页在窄屏下优先保留主要摘要和动作，不强行展示完整矩阵。

## Verification

- Documentation gate: `cd nursing-documents && npm run docs:build`
- Backend gate: `dotnet build nursing-backend-services.slnx` 和 `dotnet test nursing-backend-services.slnx`
- Frontend gate: `npm run lint` 和 `npm run build`
- Runtime path:
  - 套餐草稿 -> 定价 -> 发布 -> 从套餐生成计划
  - 计划复核 -> 任务开始/完成 -> 审计与指标变化
  - 排班页看到真实日期/班次/责任人分派，而不只是覆盖汇总

## Rollback

- 回滚代码变更后，admin 页面恢复到现有 mock workflow store。
- 若仅后端竖切出问题，可先停用 admin route handlers 对真实 BFF 的访问，改回本地 mock 数据，不影响现有 demo 展示。
- 数据库回滚路径为撤回本批 Care migration；若已写入演示数据，允许直接清空本批新增表后重新迁移。

## Residual Risks

- 当前未接 Staffing service，因此责任人和分派板仍属于 Care workflow 内的受控模型，不代表最终组织排班主数据。
- 本批 gauge 以应用实例内实时刷新值为主，若未来多实例部署，需要切换为可聚合的外部指标来源或集中计算。
- demo 登录仍通过 dev token 桥接，不等于生产登录流；后续接真实 IAM 时要替换 Next route handlers 的 token 获取逻辑。