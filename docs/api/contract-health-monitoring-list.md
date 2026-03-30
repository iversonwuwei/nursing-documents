# 完整契约文档：健康监测列表查询

## 契约目标

定义健康监测列表接口的稳定输入输出、兼容策略、异常处理与回滚约束，供前后端联调和发布使用。

## 接口定义

- 方法: `GET`
- 路径: `/api/health/monitoring`
- 调用方: 健康监测页、Dashboard 健康卡片、AI 推理入口

## 请求契约

| 字段 | 类型 | 必填 | 约束 | 说明 |
| --- | --- | --- | --- | --- |
| riskLevel | string | 否 | 枚举 | `高风险`、`中风险`、`正常` |
| organizationId | string | 否 | UUID/业务 ID | 机构筛选 |
| keyword | string | 否 | 最长 50 字符 | 姓名或房间关键词 |
| page | number | 否 | >= 1 | 页码 |
| pageSize | number | 否 | 1-100 | 分页大小 |

## 响应契约

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| total | number | 是 | 当前筛选条件下总数 |
| items | array | 是 | 结果列表 |
| items[].elderlyId | string | 是 | 长者 ID |
| items[].elderlyName | string | 是 | 长者姓名 |
| items[].roomNumber | string | 是 | 房间号 |
| items[].riskLevel | string | 是 | 风险等级 |
| items[].abnormalItems | string[] | 是 | 异常指标列表 |
| items[].latestAt | string | 是 | 最近监测时间 |
| items[].aiSummary | string | 否 | AI 摘要，允许为空 |

## 状态流

1. 页面发起查询。
2. 服务按 riskLevel、organizationId、keyword 过滤。
3. 结果按风险优先级与最近时间排序。
4. 返回分页结果。
5. 页面根据 `aiSummary` 是否为空决定展示 AI 摘要或降级说明。

## 兼容性策略

- 新增字段必须保持向后兼容，只能追加，不能改名。
- `riskLevel` 枚举扩展前需同步前端映射。
- `aiSummary` 缺失时前端必须可降级。

## 幂等与缓存

- 查询接口天然幂等。
- 可按筛选参数做短时缓存，但高风险数据不建议长缓存。

## 失败模式

- 参数非法返回 `400`
- 权限异常返回 `401/403`
- 服务内部错误返回 `500`

## 可观测性

- 建议日志字段: `riskLevel`、`organizationId`、`page`、`pageSize`
- 建议指标: 查询耗时、结果数、空结果比例、错误率

## 回滚说明

如新字段导致页面异常，可回退为只返回基础字段 `elderlyId`、`elderlyName`、`roomNumber`、`riskLevel`、`latestAt`。
