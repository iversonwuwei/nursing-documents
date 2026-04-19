# Elderly Detail Live Profile Delivery Unit

## Scope

- Entry route: src/app/elderly/[id]/page.tsx
- Affected users: 护理主管、档案管理、家属沟通与运营协同用户
- Rollout stage: 第二十三批长者详情首屏真实化说明

## User Impact

- 长者详情页首屏基础档案、机构委托与健康摘要只读取 BFF 聚合后的真实主档与健康摘要，不再把本地静态对象或 workflow 快照当成默认真值。
- 机构委托类型、委托单位、月补贴和固定服务项目只读取 elder profile 主档；主档未补齐时统一显示 `待同步` / `Pending Backfill`。
- 性别、年龄、证件号、联系电话、生日、ADL 分值与认知状态全部直接读取 elder profile 主档，缺失字段不再回退到本地 snapshot。
- 页面继续保留状态摘要、家属沟通摘要和管理跟进建议，但它们只消费 live profile 与 live health，不再内嵌本地 AI profile fallback。
- 当前后端契约未提供的字段保持可读占位，不再通过前端 mock/localSnap 伪造对象事实。

## Data Source

- Route type: client detail route with live profile and live health only
- Live sources: `/api/elders/{elderId}`, `/api/elders/{elderId}/health-summary`
- Local snapshot sources: none
- Downstream links: `/elderly`, `/elderly/face`, `/ai-assistant`

## UI States

- Loading state: profile 与 health 分别显示 syncing，整页不进入空白骨架阻塞。
- Empty state: live 数据缺失时仍保留详情结构和上下文入口，但字段统一显示 `待同步`，不再回退默认对象造成误读。
- Error state: profile / health 任一路径失败都需要局部可见状态标签，不能继续展示成功态 live 文案，也不能回退本地假数据。
- Mobile impact: 新增来源标签后保持详情卡片、AI 卡片和双栏布局在窄屏下可读。

## Health Signals

- Healthy signal: 页面能稳定显示 `Live Profile`、`Live Health`、`Pending Backfill` 边界，且年龄、性别、证件号、联系电话与编辑页回填口径一致。
- Healthy signal: 页面能稳定显示机构委托字段，且与编辑页和委托工作台口径一致。
- Failure signal: live 读取失败后页面仍展示未标注来源的伪实时字段，或详情页重新出现 `Local Snapshot` / workflow 补位标签。
- Verification proxy: lint 与 build 通过；浏览器验证 `/elderly/E002` 详情页来源标签与 live 字段。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 登录后验证 `/elderly/E002` 或真实新建 elderId 的 header、基础档案、机构委托、健康信息与摘要卡片都只展示 live 字段或待同步占位

## Rollback

- Revert this delivery note together with the elder proxy and detail page changes.
- If live reads regress, fallback is the previous detail composition, but this batch不保留任何局部回退逻辑在当前实现中。
