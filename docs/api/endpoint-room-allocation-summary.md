# 接口级文档：床位分配摘要查询

## 基本信息

- 方法: `GET`
- 路径: `/api/rooms/allocation-summary`
- 说明: 查询可入住房间、空床、预留床和机构承接摘要。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| organizationId | string | 否 | 指定机构 |
| careLevel | string | 否 | 指定护理等级 |

## 响应示例

```json
{
  "availableRooms": 6,
  "availableBeds": 10,
  "reservedBeds": 3,
  "highestPressureOrganization": "静安分院"
}
```

## 错误码

- `500`: 床位摘要查询失败

## 调用页面

- 房间列表页
- 入住办理页
- 机构详情页床位区块
