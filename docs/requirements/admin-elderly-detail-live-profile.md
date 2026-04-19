# Admin 长者详情首屏 Live Profile 收敛

## Scope

- Entry route: `/elderly/[id]`
- Affected users: 护理主管、档案管理、家属沟通与运营协同用户
- Rollout stage: 管理端长者详情首屏真实化第二阶段

## Changed Behavior

- 长者详情页首屏基础档案优先读取 Admin BFF 的长者主档，而不是继续完全依赖前端本地 `elderlyList` 与 merged registry。
- 健康信息区优先读取 Admin BFF 的健康摘要，把血压、心率、体温、血糖、血氧和风险摘要切到真实服务链路。
- 性别、年龄、证件号、联系电话、生日、ADL 分值与认知状态已纳入 Elder Service 主档；未入库字段统一显示待同步，不再保留 local snapshot 回退。
- 委托类型、委托单位、月补贴和固定服务项只读取 Elder Service 主档；若主档缺失则显示待同步，不再从 workflow 兼容快照补位。
- 状态摘要、家属沟通摘要和管理动作建议继续保留，但输入上下文只绑定 live profile 与 live health，不再回退本地 AI profile 或静态对象。

## Dependent Systems

- Admin frontend: `nursing-admin-v2` 长者详情路由与前端代理
- Admin BFF: `/api/admin/elders/{elderId}`
- Health proxy path: `/api/admin/elders/{elderId}/health-summary`
- Downstream services: Elder Service, Health Service

## Verification

- 文档门禁: `npm run docs:build`
- 前端门禁: `npm run lint` 与 `npm run build`
- 手工路径: 打开 `/elderly/E002`，验证首屏出现 `Live Profile`、`Live Health` 与 `Pending Backfill` 边界，并确认基础档案与健康摘要在 BFF 不可达时只显示可读占位，不回退本地假数据

## Rollback

- 回退本需求文档、对应 delivery 记录与前端详情页 live 读取逻辑。
- 若真实服务链路不稳定，回滚到上一版详情实现，但不扩大到其他长者页路由。
