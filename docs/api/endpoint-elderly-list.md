# 接口级文档：长者列表查询

## 基本信息

- 方法: `GET`
- 路径: `/api/admin/elders`
- 说明: 由 Admin BFF 代理 Elder Service 查询长者列表，支持按护理等级、入住状态和姓名关键词筛选。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| name | string | 否 | 长者姓名关键词 |
| careLevel | string | 否 | 护理等级 |
| status | string | 否 | 入住状态，如 `Active`、`AdmissionReviewed` |
| page | number | 否 | 页码，默认 1 |
| pageSize | number | 否 | 每页条数，默认 20 |

## 响应示例

```json
{
  "total": 2,
  "page": 1,
  "pageSize": 20,
  "items": [
    {
      "elderId": "E001",
      "tenantId": "tenant-demo",
      "elderName": "王建国",
      "age": 82,
      "gender": "male",
      "careLevel": "L3",
      "roomNumber": "A-1203",
      "admissionStatus": "Active",
      "familyContactName": "王敏",
      "admissionCreatedAtUtc": "2026-04-10T01:00:00Z"
    }
  ]
}
```

## 错误码

- `400`: 查询参数格式错误
- `401`: 未登录或会话失效
- `502`: Admin BFF 到 Elder Service 聚合失败

## 调用页面

- 长者列表页
- 入住办理页的长者选择弹层
- 健康档案新建页的长者选择下拉
