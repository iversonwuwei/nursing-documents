# Admin 报警财务通知服务契约

## Scope

- scope: 定义 Alert Service、Finance Service、Notification Service 在 admin 端补齐模块后的建议接口与字段边界。
- compatibility: 以新增读模型和动作接口为主，不破坏现有 Billing/Notification 契约。
- caller impact: Admin 前端、后续 Admin BFF、Operations/Billing/Notification 服务。
- verification: 文档构建通过，字段与现有 backend 语义不冲突。
- rollback: 回退本文并保持当前简化 API 草案。

## Alert Service

### Query alert modules summary

- method: `GET`
- path: `/api/alerts/summary`
- purpose: 返回紧急呼叫、离床预警、异常预警、SOS 处置四类模块的待处理、处理中、已结案统计。

### Query alert queue

- method: `GET`
- path: `/api/alerts`
- filters: `module`, `level`, `status`, `assignee`, `keyword`
- purpose: 返回告警列表与优先级字段。

### Submit alert action

- method: `POST`
- path: `/api/alerts/{alertId}/actions`
- actions: `acknowledge`, `dispatch`, `escalate`, `arrive`, `resolve`, `close`
- purpose: 以 append-only 方式记录告警动作。

### Query alert timeline

- method: `GET`
- path: `/api/alerts/{alertId}/timeline`
- purpose: 返回触发、接单、升级、到场、结案全链路。

## Finance Service

### Query finance modules summary

- method: `GET`
- path: `/api/finance/summary`
- purpose: 返回费用计算、账单生成、欠费预警、票据管理四类模块统计。

### Query billing batches

- method: `GET`
- path: `/api/finance/billing-batches`
- purpose: 返回账单批次、出账状态、通知状态与回款状态。

### Query overdue queue

- method: `GET`
- path: `/api/finance/overdue`
- purpose: 返回即将逾期和已逾期账单列表。

### Query receipts

- method: `GET`
- path: `/api/finance/receipts`
- purpose: 返回发票、收据、补开和归档状态。

### Confirm bill generation

- method: `POST`
- path: `/api/finance/billing-batches/{batchId}/issue`
- purpose: 确认出账并触发通知编排。

## Notification Service

### Query notification modules summary

- method: `GET`
- path: `/api/notifications/summary`
- purpose: 返回短信/推送、探视通知、定时提醒、公告广播四类模块统计。

### Query dispatch queue

- method: `GET`
- path: `/api/notifications/queue`
- filters: `channel`, `category`, `status`, `recipientType`
- purpose: 返回待发送、发送中、失败、已送达的通知队列。

### Query broadcasts

- method: `GET`
- path: `/api/notifications/broadcasts`
- purpose: 返回公告广播列表、目标范围与回执结果。

### Create broadcast

- method: `POST`
- path: `/api/notifications/broadcasts`
- purpose: 创建公告广播并指定目标组织、楼层、班次或角色。

### Trigger resend or escalation

- method: `POST`
- path: `/api/notifications/{notificationId}/retry`
- purpose: 对失败通知做重试或人工补发。

## Field Evolution Strategy

- Alert fields:
  - `module` 建议枚举值固定为 `emergency_call`、`bed_exit`、`anomaly`、`sos`。
  - `timeline` 采用 append-only event list，避免覆盖历史动作。
- Finance fields:
  - `invoiceStatus`、`notificationStatus`、`receiptStatus` 分离，避免单状态混淆多个阶段。
  - `batchId`、`invoiceId`、`receiptId` 必须可独立追踪。
- Notification fields:
  - `category` 与 `channel` 分离。
  - `broadcastScope` 使用结构化对象，避免纯文本无法筛选。

## Example Responses

### GET /api/alerts/summary

```json
{
  "emergencyCall": { "pending": 3, "processing": 1, "closed": 12 },
  "bedExit": { "pending": 2, "processing": 2, "closed": 18 },
  "anomaly": { "pending": 5, "processing": 4, "closed": 26 },
  "sos": { "pending": 1, "processing": 1, "closed": 4 },
  "generatedAtUtc": "2026-04-13T08:00:00Z"
}
```

### GET /api/finance/summary

```json
{
  "feeCalculation": { "pendingReview": 6, "confirmed": 28 },
  "billing": { "draft": 3, "issued": 18 },
  "overdue": { "dueSoon": 4, "overdue": 2 },
  "receipts": { "pendingArchive": 5, "archived": 42 },
  "generatedAtUtc": "2026-04-13T08:00:00Z"
}
```

