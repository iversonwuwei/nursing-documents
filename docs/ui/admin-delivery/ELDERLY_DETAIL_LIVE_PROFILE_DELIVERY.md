# Elderly Detail Live Profile Delivery Unit

## Scope

- Entry route: src/app/elderly/[id]/page.tsx
- Affected users: 护理主管、档案管理、家属沟通与运营协同用户
- Rollout stage: 第二十三批长者详情首屏真实化说明

## User Impact

- 长者详情页首屏基础档案和健康摘要开始优先读取 BFF 聚合后的真实主档与健康摘要，不再把本地静态对象当成默认真值。
- 机构委托类型、委托单位、月补贴和固定服务项目开始直接读取 elder profile 主档，不再只依赖本地 workflow 快照。
- 性别、年龄、证件号、联系电话、生日、ADL 分值与认知状态也开始直接读取 elder profile 主档，保证 live-profile-only 老人进入编辑页后可真实回填并持久化保存。
- 页面继续保留 AI 状态摘要、家属端摘要草稿和管理动作建议，但它们的上下文会优先绑定 live profile 与 live health。
- 当前后端契约未提供的剩余字段仍保留 local snapshot，并通过标签和来源说明显式暴露。

## Data Source

- Route type: client detail route with live profile, live health, and local snapshot fallback
- Live sources: `/api/elders/{elderId}`, `/api/elders/{elderId}/health-summary`
- Local snapshot sources: `elderly-registry`, `assessment-workflow` 兼容快照、本地 AI fallback、静态补位字段
- Downstream links: `/elderly`, `/elderly/face`, `/ai-assistant`

## UI States

- Loading state: profile 与 health 分别显示 syncing，整页不进入空白骨架阻塞。
- Empty state: live 数据缺失时仍保留详情结构和 AI 入口，避免直接回退默认对象造成误读。
- Error state: profile / health 任一路径失败都需要局部可见状态标签，不能继续展示成功态 live 文案。
- Mobile impact: 新增来源标签后保持详情卡片、AI 卡片和双栏布局在窄屏下可读。

## Health Signals

- Healthy signal: 页面能稳定显示 `Live Profile`、`Live Health`、`Local Snapshot` 边界，且年龄、性别、证件号、联系电话与编辑页回填口径一致。
- Healthy signal: 页面能稳定显示机构委托字段，且与编辑页和委托工作台口径一致。
- Failure signal: live 读取失败后页面仍展示未标注来源的伪实时字段，或 AI 上下文仍指向旧本地对象。
- Verification proxy: lint 与 build 通过；浏览器验证 `/elderly/E002` 详情页来源标签与 live 字段。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 登录后验证 `/elderly/E002` 或真实新建 elderId 的 header、基础档案、机构委托、健康信息和 AI 卡片来源说明

## Rollback

- Revert this delivery note together with the elder proxy and detail page changes.
- If live reads regress, fallback is the previous local detail composition with explicit removal of live-source labeling.