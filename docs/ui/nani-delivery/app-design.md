# 护工 APP + AI 工程设计

## 范围

本工程聚焦养老护理场景中的护工执行端，围绕任务闭环、健康录入、报警响应、交接班和 AI 护理助手建设移动端原型。

当前交付范围：

- 登录与鉴权
- 首页总览
- 消息中心
- 任务中心
- 报警处理
- 报警详情时间线
- AI 护理助手
- 我的班次
- 重点长者
- 健康录入
- 护理执行
- 交接班
- 我的排班

## 文档来源

设计与实现依据来自 nursing-documents 工程中的以下文档：

- requirements/project-overview.md
- requirements/staff-collaboration.md
- requirements/health-monitoring.md
- requirements/alerts-incidents.md
- requirements/ai-operations-center.md
- platform/MODULE_PAGE_MAPPING.md
- platform/IMPLEMENTATION_BLUEPRINT.md

## 护工端目标

- 让护工在手机端优先看到到点任务和风险对象
- 把健康录入、报警处理和交接班串成同一批业务对象
- 让 AI 只提供解释与建议，不直接改写任务归属或结案结果
- 先用 Mock 数据完成工程闭环，再逐步接入真实服务

## 模块映射

| 员工模块 | 当前页面实现 | 数据来源 | AI 能力 |
| --- | --- | --- | --- |
| 登录鉴权 | LoginView + AuthMiddleware | Mock Auth | 不接入 |
| 今日任务 | HomeView / TasksView | Care Service 摘要 | 班次摘要建议 |
| 消息中心 | HomeView / NotificationsView | Notification Mock | 暂不接入 |
| 护理执行 | CareExecutionView | Care Task Mock | 动作提示与留痕建议 |
| 健康录入 | HealthEntryView | Health Draft Mock | 异常解释入口 |
| 报警处理 | AlertsView / AlertDetailView | Alert Mock | 响应动作提示 |
| 交接班 | HandoverView | Handover Mock | 交接班草稿 |
| 我的排班 | ProfileView / ScheduleView | Staff Schedule Mock | 覆盖风险提示 |
| AI 护理助手 | AiAssistantView | Prompt + Insight Mock | 摘要、解释、建议 |

## 技术设计

### 状态管理与路由

- 使用 GetX 作为路由、依赖注入和页面状态管理方案
- 所有页面统一使用 GetView 创建
- 通过 NaniBinding 注册服务与页面控制器
- 通过 AppPages 和 AppRoutes 集中维护页面入口
- 通过 AuthRequiredMiddleware 和 GuestOnlyMiddleware 保护登录前后路由边界

### 数据层

- 当前服务层使用 MockNaniService 承接首页、任务、长者、报警、交接班、排班和 AI 数据
- 认证由 AuthService 承接，当前使用内存态登录
- 数据对象拆为 ShiftOverview、NaniUser、CareTask、ResidentSnapshot、AlertCase、NotificationMessage、AlertTimelineEntry、HandoverItem、ScheduleItem、VitalDraft、AiInsight
- 后续替换真实接口时，可按对象粒度平滑替换为 Repository 或 BFF

### 页面结构

- RootView 提供五个主 Tab：首页、任务、报警、AI、我的
- 登录页独立于主 Tab，通过路由守卫控制进入
- 次级页面通过 GetPage 管理：消息中心、报警详情、重点长者、健康录入、护理执行、交接班、排班
- 页面内部统一拆成小组件：英雄卡、状态卡、任务卡、对象卡、表单卡、交接卡

### 设计系统

- 主色采用苔绿色、米白色、浅蓝色，贴合 documents 中的温暖、安心方向
- 页面背景使用柔和渐变和轻量氛围光斑，避免扁平单色背景
- 正文默认 16px 左右，按钮和卡片保持较大触控区

## UI 状态

- 加载态：当前以 Mock 数据同步返回，后续接入真实接口时补骨架屏
- 空态：通过说明卡和兜底文案维持页面完整
- 错误态：当前用局部降级和说明文案处理，不阻断主流程
- 移动端：整体按手机端优先布局构建，底部导航和主按钮适配单手操作

## 实施结论

### 已实现

- Flutter 工程骨架与 GetX 分层
- 登录页、路由鉴权和退出登录
- 护工端五个主导航页面
- 首页消息中心与独立消息页
- 报警详情时间线页
- 健康录入、护理执行、交接班、排班、重点长者次级页面
- Mock 数据驱动的任务闭环
- AI 助手的上下文提示与边界说明

### 验证门禁

- flutter pub get
- flutter analyze
- flutter test
- 主导航和次级路由可进入

### 回滚路径

- 工程为独立新建仓目录，回滚可直接移除 nursing-nani-app
- 若 AI 页面不稳定，可先隐藏 AI 助手主入口，保留任务与报警闭环
- 若后端未就绪，可继续维持 Mock 服务演示模式

## 下一步建议

1. 将 MockNaniService 替换为真实接口仓储层
2. 将 AuthService 替换为真实认证、token 与会话刷新机制
3. 为健康录入和护理执行增加提交成功后的对象级回写
4. 为报警处理增加正式责任人、处理动作提交和结案检查项
5. 将 AI 助手与 AI 运营中心的 source、focus、entity 上下文对接