# 接口级文档：长者列表查询

## 基本信息

- 方法: `GET`
- 路径: `/api/elderly`
- 说明: 查询长者列表，支持按机构、护理等级、状态和关键词筛选。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| keyword | string | 否 | 长者姓名、编号关键词 |
| organizationId | string | 否 | 机构 ID |
| careLevel | string | 否 | 护理等级 |
| status | string | 否 | 在住、待入住、已出院 |
| page | number | 否 | 页码，默认 1 |
| pageSize | number | 否 | 每页条数，默认 20 |

## 响应示例

```json
{
  "total": 2,
  "items": [
    {
      "id": "E001",
      "name": "张桂英",
      "organizationName": "静安分院",
      "roomNumber": "201-1",
      "careLevel": "一级护理",
      "status": "在住",
      "riskTags": ["高血压", "夜间低氧"]
    }
  ]
}
```

## 错误码

- `400`: 查询参数格式错误
- `401`: 未登录或会话失效
- `500`: 长者列表查询失败

## 调用页面

- 长者列表页
- 入住办理页的长者选择弹层
