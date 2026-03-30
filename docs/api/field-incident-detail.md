# 字段级文档：事故详情查询

## 路径字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| incidentId | string | 是 | 事故 ID |

## 响应字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 事故 ID |
| title | string | 事故标题 |
| level | string | 事故等级 |
| status | string | 处理状态 |
| elderlyName | string | 关联长者姓名，可为空 |
| occurredAt | string | 发生时间 |
| owner | string | 当前责任人 |
| timeline | array | 处理时间线 |
| timeline[].time | string | 时间点 |
| timeline[].action | string | 已执行动作 |
| retrospectiveSummary | string | 复盘摘要，可为空 |

## 字段约束

- `level` 与 `status` 必须使用稳定枚举，避免详情页颜色和筛选错乱。
- `timeline` 应按时间正序或倒序固定输出，不允许混乱。
- `retrospectiveSummary` 可以为空，但严重事件建议补齐。

## 页面影响

- 详情页时间线直接依赖 `timeline[]`。
- AI 复盘卡片依赖 `retrospectiveSummary`。
- 当前责任人显示依赖 `owner`。
