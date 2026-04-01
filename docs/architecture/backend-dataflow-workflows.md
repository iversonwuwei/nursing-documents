# 后端数据流与核心 Workflow

## 交付 framing

- scope: 定义 backend 侧关键业务数据流、状态推进链路和跨端协同 workflow。
- affected audience: 后端研发、BFF 研发、前端研发、测试、AI 与通知团队。
- validation: 链路必须与当前三端功能和平台文档保持一致，并通过 `npm run docs:build`。
- rollback: 回退本文及相关索引修改。

## 设计原则

- 高风险链路统一走 workflow，不走裸 CRUD。
- 读写分离，写模型负责事实，BFF 负责场景聚合。
- 关键状态变化统一产出领域事件。

## Workflow 1: 入住评估与护理计划生成

### 目标

把 admin 的入住流程、nani 的执行流程和 family 的可见摘要打通。

### 数据流

```text
Admin 提交入住资料
-> Elder Service 创建老人档案与 AdmissionDraft
-> AI Orchestration Service 生成护理级别建议
-> Admin 确认级别
-> Care Service 生成护理计划与任务
-> Notification Service 下发任务消息
-> Nani 执行任务并回写
-> Family BFF 聚合成家属侧今日护理与状态摘要
```

### 关键状态

- AdmissionDraft
- AdmissionReviewed
- CarePlanGenerated
- CareTasksScheduled
- CareTasksExecuted

### 失败处理

- AI 建议失败: 保留人工定级入口，不阻断入住。
- 计划生成失败: AdmissionReviewed 保持成功，但 CarePlan 进入待补偿状态。
- 通知失败: 任务事实不回滚，由 Notification 重试队列补偿。

## Workflow 2: 探视预约与审批链路

### 目标

让 family 发起、admin 审批、nani 执行准备、family 接收结果成为同一条链路。

### 数据流

```text
Family 提交探视预约
-> Family BFF 校验关系与访问范围
-> Visit Service 创建 VisitRequest
-> Notification Service 通知 admin 审批
-> Admin BFF 展示待审核预约
-> Admin 审批通过或拒绝
-> Notification Service 通知 family
-> Nani BFF 获取次日探视准备清单
```

### 关键状态

- Requested
- Approved or Rejected
- CheckedIn
- Completed

### 失败处理

- 家属身份或关系校验失败: 请求直接拒绝。
- 审批通知失败: 不回滚 VisitRequest，由消息重试处理。
- 视频探视会话失败: Visit 状态保留为 Approved，视频子任务重试或改为线下探视。

## Workflow 3: 健康录入与异常响应

### 目标

让 nani 的生命体征录入、admin 的异常研判和 family 的摘要可见形成统一数据链路。

### 数据流

```text
Nani 录入生命体征
-> Health Service 写入时序与草稿事实
-> 风险规则或 AI Orchestration 识别异常
-> Operations Service 创建 AlertCase
-> Admin BFF 展示告警与长者风险卡
-> Nani 接收处理动作
-> Family BFF 只接收授权后的异常摘要或通知
```

### 关键状态

- VitalRecorded
- RiskDetected
- AlertRaised
- AlertAcknowledged
- AlertClosed

### 失败处理

- 重复录入: 通过 idempotency key 与 object version 去重。
- 规则/AI 识别延迟: 先保证指标落库，再异步产出风险判断。
- 家属通知不应直接绑定告警事实，必须经过授权规则过滤。

## Workflow 4: 报警处置与运营回放

### 目标

让设备或健康事件触发的报警，能在 admin 与 nani 之间形成可追责闭环。

### 数据流

```text
设备事件或健康事件触发报警
-> Operations Service 创建 AlertCase
-> Admin 分派责任人
-> Nani 接收报警并执行动作
-> Nani 提交处理结果
-> Admin 复核并关闭事件
-> Audit 与 Analytics 投影消费事件
```

### 关键状态

- Raised
- Dispatched
- InProgress
- Resolved
- Closed

### 失败处理

- 责任人不存在或班次失效: 回退到 Raised，重新分派。
- Nani 提交处理结果超时: 保留 InProgress，允许追加补录。
- 关闭失败: 事件不丢失，维持 Resolved 待复核。

## Workflow 5: 账单生成与家属支付同步

### 目标

让 admin 出账、family 查看和支付状态同步成为稳定的 SaaS 能力。

### 数据流

```text
Admin 生成月账单
-> Billing Service 生成 Bill
-> Notification Service 通知 family
-> Family BFF 展示账单和支付状态
-> 第三方支付或线下登记回写 Billing Service
-> Admin 查看回款与欠费列表
```

### 关键状态

- Draft
- Issued
- Paid
- Overdue

### 失败处理

- 通知失败: 账单状态不回滚。
- 支付回调失败: 进入 reconciliation queue，由 Billing 补偿对账。

## Workflow 6: AI 摘要与解释服务

### 目标

让 AI 成为三端共享能力，但不破坏业务事实边界。

### 数据流

```text
BFF 请求摘要或解释
-> AI Orchestration Service 拉取业务事实快照
-> 组装 prompt 与上下文
-> 调用模型服务
-> 保存 inference audit 与摘要快照
-> BFF 按端类型裁剪输出
```

### 关键约束

- admin 可看到内部风险解释和审计元数据。
- nani 只看到执行建议和对象上下文。
- family 只看到家属友好解释和授权摘要。

## 统一数据流治理

### 同步链路

- 登录鉴权
- BFF 聚合查询
- 关键命令提交

### 异步链路

- 通知发送
- AI 推理
- 报表投影
- 事件审计
- 工作流补偿

### 强制要求

- 所有跨服务 workflow 都要有 correlation_id。
- 所有领域事件都要带 tenant_id。
- 所有对外通知都必须可追踪到源业务对象和触发动作。