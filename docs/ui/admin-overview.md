# 管理端页面说明

## 页面范围

当前护理项目管理端建议至少覆盖以下页面组：

- 首页 Dashboard
- 长者管理与详情
- 健康监测
- 报警中心与事故管理
- 员工列表、任务中心、排班管理
- 房间、机构管理
- 设备列表、设备监控、设备详情
- 物资列表与补货跟进
- 财务与报表中心
- AI 运营中心

## 交互原则

- 列表页优先解决筛选、搜索、排序和跳详情。
- 详情页优先展示状态、摘要、关键动作和历史记录。
- 页面默认按纵向单列堆叠，交互内容先出现，静态说明、数据源边界和只读摘要后出现。
- 共享视觉系统以 Microsoft Fluent 2 为参考，统一采用中性浅层表面、清晰焦点环、克制的语义色和企业工作台控件层级，而不是装饰型营销化卡片风格。
- 桌面端不再默认使用左右分栏；桌面与移动保持同一纵向阅读顺序，避免用户在不同设备上切换心智模型。
- 静态说明默认先做删减；如果不能直接帮助用户完成当前一步操作，就不应占据首屏。
- 需要长期保留的说明迁到帮助页或培训页，页面内部只保留简短摘要和入口。
- 共享骨架优先统一这条规则：标准页通过 `StandardModulePage` 自动继承“主区在前、上下文在后”和帮助入口，自定义页优先复用共享纵向骨架，而不是继续保留左右双栏说明区。
- 首页、AI 入口、日班工作台这类高频总览页默认只保留优先级摘要和分流入口；任务表、健康明细、问答面板等具体执行逻辑应拆到独立页面。
- 高风险动作必须二次确认。
- AI 只做解释、建议、摘要和排序，不直接越权执行。

## 最近补充的布局约束

- [Admin 交互静态分栏原则](/requirements/admin-interaction-static-layout)
- [Admin 交互纵向堆叠布局设计](/architecture/admin-interaction-static-layout)
- 第二批高频页统一对象：首页、机构管理、员工列表、日班工作台、AI 运营入口都必须遵循同一主工作区在前 / 上下文区在后 / 帮助页模式。
- 其中首页进一步收敛为“运营优先级首页”，不再直接承载护理任务明细表和健康明细卡；这些内容分别回到 `/staff/tasks`、`/health-monitoring` 与老人详情页处理。

## 文档补充方向

- 为每个页面补页面目标、数据来源、加载态、空态、错误态。
- 为每个 AI 卡片补输入、输出和人工确认边界。
- 为移动端或协同端页面补适配说明。

## 已拆分页面文档

- [长者管理页面](/ui/elderly-management)
- [健康监测页面](/ui/health-monitoring)
- [报警与事件页面](/ui/alerts-incidents)
- [员工协同页面](/ui/staff-collaboration)
- [机构管理页面](/ui/organization-management)
- [房间与床位页面](/ui/room-management)
- [设备管理页面](/ui/equipment-management)
- [物资管理页面](/ui/supply-management)
- [AI 运营中心页面](/ui/ai-operations-center)

## 已补充页面级文档

- [Dashboard 首页](/ui/page-dashboard)
- [长者详情页](/ui/page-elderly-detail)
- [健康监测页](/ui/page-health-monitoring)
- [报警中心页](/ui/page-alerts-center)
- [事故详情页](/ui/page-incident-detail)
- [排班管理页](/ui/page-staff-schedule)
- [机构列表页](/ui/page-organizations-overview)
- [房间列表页](/ui/page-rooms-overview)
- [设备详情页](/ui/page-equipment-detail)
- [物资列表页](/ui/page-supplies-overview)
- [AI 日志审计页](/ui/page-ai-logs)

## 已补充流程级文档

- [健康监测页交互流程](/ui/flow-health-monitoring)
- [报警中心页交互流程](/ui/flow-alerts-center)
- [排班管理页交互流程](/ui/flow-staff-schedule)
- [设备详情页交互流程](/ui/flow-equipment-detail)
- [AI 日志审计页交互流程](/ui/flow-ai-logs)

## 已补充测试与验收文档

- [健康监测页测试与验收](/ui/test-health-monitoring)
- [报警中心页测试与验收](/ui/test-alerts-center)
- [排班管理页测试与验收](/ui/test-staff-schedule)
- [设备详情页测试与验收](/ui/test-equipment-detail)
- [AI 日志审计页测试与验收](/ui/test-ai-logs)
- [Dashboard 首页测试与验收](/ui/test-dashboard)
- [长者详情页测试与验收](/ui/test-elderly-detail)
- [机构列表页测试与验收](/ui/test-organizations-overview)
- [房间列表页测试与验收](/ui/test-rooms-overview)
- [物资列表页测试与验收](/ui/test-supplies-overview)

## 已补充实施级文档

- [Dashboard 首页实施说明](/ui/implementation-dashboard)
- [长者详情页实施说明](/ui/implementation-elderly-detail)
- [健康监测页实施说明](/ui/implementation-health-monitoring)
- [报警中心页实施说明](/ui/implementation-alerts-center)
- [设备详情页实施说明](/ui/implementation-equipment-detail)
- [房间列表页实施说明](/ui/implementation-rooms-overview)
- [物资列表页实施说明](/ui/implementation-supplies-overview)
- [AI 日志审计页实施说明](/ui/implementation-ai-logs)
