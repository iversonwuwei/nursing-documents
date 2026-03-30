# 完整契约文档：AI 日志审计查询

## 契约目标

保证 AI 日志页、推理页和规则页围绕统一上下文查询到一致结果。

## 接口定义

- 方法: `GET`
- 路径: `/api/ai/logs`
- 调用方: AI 日志页、AI 推理页、AI 规则页

## 请求契约

| 字段 | 类型 | 必填 | 约束 | 说明 |
| --- | --- | --- | --- | --- |
| source | string | 否 | 稳定枚举/标识 | 来源页标识 |
| focus | string | 否 | 稳定标识 | 关注点 |
| entityId | string | 否 | 业务 ID | 对象 ID |
| entityName | string | 否 | 最长 100 字符 | 对象名称 |
| channel | string | 否 | 文本 | 人工渠道筛选 |
| keyword | string | 否 | 最长 50 字符 | 模糊查询 |
| page | number | 否 | >= 1 | 页码 |
| pageSize | number | 否 | 1-100 | 分页大小 |

## 响应契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| items | array | 是 | 日志列表 |
| items[].id | string | 是 | 日志编号 |
| items[].agent | string | 是 | Agent 名称 |
| items[].channel | string | 是 | 渠道 |
| items[].operator | string | 是 | 操作人 |
| items[].summary | string | 是 | AI 摘要 |
| items[].outcome | string | 是 | 人工确认或后续结果 |
| items[].createdAt | string | 是 | 时间 |
| items[].source | string | 否 | 来源页标识 |
| items[].focus | string | 否 | 关注点 |
| items[].entityId | string | 否 | 对象 ID |
| items[].entityName | string | 否 | 对象名称 |

## 查询优先级

1. 先按 `source + focus + entityId/entityName` 精确匹配。
2. 无精确结果时回退到 `channel + keyword` 启发式匹配。
3. 无结果时返回空数组。

## 兼容性策略

- `source`、`focus`、`entityId`、`entityName` 属于稳定上下文字段，不允许重命名。
- 结构化字段可为空，但前端必须识别为空场景。
- `channel` 仅作为辅助筛选，不能替代结构化契约。

## 幂等与缓存

- 查询接口天然幂等。
- 不建议长缓存，以免审计页读取到过期结果。

## 失败模式

- 参数非法返回 `400`
- 无权限查看返回 `403`
- 查询失败返回 `500`

## 可观测性

- 建议日志字段: `source`、`focus`、`entityId`、`channel`
- 建议指标: 精确匹配命中率、兜底匹配命中率、空结果率、查询耗时

## 回滚说明

如结构化查询逻辑异常，可临时回退到 channel 和 keyword 过滤策略，但需在文档中说明精度下降。
