# 接口级文档：低库存摘要查询

## 基本信息

- 方法: `GET`
- 路径: `/api/supplies/low-stock-summary`
- 说明: 查询低库存物资缺口、优先级和补货建议摘要。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| category | string | 否 | 物资分类 |
| urgentOnly | boolean | 否 | 是否只看高优先级 |

## 响应示例

```json
{
  "lowStockCount": 2,
  "totalGap": 7,
  "topSupply": {
    "id": "SP001",
    "name": "成人护理垫",
    "gap": 5
  }
}
```

## 错误码

- `500`: 低库存摘要查询失败

## 调用页面

- 物资列表页
- 报表中心
- AI 运营中心
