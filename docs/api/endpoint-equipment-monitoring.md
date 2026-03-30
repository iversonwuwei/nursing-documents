# 接口级文档：设备监控状态查询

## 基本信息

- 方法: `GET`
- 路径: `/api/equipment/monitoring`
- 说明: 查询设备在线、信号、电量和最近告警。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| status | string | 否 | online、warning、offline |
| category | string | 否 | 设备分类 |
| keyword | string | 否 | 设备名、位置关键词 |

## 响应示例

```json
{
  "items": [
    {
      "id": "EQ005",
      "name": "智能手环",
      "room": "静安店-301-1",
      "status": "warning",
      "signal": 42,
      "battery": 18,
      "lastAlert": "2026-03-30 08:55"
    }
  ]
}
```

## 错误码

- `500`: 设备监控状态查询失败

## 调用页面

- 设备监控页
- 设备状态页
- AI 推理页设备上下文卡片
