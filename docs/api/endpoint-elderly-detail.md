# 接口级文档：长者详情查询

## 基本信息

- 方法: `GET`
- 路径: `/api/elderly/:elderlyId`
- 说明: 查询单个长者的基础档案、护理等级、家属信息和风险摘要。

## 路径参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| elderlyId | string | 是 | 长者 ID |

## 响应示例

```json
{
  "id": "E001",
  "name": "张桂英",
  "age": 82,
  "gender": "女",
  "organizationName": "静安分院",
  "roomNumber": "201-1",
  "careLevel": "一级护理",
  "status": "在住",
  "familyContacts": [
    {
      "name": "张敏",
      "relation": "女儿",
      "phone": "13800138000"
    }
  ],
  "riskTags": ["高血压", "夜间低氧"]
}
```

## 错误码

- `404`: 长者不存在
- `500`: 长者详情查询失败

## 调用页面

- 长者详情页
- 健康监测页详情侧栏
