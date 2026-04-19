# 接口级文档：机构详情查询

## 基本信息

- 方法: `GET`
- 路径: `/api/admin/organizations/:organizationId`
- 说明: 查询机构基础信息、rooms 聚合摘要和员工接入状态。

## 路径参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| organizationId | string | 是 | 机构 ID |

## 响应示例

```json
{
  "organization": {
    "organizationId": "ORG-1713339999123",
    "name": "静安分院",
    "manager": "王院长",
    "totalBeds": 120,
    "occupiedBeds": 108,
    "staffCount": 0,
    "staffIntegrationStatus": "pending",
    "address": "上海市静安区示例路 88 号"
  },
  "rooms": [
    {
      "roomId": "R501",
      "name": "康复双人间",
      "capacity": 2,
      "occupied": 1,
      "status": "可入住"
    }
  ]
}
```

## 错误码

- `404`: 机构不存在
- `502`: 下游 organization service 或 rooms 聚合失败

## 调用页面

- 机构列表页
- 机构详情页
- 房间新建页的机构选择前置依赖机构主档闭环