### GET /api/notifications/summary

```json
{
  "smsPush": { "queued": 14, "failed": 2 },
  "visitNotice": { "queued": 5, "delivered": 31 },
  "scheduledReminder": { "queued": 18, "escalated": 3 },
  "broadcast": { "draft": 2, "published": 7 },
  "generatedAtUtc": "2026-04-13T08:00:00Z"
}
```

## Rollback

- 若后续实现无法一次性覆盖全部新接口，允许先保留现有 Billing/Notification endpoint，并在 Admin BFF 内做聚合兜底。
- Alert Service 若仍暂挂在 Operations Service，下游 path 可先保持 Operations 前缀，但返回结构需对齐本文。

## Phase 1 Integration Delivery

- scope: 本轮先交付 admin 端可直接消费的真实读模型与最小动作接口，不一次性替换所有前端 demo 编辑能力。
- compatibility: 前端保留 demo/mock 回退；后端新增接口以追加为主，不破坏现有 Billing/Notification endpoint。
- caller impact: `nursing-admin-v2` 的 `/alerts`、`/financial`、`/notifications` 页面优先读取 Admin BFF 新接口。
- verification: `dotnet build nursing-backend-services.slnx && dotnet test nursing-backend-services.slnx`；`npm run lint && npm run build && CI=1 npm run test:smoke`。
- rollback: 回退新增 Admin BFF / domain-service endpoint 与前端 API 读取层，页面自动继续使用现有 demo/mock 数据。

### Phase 1 Paths

- Alert Service:
  - `GET /api/admin/alerts/summary`
  - `GET /api/admin/alerts`
  - `POST /api/admin/alerts/{alertId}/actions`
- Finance Service:
  - `GET /api/admin/finance/summary`
  - `GET /api/admin/finance/invoices`
  - `POST /api/admin/finance/invoices`
- Notification Service:
  - `GET /api/admin/notifications/summary`
  - `GET /api/admin/notifications/queue`

### Phase 1 Fallback Rule

- 当前页若收到 `503`、超时或空载荷，不直接报错中断，而是显式提示“已回退到本地 Demo 视图”，确保前端工作流可继续验证。

## Phase 1.1 Finance Invoice Create

- scope: 补齐财务页最小真实写路径，让评定结算单可以发起真实账单。
- compatibility: 新增 Admin BFF `POST` 代理，不变更既有 `GET` 返回结构。
- caller impact: `nursing-admin-v2` 的 `/financial` 页面从“按钮仅演示”升级为“按钮可创建真实账单”。
- verification: `npm run docs:build`；`dotnet build nursing-backend-services.slnx && dotnet test nursing-backend-services.slnx`；`npm run lint && npm run build && CI=1 npm run test:smoke`。
- rollback: 删除 Admin BFF `POST /api/admin/finance/invoices` 代理与前端 create 调用，保留财务摘要和账单列表读接口。

### POST /api/admin/finance/invoices

- purpose: 由 admin 财务页发起评估费结算对应的真实账单。
- downstream: `POST /api/billing/invoices`

#### Request Body

```json
{
  "elderId": "assessment-case-001",
  "elderName": "张秀英",
  "packageName": "长护险评定服务费",
  "amount": 680,
  "dueAtUtc": "2026-04-20T00:00:00.000Z"
}
```

#### Response Body

```json
{
  "invoiceId": "INV-1770000000000",
  "tenantId": "tenant-demo",
  "elderId": "assessment-case-001",
  "elderName": "张秀英",
  "packageName": "长护险评定服务费",
  "amount": 680,
  "dueAtUtc": "2026-04-20T00:00:00.000Z",
  "status": "Issued",
  "notificationStatus": "Pending",
  "createdAtUtc": "2026-04-13T09:00:00.000Z",
  "updatedAtUtc": null
}
```

### Frontend Mapping Rule

- `elderId`: 当前评定结算页若尚未暴露独立老人主键，允许先以结算关联 `assessmentId` 作为临时稳定标识。
- `packageName`: 优先使用评定模板名称；缺省时回退为 `长护险评定服务费`。
- `amount`: 以结算单 `totalAmount` 为开票金额，避免只把基金支付部分推进到账单。
- `dueAtUtc`: 首版固定按发起时间后 7 天计算，后续再切到真实账期规则。