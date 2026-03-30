# 接口级文档：AI 日志审计查询

## 基本信息

- 方法: `GET`
- 路径: `/api/ai/logs`
- 说明: 按 source、focus、entity、channel 过滤 AI 日志记录。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| source | string | 否 | 来源页标识 |
| focus | string | 否 | 关注点 |
| entityId | string | 否 | 对象 ID |
| entityName | string | 否 | 对象名称 |
| channel | string | 否 | 渠道 |
| keyword | string | 否 | 关键词 |

## 响应示例

```json
{
  "items": [
    {
      "id": "Q-3011",
      "agent": "设备巡检 Agent",
      "summary": "对待维修和维保预警设备生成优先巡检与备用方案建议。",
      "outcome": "已标出 2 台优先巡检设备，等待本班处理。",
      "source": "equipment-list",
      "focus": "equipment-patrol",
      "entityId": "equipment-board"
    }
  ]
}
```

## 错误码

- `400`: 查询参数非法
- `500`: AI 日志查询失败

## 调用页面

- AI 日志审计页
- AI 推理页上下文证据区块
- AI 规则治理页关联日志区块
