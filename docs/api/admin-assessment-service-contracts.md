# Admin 个案评定服务契约

## Scope

- scope: 为 admin 个案评定中心补齐真实个案列表、创建、人工认定确认和认定生效接口。
- compatibility: 新增 `assessment` 读写契约，不替换已上线的长者列表、长者详情和健康档案接口。
- caller impact: Admin 前端 `/elderly/checkin`、后续 `/elderly/new` / `/elderly/import` 对接入口、Admin BFF、Elder Service、AI Orchestration。
- verification: `npm run docs:build`、`dotnet build nursing-backend-services.slnx`、`npm run lint && npm run build`。
- rollback: 回退本文、Admin BFF assessment 代理、Elder Service assessment 持久化字段和 admin checkin live 化实现，恢复当前本地 assessment workflow。

## Delivery Boundary

- 当前交付只真实化 `assessment case` 主流程。
- 评定规则集、认定模板、评估机构协同仍未接入真实服务，前端必须显式展示 `Pending Integration`，不能继续读取本地 store 伪造生产数据。
- AI 建议由 Admin BFF 在创建个案时调用 `/api/admin/ai/admission-assessment` 生成后再写入 Elder Service。

## Endpoints

### Query assessment cases

- method: `GET`
- path: `/api/admin/assessments`
- purpose: 返回个案评定中心所需的真实 assessment case 列表。
- filters:
  - `keyword`: 匹配姓名、评定编号、房间号
  - `status`: `待人工确认` / `计划已生成` / `已入住`
  - `sourceType`: `manual-form` / `document-import`
  - `scene`: `institutional` / `home`

### Create assessment case

- method: `POST`
- path: `/api/admin/assessments`
- purpose: 提交个案评定输入，调用 AI 生成建议后写入真实 assessment case。
- orchestration:
  - Admin BFF 调用 AI `POST /api/admin/ai/admission-assessment`
  - Admin BFF 将 AI 返回结果与表单一起写入 Elder Service

### Confirm assessment decision

- method: `PUT`
- path: `/api/admin/assessments/{assessmentId}/decision`
- purpose: 写入人工确认等级、认定说明、确认人和确认时间。

### Activate assessment case

- method: `PUT`
- path: `/api/admin/assessments/{assessmentId}/activate`
- purpose: 将个案从 `计划已生成` 推进到 `已入住`，作为认定生效信号。

## Field Model

### AssessmentCase

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| assessmentId | string | 稳定个案编号，当前可复用 admission id |
| elderId | string | 长者主档 id |
| tenantId | string | 租户 id |
| elderName | string | 长者姓名 |
| age | number | 年龄 |
| gender | string | 性别 |
| roomNumber | string | 房间号或服务地址 |
| phone | string | 联系电话 |
| emergencyContact | string | 紧急联系人 |
| requestedCareLevel | string | 申请照护等级 |
| status | string | 评定状态 |
| chronicConditions | string | 慢病与既往病史 |
| medicationSummary | string | 长期用药 |
| allergySummary | string | 过敏史 |
| adlScore | number | ADL 评分 |
| cognitiveLevel | string | 认知状态 |
| riskNotes | string | 风险备注 |
| entrustmentType | string? | 委托类型 |
| entrustmentOrganization | string? | 委托单位 |
| monthlySubsidy | number? | 月度补贴 |
| serviceItems | string[] | 固定服务项 |
| serviceNotes | string? | 服务备注 |
| sourceType | string | 来源类型 |
| sourceLabel | string | 来源文案 |
| sourceDocumentNames | string[] | 导入资料名 |
| sourceSummary | string? | 来源摘要 |
| aiRecommendation | object | AI 推荐对象 |
| aiRecommendation.recommendedLevel | string | AI 推荐等级 |
| aiRecommendation.confidence | number | 置信度 |
| aiRecommendation.assessmentScore | number | 综合评分 |
| aiRecommendation.reasonSummary | string | AI 摘要 |
| aiRecommendation.reasons | string[] | 结构化依据 |
| aiRecommendation.focusTags | string[] | 风险标签 |
| aiRecommendation.planTemplateCode | string | 当前命中的模板编码 |
| confirmedCareLevel | string? | 人工确认等级 |
| reviewNote | string? | 人工认定说明 |
| confirmedAtUtc | string? | 确认时间 |
| confirmedBy | string? | 确认人 |
| createdAtUtc | string | 创建时间 |

