# 接口级文档：机构详情查询

## 基本信息

- 方法: `GET`
- 路径: `/api/organizations/:organizationId`
- 说明: 查询机构基础信息、床位、员工和承接压力摘要。

## 路径参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| organizationId | string | 是 | 机构 ID |

## 响应示例

```json
{
  "id": "O001",
  "name": "静安分院",
  "manager": "王院长",
  "totalBeds": 120,
  "occupiedBeds": 108,
  "staffCount": 42,
  "address": "上海市静安区示例路 88 号"
}
```

## 错误码

- `404`: 机构不存在
- `500`: 机构详情查询失败

## 调用页面

- 机构列表页
- 机构详情页
