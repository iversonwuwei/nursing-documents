# 字段级文档：AI 日志审计查询

## 请求参数字段

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| source | string | 否 | 无 | 来源页标识，例如 `health-monitoring`、`equipment-list` |
| focus | string | 否 | 无 | 当前关注点，例如 `health-risk`、`equipment-patrol` |
| entityId | string | 否 | 无 | 对象 ID |
| entityName | string | 否 | 无 | 对象名称 |
| channel | string | 否 | 全部 | 业务渠道，例如 `Admin / 健康总览` |
| keyword | string | 否 | 空字符串 | 模糊搜索关键词 |
| page | number | 否 | 1 | 页码 |
| pageSize | number | 否 | 20 | 每页条数 |

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| items | array | 日志列表 |
| items[].id | string | 日志编号 |
| items[].agent | string | Agent 名称 |
| items[].channel | string | 所属渠道 |
| items[].operator | string | 操作人 |
| items[].summary | string | AI 生成内容摘要 |
| items[].outcome | string | 人工确认或后续结果 |
| items[].createdAt | string | 生成时间 |
| items[].source | string | 来源页标识 |
| items[].focus | string | 关注点 |
| items[].entityId | string | 对象 ID |
| items[].entityName | string | 对象名称 |

## 字段约束

- `source`、`focus`、`entityId` 是上下文联动的稳定字段，禁止随意改名。
- `summary` 与 `outcome` 必须保留审计含义，不应用于承载结构化状态。
- `channel` 用于人工筛选，不能替代结构化上下文字段。

## 降级策略

- 若结构化字段缺失，页面可回退到 channel 和 keyword 筛选。
- 若无日志命中，页面应展示空态而非错误态。
