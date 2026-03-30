# 接口级文档：健康监测列表查询

## 基本信息

- 方法: `GET`
- 路径: `/api/health/monitoring`
- 说明: 查询健康监测对象列表及当前风险信号。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| riskLevel | string | 否 | 高风险、中风险、正常 |
| organizationId | string | 否 | 机构 ID |
| keyword | string | 否 | 长者姓名、房间关键词 |
| page | number | 否 | 页码 |
| pageSize | number | 否 | 分页大小 |

## 响应示例

```json
{
  "total": 1,
  "items": [
    {
      "elderlyId": "E001",
      "elderlyName": "张桂英",
      "roomNumber": "201-1",
      "riskLevel": "高风险",
      "abnormalItems": ["血氧偏低", "高压偏高"],
      "latestAt": "2026-03-30 09:18"
    }
  ]
}
```

## 错误码

- `400`: 风险等级参数非法
- `500`: 健康监测列表查询失败

## 调用页面

- 健康监测页
- Dashboard 健康摘要卡片
