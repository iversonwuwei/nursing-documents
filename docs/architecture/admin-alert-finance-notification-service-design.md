# Admin 报警财务通知服务设计

## Scope

- scope: 设计 admin 端 Alert Service、Finance Service、Notification Service 的页面承载、模块分层与 workflow 闭环。
- boundaries: 覆盖 `alerts`、`financial`、`notifications` 页面及相关导航曝光；不在本轮新增独立 backend 微服务。
- dependencies: mock workflow store、Operations/Billing/Notification backend 契约、AI 上下文入口。
- rollback: 回退本文与对应页面改动，恢复当前三页的轻量语义。

## Entry Points

- Alert Service entry: `/alerts`
- Finance Service entry: `/financial`
- Notification Service entry: `/notifications`

## Help Surfaces

- Alert help entry: `/alerts/help`，承载告警分级口径、升级链路、AI 建议边界和班次协同说明。
- Finance help entry: `/financial/help`，承载费用构成口径、开票与催缴流程、票据归档边界和异常处理说明。
- Notification help entry: `/notifications/help`，承载模板编排原则、通道回执解释、广播触达边界和失败升级说明。
- 页面首屏只保留帮助入口卡片与最小必要摘要，不再把整套培训式说明直接铺在工作台右侧。
- 帮助页属于可选进入的二级认知面，不承担首屏主动作，不改变三条服务线当前的主工作流。

## Affected Users

- 值班主管: 关注紧急呼叫、离床预警、异常预警、SOS 处置优先级与升级链路。
- 财务专员: 关注费用计算、账单批次、欠费催缴、票据归档与回款状态。
- 运营与家属协同: 关注探视通知、定时提醒、广播公告与消息触达结果。

## Design Principles

- 页面必须先呈现服务模块，再呈现单条记录列表，避免用户只看到事件而看不到服务边界。
- workflow 要以“触发 -> 分派/生成 -> 执行/发送 -> 回执 -> 结案/归档”为统一闭环表达。
- 现有 mock 或 demo 数据要保留，但必须与目标 backend 语义对齐。
- 告警、账单、通知的状态推进都要可观察，不能只展示静态结果。
- 首屏默认不展开培训式长说明；与当前动作无直接关系的模块解释、口径和流程说明迁移到页面级帮助页。
- 后置上下文区保留系统状态、关键边界和帮助入口，不再同时承载多张长说明卡片。

## Service Design

### Alert Service

- modules:
  - 紧急呼叫: 来自呼叫器、护理站、人工协助请求。
  - 离床预警: 来自床垫、床旁设备、夜间活动异常。
  - 异常预警: 来自生命体征、设备异常、行为异常。
  - SOS 处置: 来自一键求助、高风险事件、需要升级安保或医生联动的场景。
- workflow:
  1. 事件触发。
  2. 生成告警卡片并进入优先队列。
  3. 接单并指定责任人。
  4. 联动医生/家属/安保。
  5. 结案并保留复盘建议。
- data source:
  - current: `alerts-data` + admin AI mock
  - target: Operations/Alert domain read model + dispatch timeline
- loading, empty, error, mobile:
  - loading: 优先队列和模块摘要骨架
  - empty: 当前无待处理告警，仍保留模块看板
  - error: AI 建议、状态推进或回执异常局部可见
  - mobile: 模块卡片先于列表，CTA 保持单列可点按

### Finance Service

- modules:
  - 费用计算: 服务包、耗材、附加项、减免与手工调整
  - 账单生成: 周期出账、批次确认、家属通知
  - 欠费预警: 即将逾期、已逾期、升级催缴、补偿跟踪
  - 票据管理: 发票、收据、补开、归档与对账说明
- workflow:
  1. 汇总费用项并生成应收。
  2. 财务确认账单批次。
  3. Notification Service 发送账单通知。
  4. 欠费预警与催缴升级。
  5. 回款后归档票据和异常记录。
- data source:
  - current: 长护险结算 mock + 新增机构账单运营 mock
  - target: Billing Service invoice/overdue/receipt read model
- loading, empty, error, mobile:
  - loading: 批次统计与账单列表分区骨架
  - empty: 无账单时保留费用模块与生成动作说明
  - error: 费用归集异常、通知失败、票据缺失必须可见
  - mobile: 批次看板、账单明细、票据 CTA 垂直堆叠

### Notification Service

- modules:
  - 短信/推送: 面向家属、员工、协同人员的即时通知
  - 探视通知: 预约成功、改期、拒绝、签到提醒
  - 定时提醒: 护理计划、用药、回访、催缴提醒
  - 公告广播: 面向机构、楼层、班次或全员的公告
- workflow:
  1. 选择模板、渠道与受众。
  2. 编排发送时机与升级规则。
  3. 记录发送结果与回执。
  4. 失败通知进入重试或人工补发。
  5. 广播与探视通知保留触达范围与已读结果。
- data source:
  - current: admission reminder store + 新增通知运营 mock
  - target: Notification Service dispatch/attempt/receipt read model
