# Admin 共享页面场景上下文设计

## Scope

- scope: 设计 nursing-admin-v2 共享页面的 scene 参数协议、默认筛选策略、入口拼装方式与验证门禁。
- boundaries: 覆盖共享页面内部展示逻辑、入口链接拼装、AI 追踪上下文透传、首页与详情 CTA 保留 scene，以及 navbar 激活态；不修改后端接口与 mock 数据结构。
- dependencies: `src/lib/care-scenes.ts`、`TopNavbar.tsx`、`NursingWorkflowPages.tsx`、共享页本地筛选逻辑、Playwright scene smoke。
- failure modes: 带 query 的导航失去激活态、共享页只改标题未改默认数据、机构/居家入口落到相同数据视图、长护险业务入口被错误绑定到单一场景、scene 页面跳入 AI 后上下文丢失、首页或详情页 CTA 在跳转时丢失 scene、本地 BFF 模式下 identity / 下游拒连导致 `/api/nursing/workflow/board` 或 `/api/ai/*` 直接抛错并污染浏览器控制台。
- verification: nursing-documents `npm run docs:build`，admin `npm run lint`、`npm run build`、聚焦 scene smoke。
- rollback: 回退 `care-scenes.ts`、入口链接、共享页 scene 逻辑和 smoke 用例。

## Design Strategy

- 使用 `scene=institutional|home` 作为共享页显式上下文，不新增重复路由。
- 通过 `src/lib/care-scenes.ts` 统一承载场景解析、来源匹配与带参链接拼装，避免每个页面重复写字符串分支。
- 让 `长护险业务` 菜单继续保留无 scene 的通用入口，而 `机构养老`、`居家养老` 与 `/nursing/services` 场景卡片负责携带上下文。
- 页面内优先复用现有来源字段做筛选：认定页看 `sourceType`，任务/排期看 `employmentSource`，结算页回溯评定来源，机构协同页只调整默认聚焦与跳转。
- 服务打卡页沿用排期里的责任人来源映射做分流；AI 助手入口在 tracking context 中附带 scene，所有 AI 子页继续透传。
- 当运行在 BFF 模式且本地 identity / 下游不可达时，Next 代理层返回结构化 503 细节，页面侧把 read-path 回退到 demo/mock 快照；这样既保留“当前已降级”的可观测信号，也避免把首屏渲染升级成未处理异常。

## State Model

- `CareScene`: 仅允许 `institutional` 与 `home` 两个值，其余值按无 scene 处理。
- `matchesAdmissionScene`: 用于认定页、任务页、结算页按手工建档/资料导入分流。
- `matchesEmploymentScene`: 用于任务页、排期页按自有团队/第三方合作分流。
- `withSceneQuery`: 统一拼装场景入口与带 partner 等附加查询参数的跳转。
- `AiTrackingContext.scene`: 统一承载 AI 助手里的机构/居家上下文，保证子页切换时不丢 scene。

## Entry Rules

- 顶部导航中的场景页入口必须带 scene 参数，避免用户进入后再手工切换视角。
- `/nursing/services` 的机构养老与居家养老 workflow 步骤、模块卡片也必须带 scene 参数，保证总览页和导航一致。
- 长护险业务菜单保持无 scene 的完整主链路入口，用于跨场景运营与监管视图。
- 共享页里的返回任务、排期、打卡、报表和 AI 入口在已有 scene 时也必须继续带 scene，不能在链路中途回退到通用视图。
- 首页场景入口和日班工作台属于 scene 链路的上游入口，只要当前页面已带 scene，后续 CTA 也必须继续带 scene。

## Healthy Signals

- 共享页标题、摘要和默认数据列表在同一条路由下能明确区分机构与居家视角。
- scene 页面进入后，顶部导航一级菜单仍能保持激活态。
- scene 缺失时，共享页继续提供长护险通用视图，不影响既有主链路 smoke。
- scene 页面进入 AI 助手后，Tracked Context 和子页导航继续显示并透传场景标签。
- scene 页面从首页进入日班工作台、再进入老人详情或共享页时，URL 查询参数仍然连续保留。
- 本地 BFF 降级期间，scene 链路页仍能渲染既有 mock 数据，错误只出现在页面显式 signal 中，而不会表现为未捕获 rejection 或整页空白。

## Verification Strategy

- 文档侧通过 `npm run docs:build` 验证站点结构与索引。
- 前端侧通过 `npm run lint`、`npm run build` 验证类型、路由和 SSR 构建正常。
- 浏览器侧至少验证三类信号：scene 入口标题变化、默认筛选语义变化、navbar 在 query 路由上的激活态。
- 若本地未启动 identity / BFF，还需验证 workflow board 和 AI 总览请求会降级到 demo/mock 呈现，控制台不再出现未处理异常。

## Evolution Rules

- 后续如继续扩展共享页，优先沿用 scene 参数而不是复制新二级导航项。
- 若某页已无法用现有来源字段稳定区分场景，应先补文档说明再决定是否升级为独立契约。
- 新增 scene 页面时，必须同步补入口链接和 smoke，用验证门禁防止“只改页面不改入口”或“只改入口不改页面”。

## Residual Risks

- 当前场景分流仍依赖 mock 数据已有字段，若未来真实接口的来源字段不同，需要补一层 BFF 适配。
- 机构协同页当前仍展示评估机构与护理服务机构两类内容，scene 只调整聚焦顺序和跳转，不代表已经拆成完全独立的信息架构。
- 本地 BFF 降级当前只保证读路径稳定可见；若开发者在降级期间继续执行真实写操作，仍会收到显式失败，需要在依赖恢复后重试。
