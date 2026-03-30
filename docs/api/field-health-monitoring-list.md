# 字段级文档：健康监测列表查询

## 请求参数字段

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| riskLevel | string | 否 | 全部 | 风险等级，取值建议为 `高风险`、`中风险`、`正常` |
| organizationId | string | 否 | 无 | 机构 ID，用于按分院筛选 |
| keyword | string | 否 | 空字符串 | 长者姓名、房间号关键词 |
| page | number | 否 | 1 | 页码，从 1 开始 |
| pageSize | number | 否 | 20 | 每页条数，建议上限 100 |

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| total | number | 当前筛选条件下的总记录数 |
| items | array | 监测对象列表 |
| items[].elderlyId | string | 长者 ID |
| items[].elderlyName | string | 长者姓名 |
| items[].roomNumber | string | 房间号 |
| items[].riskLevel | string | 当前风险等级 |
| items[].abnormalItems | string[] | 异常指标列表 |
| items[].latestAt | string | 最近一次监测时间 |
| items[].aiSummary | string | AI 异常解释摘要，可为空 |

## 字段约束

- `riskLevel` 应与页面展示枚举保持一致，避免前后端定义不一致。
- `abnormalItems` 至少返回 1 项异常指标，若为空则不应进入高风险列表。
- `latestAt` 建议统一为 `YYYY-MM-DD HH:mm`。
- `aiSummary` 允许为空，页面需支持无 AI 解释降级。

## 状态流

1. 用户进入健康监测页。
2. 页面按筛选条件请求列表。
3. 若命中高风险对象，优先展示在顶部。
4. 用户点击对象后进入详情或 AI 推理页。
