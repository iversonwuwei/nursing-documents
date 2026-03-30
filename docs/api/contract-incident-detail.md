# 完整契约文档：事故详情查询

## 契约目标

保证事故详情页、AI 复盘页和日志追溯页读取到一致的事故主体、处理时间线与责任人信息。

## 接口定义

- 方法: `GET`
- 路径: `/api/incidents/:incidentId`
- 调用方: 事故详情页、AI 日志页、AI 推理页

## 路径契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| incidentId | string | 是 | 事故 ID |

## 响应契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| id | string | 是 | 事故 ID |
| title | string | 是 | 事故标题 |
| level | string | 是 | 事故等级 |
| status | string | 是 | 当前处理状态 |
| elderlyName | string | 否 | 关联长者姓名 |
| occurredAt | string | 是 | 发生时间 |
| owner | string | 是 | 当前责任人 |
| timeline | array | 是 | 处理时间线 |
| timeline[].time | string | 是 | 动作时间 |
| timeline[].action | string | 是 | 动作说明 |
| retrospectiveSummary | string | 否 | 复盘摘要 |

## 状态流

1. 页面按 `incidentId` 拉取详情。
2. 服务返回基础信息和处理时间线。
3. 页面展示责任人、状态和复盘摘要。
4. 用户可继续进入 AI 日志或规则页。

## 兼容性策略

- `level` 和 `status` 需维持稳定枚举。
- `timeline` 可追加字段，但不应移除 `time` 与 `action`。
- `retrospectiveSummary` 允许为空，前端应降级处理。

## 失败模式

- `404`: 事故不存在
- `500`: 事故详情查询失败

## 可观测性

- 建议日志字段: `incidentId`、`level`、`status`
- 建议指标: 详情查询成功率、时间线缺失率、复盘摘要覆盖率

## 回滚说明

如时间线结构变更导致页面异常，可临时回退为只展示基础事故信息与责任人。
