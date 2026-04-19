# 接口级文档：健康档案列表与建档

## 基本信息

- 方法: `GET` / `POST`
- 路径: `/api/admin/health/archives`
- 说明: 由 Admin BFF 聚合 Health Service 与 Elder Service，返回 admin 健康档案页的真实列表，并提供最小健康建档写入口。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| keyword | string | 否 | 长者姓名、房间关键词 |

## 响应示例

```json
{
  "items": [
    {
      "elderId": "E003",
      "tenantId": "tenant-demo",
      "elderName": "陈志华",
      "roomNumber": "C-1502",
      "age": 85,
      "careLevel": "L4",
      "admissionStatus": "Active",
      "bloodPressure": "145/92",
      "heartRate": 92,
      "temperature": 37.1,
      "bloodSugar": 8.8,
      "oxygen": 90,
      "riskSummary": "血氧偏低且血糖波动，需要复测并观察医生随访。",
      "updatedAtUtc": "2026-04-13T00:05:00Z"
    }
  ],
  "generatedAtUtc": "2026-04-13T00:20:00Z"
}
```

## POST 请求体示例

```json
{
  "elderId": "E003",
  "bloodPressure": "138/85",
  "heartRate": 78,
  "temperature": 36.7,
  "bloodSugar": 6.2,
  "oxygen": 96,
  "riskSummary": "晨间复测后需继续观察夜间离床风险。"
}
```

## POST 响应示例

```json
{
  "elderId": "E003",
  "tenantId": "tenant-demo",
  "elderName": "陈志华",
  "roomNumber": "C-1502",
  "age": 85,
  "careLevel": "L4",
  "admissionStatus": "Active",
  "bloodPressure": "138/85",
  "heartRate": 78,
  "temperature": 36.7,
  "bloodSugar": 6.2,
  "oxygen": 96,
  "riskSummary": "晨间复测后需继续观察夜间离床风险。",
  "updatedAtUtc": "2026-04-14T01:00:00Z"
}
```

## 错误码

- `400`: 长者主档缺失、核心健康字段不合法或请求参数格式错误
- `401`: 未登录或会话失效
- `502`: Admin BFF 到 Health Service 或 Elder Service 聚合失败

## 调用页面

- 健康档案页
- 健康档案新建页