## Example Requests

### POST /api/admin/assessments

```json
{
  "elderName": "王秀兰",
  "age": 81,
  "gender": "女",
  "phone": "13800001111",
  "emergencyContact": "王敏 13900002222",
  "roomNumber": "A-1203",
  "requestedCareLevel": "二级护理",
  "chronicConditions": "高血压、糖尿病",
  "medicationSummary": "缬沙坦、二甲双胍",
  "allergySummary": "无",
  "adlScore": 46,
  "cognitiveLevel": "轻度受损",
  "riskNotes": "近三个月有一次跌倒史",
  "entrustmentType": "政府委托",
  "entrustmentOrganization": "鼓楼区医保中心",
  "monthlySubsidy": 2400,
  "serviceItems": ["生活照料", "健康监测"],
  "serviceNotes": "需重点关注夜间巡视",
  "sourceType": "manual-form",
  "sourceLabel": "前台手工建档",
  "sourceDocumentNames": [],
  "sourceSummary": "院内评估首评"
}
```

### PUT /api/admin/assessments/ADM-1744876800000/decision

```json
{
  "confirmedCareLevel": "一级护理",
  "reviewNote": "结合夜间跌倒风险与认知状态，人工上调一级",
  "confirmedBy": "护理主管"
}
```

## Example Response

```json
{
  "assessmentId": "ADM-1744876800000",
  "elderId": "ELD-1744876800000",
  "tenantId": "tenant-demo",
  "elderName": "王秀兰",
  "age": 81,
  "gender": "女",
  "roomNumber": "A-1203",
  "phone": "13800001111",
  "emergencyContact": "王敏 13900002222",
  "requestedCareLevel": "二级护理",
  "status": "待人工确认",
  "chronicConditions": "高血压、糖尿病",
  "medicationSummary": "缬沙坦、二甲双胍",
  "allergySummary": "无",
  "adlScore": 46,
  "cognitiveLevel": "轻度受损",
  "riskNotes": "近三个月有一次跌倒史",
  "entrustmentType": "政府委托",
  "entrustmentOrganization": "鼓楼区医保中心",
  "monthlySubsidy": 2400,
  "serviceItems": ["生活照料", "健康监测"],
  "serviceNotes": "需重点关注夜间巡视",
  "sourceType": "manual-form",
  "sourceLabel": "前台手工建档",
  "sourceDocumentNames": [],
  "sourceSummary": "院内评估首评",
  "aiRecommendation": {
    "recommendedLevel": "一级护理",
    "confidence": 88,
    "assessmentScore": 76,
    "reasonSummary": "存在慢病、ADL 中度受限和跌倒风险，建议上调护理等级并人工复核。",
    "reasons": [
      "ADL 评分提示中度依赖",
      "近三个月存在跌倒风险",
      "需持续监测慢病用药执行"
    ],
    "focusTags": ["跌倒风险", "夜间巡视", "慢病监测"],
    "planTemplateCode": "ASSESS-L2-HIGH-RISK"
  },
  "confirmedCareLevel": null,
  "reviewNote": null,
  "confirmedAtUtc": null,
  "confirmedBy": null,
  "createdAtUtc": "2026-04-17T08:00:00Z"
}
```

## Error Codes

- `400`: 必填字段缺失或状态流转非法
- `401`: 未登录或缺少租户上下文
- `404`: assessment case 不存在
- `502`: Admin BFF 下游 AI 或 Elder Service 调用失败

## Frontend Rules

- `/elderly/checkin` 在 live 模式下只读取该契约返回的 assessment cases。
- 规则集、认定模板、评估机构协同未接通时，只能显示 `Pending Integration` 或 `Live Unavailable`。
- 不允许在 live 模式下继续回退 `assessment-workflow`、`assessment-config-workflow`、`master-data-workflow` 的浏览器数据。