# Admin 长者详情首屏 Live Profile 设计

## Scope

- Route boundary: `/elderly/[id]`
- Design target: 让长者详情首屏区分真实主档、真实健康摘要与 local snapshot 补位字段，并把机构委托主数据纳入 elder profile 真值边界
- Delivery unit: 首屏展示仍聚焦 `/elderly/[id]`，但需要依赖 `/elderly/new` 与 `/elderly/[id]/edit` 的真实写回链路保证数据闭环

## Boundaries

- Live profile boundary: `/api/admin/elders/{elderId}` 返回姓名、性别、年龄、证件号、生日、联系电话、ADL 分值、认知状态、护理等级、房间、入住状态、家属联系人、医疗提醒，以及委托类型、委托单位、月补贴、固定服务项、服务备注。
- Live health boundary: `/api/admin/elders/{elderId}/health-summary` 返回风险摘要与核心生命体征。
- Local snapshot boundary: 当前仍由前端 registry 或静态数据补位的字段，主要包括生活习惯、身高体重、部分入住台账说明等。
- AI boundary: AI 卡片仍为前端拼装或 AI 辅助结果，不声明为数据库直接输出，但其上下文应优先读取 live profile 与 live health。

## Dependencies

- Frontend proxy layer: 新增 elder detail 读取代理，沿用现有 Admin BFF 鉴权与租户透传。
- Admin BFF dependencies: Elder Service 与 Health Service；机构委托字段和编辑页必填主档字段写回继续复用 Elder Service 主档。
- Rendering strategy: 客户端页并行请求 profile 与 health，分别维护 loading、error、fallback 状态，避免单一路径失败导致整页不可用。

## Failure Modes

- Profile unavailable: 页面继续显示 local snapshot 主档，但标签必须降级为 `Profile Unavailable` 或 `Local Snapshot`。
- Health unavailable: 生命体征区显示 unavailable 或 local fallback，不能继续展示成功态的伪实时值。
- Mixed-source drift: 若 live profile 与 local snapshot 字段不一致，页面必须以 live 字段优先，并明确 local 仅用于补位。
- Entrustment drift: 若详情页看到的委托字段与工作台/编辑页不一致，应优先判定为写回链路或本地兼容快照未同步，不允许继续把 local workflow 当主真值。

## Verification

- Static verification: `npm run lint`, `npm run build`
- Runtime verification: 浏览器打开 `/elderly/E002` 或新建后的真实 elderId，检查 header 标签、基础档案、机构委托、健康摘要与 AI 卡片上下文
- Observability proxy: 页面能把 profile / health / local 三类来源稳定渲染为可观察状态标签，且委托字段来源固定标记为 live profile 或 local snapshot

## Rollback

- 移除 elder detail 代理与 live fetch，恢复到原先的本地 detail 组装逻辑。
- 保持页面结构与 AI 入口不变，确保回滚后用户导航与场景透传不受影响。