- loading, empty, error, mobile:
  - loading: 通道统计、发送队列与广播列表骨架
  - empty: 无提醒也应展示通知能力矩阵
  - error: 发送失败、回执延迟、升级未处理需局部显式暴露
  - mobile: 筛选、卡片、备注和动作区避免并排拥挤

## Health Signals

- healthy: 三页都能同时看到模块摘要、workflow 状态和记录列表。
- healthy: Alert/Finance/Notification 三条线都存在“待处理/处理中/已完成或已归档”的可观察状态。
- healthy: 通知和账单的下游联动失败能被显式看见，而不是只在日志里出现。
- unhealthy: 仍只剩单列表或单用途页面，无法区分服务模块。
- unhealthy: 说明类静态内容抢占首屏，用户需要先阅读多张帮助卡片才能定位主操作入口。

## Verification

- docs: `npm run docs:build`
- frontend: `npm run lint && npm run build`
- manual:
  1. 打开 `/alerts`，确认可见四类告警模块与处置 workflow。
  2. 打开 `/financial`，确认可见费用计算、账单生成、欠费预警、票据管理模块。
  3. 打开 `/notifications`，确认可见短信/推送、探视通知、定时提醒、公告广播模块。

## Phase 1.1 Finance Write Path

- scope: 仅补齐 `/financial` 页面顶部“发起评估费结算”的真实开票动作，不扩散到票据补偿、通知补发或通知页编辑动作。
- user impact: 财务专员在选中一张结算单后，可以直接把当前评定结算推送到 Billing Service 形成真实账单；资料待补充单保持禁用。
- data source: 读模型继续来自 Billing summary 和 invoice queue；写模型新增 Admin BFF -> Billing Service invoice create 调用。
- loading, empty, error, mobile:
  - loading: 开票按钮进入提交中状态，避免重复提交。
  - empty: 无选中结算单时不触发写动作，仍保留原有本地结算看板。
  - error: 开票失败时在财务页显式展示失败原因，不吞掉错误。
  - mobile: 动作反馈仍放在顶部卡片区域，避免移动端跳转到隐藏区域。
- health signals:
  - healthy: 开票成功后账单队列出现新增 `Issued` 账单或财务摘要刷新。
  - healthy: 资料待补充结算单保持禁用，不把脏数据推进到 Billing。
  - unhealthy: 用户点击后无反馈，或 Billing 失败但页面仍误报成功。
- rollback: 回退财务页按钮联动和 Admin BFF POST 代理；保留已存在的财务读模型接口。

  ## Phase 1.2 Live Read Preference And Empty States

  - scope: 收口 admin 三页的真实读模型接入细节，保证本地联调时由 Next route handler 正确转发到 Admin BFF，并把“空结果”与“接口失败”明确区分。
  - user impact: 值班、财务、通知页面在 backend 已连通但当前无记录时，会稳定展示 live 空态，不再误导用户认为系统已经回退到 demo。
  - data source:
    - current target: `src/app/api/content/[...segments]/route.ts` 统一把 alerts、finance、notifications 转发到 Admin BFF。
    - auth baseline: `src/lib/server/platform-auth.ts` 的 identity/tenant 默认端口需与本地 backend launchSettings 对齐。
    - empty-state rule: 后端返回空数组或空队列时，前端保留真实 summary 和 `Live API` 标记，只把记录区呈现为空态。
  - loading, empty, error, mobile:
    - loading: 保持当前“正在同步服务数据”文案，不因这轮变更改变页面骨架结构。
    - empty: alerts 显示“当前优先队列为空”的空态卡片；notifications 显示“真实发送队列为空”的空态卡片；financial 保留真实 summary，并用集成说明明确账单队列当前为空。
    - error: 只有代理不可达、鉴权失败或 Admin BFF 返回错误时才降级 demo，并在 API 对接状态卡片里显式展示失败原因。
    - mobile: 空态与 Live API 状态卡片保持单列堆叠，不新增需要横向滚动的组件。
  - health signals:
    - healthy: backend 可达且队列为空时，页面显示 `Live API` 而不是 `Demo Fallback`。
    - healthy: `/api/content/*` route handler 能覆盖内容管理、报警、财务、通知三组路径。
    - unhealthy: backend 已健康但页面仍因为空数组直接回退 demo，或 platform auth 因旧端口默认值持续打 warning。
  - verification:
    - docs: `npm run docs:build`
    - frontend: `npm run lint && npm run build`
    - local runtime: 启动本地 backend 后，`/alerts`、`/financial`、`/notifications` 的 API 状态卡片应在真实空队列下保持 `Live API`。
  - rollback: 回退 route handler 路由映射、platform auth 默认端口以及三页空态策略，恢复此前 demo-first 行为。
## Rollback

- 回退本文和相关 requirements/API 文档。
- 回退 admin 三个页面与导航入口改动。
- 如果只有某一页回归失败，优先局部回退该页，不影响另外两条服务线。

